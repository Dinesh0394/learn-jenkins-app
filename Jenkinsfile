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
                    ls -la
                    test -f build/index.html
                    CI=true npm test -- --watchAll=false
                '''
            }
        }
        stage('E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.63.0-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install -g playwright
                    serve -s build
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
            cleanWs()
        }
    }
}