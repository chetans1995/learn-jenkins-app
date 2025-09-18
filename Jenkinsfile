pipeline {
    agent any

    environment{
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        AWS_DEFAULT_REGION = 'eu-north-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Production'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearJenkinsApp-TaskDefinition-Production'
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


        stage('Build Docker Image'){
            steps{
                sh 'docker build -t myjenkinsapp .'
            }
        }


        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    yum install jq -y
                    echo "Hello S3!" > index.html
                    LATEST_TD_REV=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    echo $LATEST_TD_REV

                    aws ecs update-service \
                        --cluster $AWS_ECS_CLUSTER \
                        --service $AWS_ECS_SERVICE_PROD \
                        --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REV
                        aws ecs wait services-stable \
                            --cluster $AWS_ECS_CLUSTER \
                            --services $AWS_ECS_SERVICE_PROD
                '''
                }
            }
        }

    }
}