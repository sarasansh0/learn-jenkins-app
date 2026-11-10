pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    mkdir -p test-results
                    CI=true JEST_JUNIT_OUTPUT_DIR=test-results \
                    JEST_JUNIT_OUTPUT_NAME=junit.xml \
                    npm test -- --watchAll=false --reporters=default --reporters=jest-junit
                '''
            }
            post {
                always {
                    junit 'test-results/junit.xml'
                }
            }
        }
    }
}