pipeline {
    agent any

    environment {
        NODE_HOME = '/usr/bin/node'
        NPM_HOME = '/usr/bin/npm'
        PATH = "$NODE_HOME/bin:$NPM_HOME/bin:$PATH"
        EC2_INSTANCE_ID = 'i-078583d81293b3267'  // Replace with your actual EC2 instance ID
        APP_DIR = '/var/www/reactjs-app'  // Directory on the EC2 instance where the app will be deployed
        REGION = 'us-east-1'  // Replace with your AWS region
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from the dev branch
                git branch: 'dev', url: 'https://github.com/rayapatiii/simple-reactjs-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install npm dependencies
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                // Build the React application
                sh 'npm run build'
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    // Zip the build files
                    sh 'zip -r build.zip build'

                    // Upload the build.zip to the EC2 instance using SSM
                    sh "aws ssm send-command --instance-ids ${EC2_INSTANCE_ID} --region ${REGION} --document-name 'AWS-RunShellScript' --comment 'Deploy ReactJS App' --parameters commands='sudo mkdir -p ${APP_DIR} && sudo rm -rf ${APP_DIR}/* && sudo apt-get install unzip -y && sudo unzip /tmp/build.zip -d ${APP_DIR}' --output text"

                    // Transfer the build.zip file to the EC2 instance's /tmp directory using SSM
                    sh "aws ssm send-command --instance-ids ${EC2_INSTANCE_ID} --region ${REGION} --document-name 'AWS-RunShellScript' --comment 'Upload Build' --parameters commands='aws s3 cp s3://your-s3-bucket-name/build.zip /tmp/build.zip' --output text"

                    // Start the React app on port 3000 using a background process
                    sh "aws ssm send-command --instance-ids ${EC2_INSTANCE_ID} --region ${REGION} --document-name 'AWS-RunShellScript' --comment 'Start ReactJS App' --parameters commands='cd ${APP_DIR} && npm install -g serve && nohup serve -s . -l 3000 > /dev/null 2>&1 &' --output text"
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed!'
        }
    }
}
