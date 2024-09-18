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
        stage ('Clean') {
            steps {
                sh '''#!/bin/bash
                # Clean up any existing Gunicorn processes if needed
                if [[ $(ps aux | grep -i "gunicorn" | grep -v "grep" | tr -s " " | head -n 1 | cut -d " " -f 2) != "" ]]
                then
                    ps aux | grep -i "gunicorn" | grep -v "grep" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
                    kill $(cat pid.txt)
                fi
                '''
            }
        }
        stage ('Deploy') {
            steps {
                sh '''#!/bin/bash
                source venv/bin/activate
                gunicorn -b :5000 -w 4 microblog:app > gunicorn.log 2>&1 &
                sleep 10  # Give some time for Gunicorn to start
                cat gunicorn.log  # Print the log to Jenkins output for debugging
        '''
            }
        }
    }
}
