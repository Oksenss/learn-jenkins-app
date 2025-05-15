pipeline {
    agent any

    stages {
        stage('Prepare') {
            steps {
                sh 'docker pull mcr.microsoft.com/playwright:v1.39.0-jammy'
            }
        }

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

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '--pull=never' // Don't try to pull the image again
                        }
                    }

                    steps {
                        sh '''
                            npm ci
                            npm install serve
                            node_modules/.bin/serve -s build &
                            SERVE_PID=$!
                            sleep 10
                            npx playwright test --reporter=html
                            EXIT_CODE=$?
                            kill $SERVE_PID || true
                            exit $EXIT_CODE
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
    }
}
