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

        stage('aws'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                '''
            }
            }
        }
       

        stage('Build') {
            agent {
                docker {
                    image 'my-playwright'
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
                            image 'my-playwright'
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify deploy --dir=build --prod --json > deploy-output.json
                    node-jq -r '.deploy_url' deploy-output.json       
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

    

