
pipeline {
    agent {
        label 'jmeter-linux'
    }

    parameters {
        choice(
            name: 'TEST_SCRIPT',
            choices: ['sample-api-performance-test.jmx'],
            description: 'Select JMeter script'
        )

        string(
            name: 'USERS',
            defaultValue: '5',
            description: 'Number of virtual users'
        )

        string(
            name: 'RAMPUP',
            defaultValue: '5',
            description: 'Ramp-up time in seconds'
        )

        string(
            name: 'LOOPS',
            defaultValue: '2',
            description: 'Loop count'
        )
    }

    stages {
        stage('Run JMeter Test') {
            steps {
                sh '''
                    set -e

                    rm -rf results.jtl HTMLReport

                    echo "Selected Script: $TEST_SCRIPT"
                    echo "Virtual Users: $USERS"
                    echo "Ramp-up: $RAMPUP seconds"
                    echo "Loop Count: $LOOPS"

                    /opt/apache-jmeter-5.6.3/bin/jmeter \
                      -n \
                      -t "jmeter/$TEST_SCRIPT" \
                      -Jusers="$USERS" \
                      -Jrampup="$RAMPUP" \
                      -Jloops="$LOOPS" \
                      -l results.jtl \
                      -e \
                      -o HTMLReport
                '''
            }
        }

        stage('SLA Validation') {
            steps {
                sh '''
                    python3 - <<'PY'
import csv
import math
import sys

with open('results.jtl', newline='') as file:
    rows = list(csv.DictReader(file))

if not rows:
    print("FAIL: No JMeter results found")
    sys.exit(1)

times = sorted(float(row['elapsed']) for row in rows)

errors = sum(
    row['success'].strip().lower() != 'true'
    for row in rows
)

p95 = times[math.ceil(len(times) * 0.95) - 1]
error_rate = errors / len(rows) * 100

print(f"Total Samples: {len(rows)}")
print(f"P95 Response Time: {p95:.2f} ms")
print(f"Error Rate: {error_rate:.2f}%")

if p95 > 3000 or error_rate > 1:
    print("SLA STATUS: FAIL")
    sys.exit(1)

print("SLA STATUS: PASS")
PY
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'results.jtl,HTMLReport/**',
                             allowEmptyArchive: true,
                             fingerprint: true
        }
    }
}
