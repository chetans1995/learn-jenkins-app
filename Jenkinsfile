pipeline {
    agent any

    environment{
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        AWS_DEFAULT_REGION = 'eu-north-1'
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

        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }


            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    echo "Hello S3!" > index.html
                    aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                    aws ecs update-service \
                        --cluster LearnJenkinsApp-Cluster-Production \
                        --service LearnJenkinsApp-Service-Prod \
                        --task-definition LearJenkinsApp-TaskDefinition-Production:4
                '''
                }
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
    }
}