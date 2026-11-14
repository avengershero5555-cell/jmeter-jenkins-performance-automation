
pipeline {
    agent {
        label 'jmeter-linux'
    }

    parameters {
        string(name: 'USERS', defaultValue: '5',
               description: 'Virtual users')

        string(name: 'RAMPUP', defaultValue: '5',
               description: 'Ramp-up seconds')

        string(name: 'LOOPS', defaultValue: '2',
               description: 'Loop count')
    }

    stages {
        stage('Run JMeter Scripts Sequentially') {
            steps {
                sh '''
                    set -e
                    rm -rf reports
                    mkdir -p reports

                    for script in \
                      sample-api-performance-test.jmx \
                      second-api-performance-test.jmx
                    do
                        name="${script%.jmx}"

                        echo "Running: $script"

                        /opt/apache-jmeter-5.6.3/bin/jmeter \
                          -n \
                          -t "jmeter/$script" \
                          -Jusers="$USERS" \
                          -Jrampup="$RAMPUP" \
                          -Jloops="$LOOPS" \
                          -l "reports/${name}.jtl" \
                          -e \
                          -o "reports/${name}-dashboard"

                        echo "Completed: $script"
                    done
                '''
            }
        }

        stage('SLA Validation') {
            steps {
                sh '''
                    python3 - <<'PY'
import csv
import glob
import math
import sys

failed = False

files = sorted(glob.glob('reports/*.jtl'))

if len(files) != 2:
    print("FAIL: Expected 2 JTL files")
    sys.exit(1)

for path in files:
    with open(path, newline='') as f:
        rows = list(csv.DictReader(f))

    if not rows:
        print(f"FAIL: No results in {path}")
        failed = True
        continue

    times = sorted(float(r['elapsed']) for r in rows)
    errors = sum(
        r['success'].strip().lower() != 'true'
        for r in rows
    )

    p95 = times[math.ceil(len(times) * 0.95) - 1]
    error_rate = errors / len(rows) * 100

    print(f"\\nScript: {path}")
    print(f"Total Samples: {len(rows)}")
    print(f"P95: {p95:.2f} ms")
    print(f"Error Rate: {error_rate:.2f}%")

    if p95 > 3000 or error_rate > 1:
        print("SLA STATUS: FAIL")
        failed = True
    else:
        print("SLA STATUS: PASS")

if failed:
    sys.exit(1)
PY
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/**',
                             allowEmptyArchive: true,
                             fingerprint: true
        }
    }
}
