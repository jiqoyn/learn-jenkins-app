pipeline{
    agent any

    stages{
        /*stage("Build"){
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

        stage('Run Tests'){
            parallel {        
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
                            '''
                        }
            }
            }
                    }
        }
       
        
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    } 
        
}