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
                sh 'wget "https://raw.githubusercontent.com/lintechso/laravel-hello-world/master/owasp-dependency-check.sh" '
                sh 'chmod +x owasp-dependency-check.sh'
                sh 'bash owasp-dependency-check.sh'
                sh 'cat /home/parham/OWASP-Dependency-Check/reports/dependency-check-report.xml'
            }
        }
        stage ('Test with phpstan'){
            agent { label 'parham' }
            steps {
                  sh 'docker run --rm -v $(pwd):/usr/src/app -w /usr/src/app jakzal/phpqa composer install --ignore-platform-reqs --no-scripts --no-progress --no-suggest'
                  sh 'docker run --rm -v $(pwd):/usr/src/app -w /usr/src/app jakzal/phpqa phpstan analyse --level 3 app || exit 0'
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
                }
                failure {
                    echo '> Error in Building ...'
                    sh 'docker-compose down'
                }
            }
        }
    }
}
