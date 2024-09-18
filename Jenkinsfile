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
                if [[ $(ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2) != 0 ]]
                then
                ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
                kill $(cat pid.txt)
                exit 0
                fi
                '''
            }
        }
        stage ('Deploy') {
        steps {
            sh '''#!/bin/bash
            source venv/bin/activate
            echo "Starting Gunicorn..."
            
            # Create stayAlive script
            echo '#!/bin/bash' > stayAlive.sh
            echo 'while true; do sleep 1000; done' >> stayAlive.sh
            chmod +x stayAlive.sh
            
            # Start Gunicorn with --daemon option
            gunicorn -b :5000 -w 4 microblog:app --daemon
            
            # Start stayAlive script in the background
            ./stayAlive.sh &
            
            # Print a message to indicate that Gunicorn has been started
            echo "Gunicorn started with PID: $(pgrep -f 'gunicorn')"
            '''
        }
        }

        }

    }
}
