pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '55ac451d-fccb-4424-8b28-a8111989d5c5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-Jenkins')
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
                '''
            }
        }
        stage('Run Tests'){
            parallel {
                stage("Tests") {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            echo "Test Stage"
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage("E2E"){
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 15
                            npx playwright test  --reporter=line
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
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
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploy to production Ste ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod
                    echo 'END'
                '''
            }
        }
    }
    
}
