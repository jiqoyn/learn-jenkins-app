pipeline {
    agent any

    stages {
        // this is my build stage

        /*
        this is another way to write comments, but 
        can span lines
         */
        /* stage('Build') {
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
        } */
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Starting Testing Phase'
                    npm test
                    test -f build/index.html     
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.51.0-noble'
                    args '--ipc=host'
                    /*Initially, I had the entire line like this:  
                    docker pull mcr.microsoft.com/playwright:v1.51.0-noble
                    but was getting errors. checked his video and he only had starting at mcr. not sure why, i'll 
                    ask gpt later. it turns out the jenkins software knows to use the docker pull command*/
                    reuseNode true            
                }
            }
            steps {
                sh '''
                  npm install --save-dev @playwright/test@1.51.0
                  npm install -g serve
                  serve -s build & 
                  sleep 10 
                  npx playwright test     
                '''
            }
        }
    }
    post {
        always {
            junit '**/test-results/*.xml'
        }
    }
}

