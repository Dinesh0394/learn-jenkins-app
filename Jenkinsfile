pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'b17192a9-8d95-4b57-a290-26d646bd07fc'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:20'
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
            parallel {

                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:20'
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

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                icon: '',
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'HTML Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }

            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:20'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to Netlify..."
                    echo "Site ID: ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status

                '''
            }
        }
    }
}