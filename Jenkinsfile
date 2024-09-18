pipeline {
    agent any
    stages {
        stage ('Build') {
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
        stage ('Test') {
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
        stage ('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ('Deploy') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                echo "Starting Gunicorn..."
                nohup gunicorn -b :5000 -w 4 microblog:app > gunicorn.log 2>&1 &
                echo $! > gunicorn.pid
                sleep 5 # Wait to ensure Gunicorn starts
                echo "Gunicorn PID: $(cat gunicorn.pid)"
                echo "Gunicorn Logs:"
                tail -n 10 gunicorn.log
                '''
            }
        }
    }
    post {
        always {
            echo "Pipeline completed. Checking Gunicorn status..."
            sh '''#!/bin/bash
            if [ -f gunicorn.pid ]; then
                echo "Gunicorn PID file exists. It should be running in the background."
            else
                echo "Gunicorn PID file does not exist. Gunicorn may not be running."
                exit 1
            fi
            '''
        }
    }
}
