pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '3cd82567-95c3-4835-820c-52acf1ed35bc'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
    }

    stages {
        // This is a comment for one liners
        
        /*
        This is a block comment
        line 2 
        line 3
        You could use this to disable a stage as npm tets is independent of build stage, 
        we could use this style to not be a part of pipeline.
        This is better for pipeline buildup
        */

        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                }
            }
            steps{
                sh '''
                    aws --version
                '''
            }
        } 

        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo 'Small Change'
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }
        
        stage('Tests'){
            parallel{
                stage('Unit Tests') {
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh'''
                        echo "Initating Test..."
                        mkdir -p test-results
                        test -f build/index.html
                        npm test
                        echo 'Completed Test'
                        '''
                    }
                }
                stage('E2E') {
                    agent{
                        docker{
                            image 'my-playwright'
                            reuseNode true
                            // args '-u root:root' - Is a bad idea as mounted with different username

                        }
                    }
                    steps{
                        sh'''
                        serve -s build & # This & will not block the execution 
                        sleep 10 # To avoid commands starting one after the other, delay untill server is setup
                        npx playwright test  --reporter=html
                        echo 'E2E Completed'
                        '''
                    }
                    post{
                        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])        
                        }
                    }
                }
            }
        }


        stage('Deploy staging') {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                    // args '-u root:root' - Is a bad idea as mounted with different username
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'VAR_TO_BE_SET'
            }

            steps{
                sh'''
                netlify --version
                echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --no-build --json > deploy-output.json # removed --prod so Jenkins will create temporary preview environment
                CI_ENVIRONMENT_URL="$(jq -r '.deploy_url' deploy-output.json)"

                npx playwright test  --reporter=html
                echo 'E2E Prod Completed'
                '''
            }

            post{
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])        
                }
            }
        }


        stage('Deploy Prod') {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                    // args '-u root:root' - Is a bad idea as mounted with different username
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'https://dreamy-pasca-c43c37.netlify.app'
            }

            steps{
                sh'''
                node --version
                npm install netlify-cli                                          # for Deploy
                netlify --version                                                # for Deploy
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"        # for Deploy
                netlify status                                                   # for Deploy
                netlify deploy --dir=build --no-build --prod                     # for Deploy
                sleep 5                                                          # need time for Netlify to respond
                echo 'Deploy Prod Completed'

                npx playwright test  --reporter=html                             # for E2E
                echo 'E2E Prod Completed'                                        # for Deploy
                '''
            }

            post{
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])        
                }
            }
        }
    }
}