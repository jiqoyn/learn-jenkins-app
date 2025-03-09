pipeline{
    agent any
    environment {
        NETLIFY_SITE_ID = '6e9ec4b6-860f-486f-9343-08ef604712a1'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') /* this is the token we created within netlify, and stored with
        jenkins under credentials. this is to keep the token secured */      
        }
    
    stages{
        stage("Build"){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                    #ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    #ls -la        
                '''
            }
        } 

        stage('Run Tests'){
            parallel {        
                stage('Unit test') {
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            
                        }
                    }            
                }
                stage('E2E'){
                    
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
                        /*for all installation and arguments, there are docs. we went over them, and the links to the main 
                        set of docs for the commands below are within onenote under links for jenkins.  */
                        sh '''
                            npm install  @playwright/test@1.51.0
                            npx playwright install 
                            npm install -g serve
                            serve -s build &   
                            sleep 10                
                            npx playwright test --reporter=line
                            #testing comments
                            '''
                    }
                    
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    } 
                }
            }
        }

        stage("Deploy Staging"){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                     npm install netlify-cli -g
                    netlify --version 
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status   
                    netlify deploy --dir=build 
                    #comment
                '''
            }
        }
        stage("Deploy Prod"){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                     npm install netlify-cli -g
                    netlify --version 
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status   
                    netlify deploy --dir=build --prod
                    #comment
                '''
            }
        } 
        
        stage('Prod E2E'){
                    
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
                    
                    environment {
                        CI_ENVIRONMENT_URL = 'https://jolly-youtiao-2bb5cf.netlify.app/'
                    }
                    
                    steps {
                        /*for all installation and arguments, there are docs. we went over them, and the links to the main 
                        set of docs for the commands below are within onenote under links for jenkins.  */
                        sh '''
                            npm install  @playwright/test@1.51.0
                            npx playwright install                       
                            npx playwright test --reporter=line
                            #testing comments
                            '''
                    }
                    
                    post {
                        always {                              
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    } 
            }
    }
       
        
    
        
}