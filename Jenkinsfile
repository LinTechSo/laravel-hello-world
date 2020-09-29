pipeline {
    agent none
    stages {
        stage ('CHECK-GIT-SECRETS') {
            agent { label 'parham'}
            steps {
                sh 'rm trufflehog || true'
                sh 'docker run gesellix/trufflehog --json https://github.com/lintechso/webapp.git > trufflehog'
                sh 'cat trufflehog'
            }
        }
        stage ('Source Composition Analysis') {
            agent { label 'parham'}
            steps {
                sh 'rm owasp* || true'
                sh 'wget "https://raw.githubusercontent.com/lintechso/webapp/master/owasp-dependency-check.sh" '
                sh 'chmod +x owasp-dependency-check.sh'
                sh 'bash owasp-dependency-check.sh'
                sh 'cat /home/parham/OWASP-Dependency-Check/reports/dependency-check-report.xml'
            }
        }
        stage ('SAST') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    label 'parham'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar'
                    sh 'cat target/sonar/report-task.txt'
                }
            }
        }
        stage ('BUILD With Docker Compose'){
            agent { label 'master'}
            steps {
                sh 'docker-compose up -d --build'
                echo '$(docker-compose ps)'
            }
            post {
                success {
                    echo ' > Everything works well ...'
                    sh 'docker-compose down'
                }
                failure {
                    echo '> Error in Building ...'
                }
            }
        }
    }
}