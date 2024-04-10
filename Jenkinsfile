pipeline {
    agent any
    
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
        REGION ="eu-west-3"
        AWS_S3_BUCKET = "mm167-ebs"
        ARTIFACT_NAME = "demo-boot.jar"
        GIT_URL = "https://github.com/mm167/demo-boot.git"
        AWS_EB_APP_NAME = "isika-pipeline"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Isika-pipeline-env"
    }

    stages {
        stage('AWS cli') {
            steps {
                sh "aws --version"
                
                //set region
                sh "aws configure set region $REGION"
                
                // list s3 bucket
                sh "aws s3 ls"
            }
        }
        
        stage('Clone projet') {
            steps {
                git(
                    url: "$GIT_URL",
                    branch: "main"
                )
            }
        }
        
        stage('Construction du jar') {
            steps {
                sh "mvn clean package -DskipTests=true"
                archive 'target/*.jar'
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "mvn test"
            }
            
            post {
                success {
                    sh "aws configure set region $REGION"
                    sh "aws s3 cp ./target/*.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
                }
            }
        }
        
        stage('Deploy to aws') {
            steps {

                sh "aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME"
                
                sh "aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME  --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION"
            }
        }      
        
    }
}
