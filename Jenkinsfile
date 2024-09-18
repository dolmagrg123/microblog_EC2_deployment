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
        gunicorn -b :5000 -w 4 microblog:app &
        echo $! > gunicorn.pid
        '''
      }
    }
  }
  post {
    always {
      sh '''#!/bin/bash
      if [ -f gunicorn.pid ]; then
        kill $(cat gunicorn.pid) || true
        rm gunicorn.pid
      fi
      '''
    }
  }
}
