pipeline {
    agent any
    
    options {
        ansiColor('xterm')
    }

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:22-alpine'
                }
            }
            steps {
                sh 'npm ci --include=dev'
                sh 'npm run build'
            }
        }

        stage('test') {
            parallel {
                stage('unit tests') {
                    agent {
                        docker {
                            image 'node:22-alpine'
                            //reuseNode true
                        }
                    }
                    steps {
                        sh 'npm ci --include=dev'
                        // Unit tests with Vitest
                        sh 'npx vitest run --reporter=verbose'
                    }
                }
                stage('integration tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.60.0-jammy'
                            //reuseNode true
                        }
                    }
                    steps {
                        //sh 'rm -rf node_modules'
                        sh 'npm ci --include=dev'
                        sh 'npm install cssesc --save-dev'
                        sh 'npx playwright test'
                    }
                }
            }
        }

        stage('deploy') {
            agent {
                docker {
                    image 'alpine'
                }
            }
            steps {
                // Mock deployment which does nothing
                echo 'Mock deployment was successful!'
            }
        }

        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.60.0-jammy'
                    //reuseNode true
                }
            }
            environment {
                E2E_BASE_URL = 'https://spanish-cards.netlify.app/'
            }
            steps {
                //sh 'rm -rf node_modules'
                sh 'npm ci --include=dev'
                sh 'npm install cssesc --save-dev'
                sh 'npx playwright test'
            }
            post {
                always {
                    publishHTML(allowMissing: true, alwaysLinkToLastBuild: true, icon:'',keepAll: false, reportDir: 'reports-e2e/html/', reportFiles: 'index.html', reportName: 'Playwright Test HTML Report', reportTitles:'', useWrapperFileDirectly:true)
                    junit stdioRetention: 'ALL', testResults: 'reports-e2e/*.xml'
                }
            }
        }
    }
}