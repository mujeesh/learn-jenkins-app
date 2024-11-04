pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '55ac451d-fccb-4424-8b28-a8111989d5c5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-Jenkins')
    }

    stages {

        stage ('Docker')
        {
            steps {
                sh 'docker build -t my-dock .'
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
                stage("E2E"){
                    agent {
                        docker {
                            image 'my-dock'
                            reuseNode true
                        }
                    }

                    steps{
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test  --reporter=line
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report',
                             reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }

        }
            
        stage('Deploy Staging') {
            agent {
                docker {
                     image 'my-dock'
                     reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to production Ste ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build  --json > deploy-output.json
                    node-jq -r ".deploy_url" deploy-output.json
                '''

                script {
                    env.STAGING_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }
        stage ('Approval')
        {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    input message: 'Ready to deploy?.', ok: 'Yes, I am sure I want to deploy'
                }
                
            }
        }

        stage('Deploy Production') {
            agent {
                docker {
                     image 'my-dock'
                     reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to production Ste ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                '''
            }
        }

        stage("Prod E2E"){
            agent {
                docker {
                    image 'my-dock'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps{
                sh '''
                    npx playwright test  --reporter=line
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report',
                     reportFiles: 'index.html', reportName: 'E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
    
}
