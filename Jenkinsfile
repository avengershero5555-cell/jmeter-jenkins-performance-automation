
pipeline {
    agent {
        label 'jmeter-linux'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Run JMeter Performance Test') {
            steps {
                sh '''
                    rm -rf results.jtl HTMLReport

                    /opt/apache-jmeter-5.6.3/bin/jmeter \
                      -n \
                      -t jmeter/sample-api-performance-test.jmx \
                      -l results.jtl \
                      -e \
                      -o HTMLReport
                '''
            }
        }

        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: 'results.jtl,HTMLReport/**',
                                 fingerprint: true
            }
        }
    }
}
