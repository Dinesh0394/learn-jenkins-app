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
            parallel{
                stage('Unit Test') {
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
                post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                        }
                stage('E2E Test') {
                    agent {
                        docker {
                            imaoge 'mcr.microsoft.com/playwright:v1.63.0-noble'
                            reuseNde true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }
                }
            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
            }
        }   

        
    }
}