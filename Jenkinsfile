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
                            // Use a smaller, more specific image
                            image 'mcr.microsoft.com/playwright/node:v1.39.0-focal'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm ci
                            npm install serve
                            npx serve -s build &
                            SERVE_PID=$!
                            sleep 10
                            
                            # Make sure browsers are installed
                            npx playwright install --with-deps chromium
                            
                            # Run tests with only chromium for speed
                            npx playwright test --reporter=html --project=chromium
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
