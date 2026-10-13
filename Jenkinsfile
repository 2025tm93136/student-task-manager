pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Installing Python dependencies'
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Running automated tests'
                sh '''
                    .venv/bin/pytest -q
                '''
            }
        }
        stage('Code Inspection') {
            steps {
                echo 'Running SonarQube analysis'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        export PATH=/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/mySonarScanner/bin:$PATH
                        sonar-scanner \\
                          -Dsonar.host.url=http://localhost:9000 \\
                          -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application'
                sh '''
                    mkdir -p /tmp/student-task-manager
                    cp -r app.py templates static requirements.txt /tmp/student-task-manager/
                    cd /tmp/student-task-manager
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                    if [ -f app.pid ]; then
                        kill $(cat app.pid) 2>/dev/null || true
                        sleep 2
                    fi
                    JENKINS_NODE_COOKIE=dontKillMe nohup .venv/bin/python app.py > app.log 2>&1 &
                    echo $! > app.pid
                    sleep 3
                    curl -fs http://localhost:5000/health
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed - deployment was not completed'
        }
    }
}
