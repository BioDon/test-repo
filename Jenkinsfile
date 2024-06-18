pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_REGION = 'eu-central-1'
        S3_BUCKET_NAME = 'your-s3-bucket-name'
        CLOUDFRONT_DISTRIBUTION_ID = 'your-cloudfront-distribution-id'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone the repository
                git 'https://github.com/BioDon/deploy-website-s3-cloudfront-codepipeline.git'
            }
        }

        stage('Install AWS CLI') {
            steps {
                // Install AWS CLI if not already installed
                sh '''
                    if ! command -v aws &> /dev/null; then
                        echo "AWS CLI not found. Installing..."
                        curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
                        sudo installer -pkg AWSCLIV2.pkg -target /
                    fi
                '''
            }
        }

        stage('Deploy to S3') {
            steps {
                // Sync the website files to the S3 bucket
                sh '''
                    aws s3 sync . s3://$S3_BUCKET_NAME --delete
                '''
            }
        }

        stage('Invalidate CloudFront Cache') {
            steps {
                // Invalidate the CloudFront cache to reflect the latest changes
                sh '''
                    aws cloudfront create-invalidation --distribution-id $CLOUDFRONT_DISTRIBUTION_ID --paths "/*"
                '''
            }
        }
    }

    post {
        always {
            // Clean up workspace after the build
            cleanWs()
        }
    }
}
