pipeline {
    agent any

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
        /*
        stage('Build') {
            agent{
                docker{
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
        */
        stage('Test') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh'''
                echo "Initating Test...13"
                mkdir -p test-results
                test -f build/index.html
                npm test
                '''
            }
        }
                stage('E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    // args '-u root:root' - Is a bad idea as mounted with different username

                }
            }
            steps{
                sh'''
                npm install serve # -g is for global dependency
                node_modules/.bin/serve -s build
                npx playwright test
                '''
            }
        }
    }
    post{
        always {
            junit 'test-results/junit.xml'
        }
    }
}
