pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'e6f532aa-f0d0-4c61-8c85-9b493975742d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Docker'){
            steps{
                sh '''
                    docker build -t my-playwright .
                '''
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
                stage('Unit Test') {
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
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test
                        '''
                    }
                }
            }
        }
        stage('Approval'){
            steps{
                timeout(time: 15, unit: 'HOURS') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
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
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json       
                '''
            }
        }


    }
    post {
        always {
            junit 'jest-results/junit.xml'
            
        }
    }

        
}

    

