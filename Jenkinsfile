pipeline {
  agent any
    stages {
        stage('Build') {
            steps {
                sh '''#!/bin/bash
                python3.9 -m venv venv
                source venv/bin/activate
                pip install pip --upgrade
                pip install -r requirements.txt
                pip install gunicorn pymysql cryptography
                export FLASK_APP=microblog.py
                flask translate compile
                flask db upgrade
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                py.test ./tests/unit/ --verbose --junit-xml test-reports/results.xml
                '''
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Clean') {
            steps {
                sh '''#!/bin/bash
                PID=$(pgrep -f gunicorn)

                if [ -n "$PID" ]; then
                    echo "Stopping Gunicorn process with PID $PID"
                    kill $PID
                    # Optionally wait for the process to be terminated
                    wait $PID 2>/dev/null || true
                else
                    echo "No Gunicorn process found."
                fi
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                gunicorn -b :5000 -w 4 microblog:app & echo $! > gunicorn.pid
                '''
            }
        }
    }
  }
}
