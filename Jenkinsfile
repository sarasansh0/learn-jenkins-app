pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c2a1c17c-9d33-4195-901d-ad10c3e86db5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                  retry(3) {
                    sh 'npm install serve'
                 }
                sh '''
                  node_modules/.bin/serve -s build &
                   sleep 10
                  npx playwright test --reporter=html
                  '''
                }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Report'
                    ])
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                retry(3) {
                 sh 'npm install netlify-cli'
             }
              sh '''
                 node_modules/.bin/netlify --version
                 echo "Deploying to Netlify Site ID: $NETLIFY_SITE_ID"
                 node_modules/.bin/netlify status
                 node_modules/.bin/netlify deploy --dir=build --prod --no-build
                '''
}
        }
    }
}