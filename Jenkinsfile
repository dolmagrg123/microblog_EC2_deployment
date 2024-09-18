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
                gunicorn -b :5000 -w 4 microblog:app > gunicorn.log 2>&1 &
                echo $! > gunicorn.pid
                '''
            }
        }
        stage ('Post-Deploy') {
            steps {
                script {
                    // Confirm Gunicorn is running and exit gracefully
                    sh '''#!/bin/bash
                    if [ -f gunicorn.pid ]; then
                        echo "Gunicorn PID file found. Gunicorn should be running."
                        echo "Gunicorn PID: $(cat gunicorn.pid)"
                    else
                        echo "Gunicorn PID file not found. Gunicorn may not be running."
                    fi
                    '''
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline completed. Gunicorn should be running in the background."
        }
    }
}
