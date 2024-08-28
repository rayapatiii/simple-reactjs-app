pipeline {
    agent any

    environment {
        NODE_HOME = '/usr/local/bin/node'
        NPM_HOME = '/usr/local/bin/npm'
        PATH = "$NODE_HOME:$NPM_HOME:$PATH"
        EC2_INSTANCE_ID = '54.88.141.71'  // Replace with your EC2 instance ID
        APP_DIR = '/var/www/reactjs-app'  // Directory on the EC2 instance where the app will be deployed
        REGION = 'us-east-1'  // Replace with your AWS region
    }

    stages {
        stage('Identify User') {
            steps {
                script {
                    def buildUser = env.BUILD_USER ?: 'System'
                    echo "Pipeline triggered by user: ${buildUser}"

                    // Since we're not checking permissions, this stage ends here.
                    // If permissions checking is needed, that typically requires additional plugins or admin privileges.
                    echo "Note: Permission checking is not included in this script."
                }
            }
        }

        stage('Checkout') {
            steps {
                // Checkout the code from the dev branch
                git branch: 'dev', url: 'https://github.com/rayapatiii/simple-reactjs-app.git'
            }
        }

        stage('Install Node.js and npm') {
            steps {
                sh '''
                # Check if Node.js is installed
                if ! command -v node > /dev/null; then
                    echo "Node.js not found. Installing..."
                    curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
                    sudo apt-get install -y nodejs
                fi

                # Check Node.js and npm versions
                node -v
                npm -v
                '''
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

                    // Upload the build.zip to the EC2 instance's /tmp directory using SSM
                    sh "aws ssm send-command --instance-ids ${EC2_INSTANCE_ID} --region ${REGION} --document-name 'AWS-RunShellScript' --comment 'Upload Build' --parameters commands='aws s3 cp s3://your-s3-bucket-name/build.zip /tmp/build.zip' --output text"

                    // Deploy the application to the specified directory on the EC2 instance
                    sh "aws ssm send-command --instance-ids ${EC2_INSTANCE_ID} --region ${REGION} --document-name 'AWS-RunShellScript' --comment 'Deploy ReactJS App' --parameters commands='sudo mkdir -p ${APP_DIR} && sudo rm -rf ${APP_DIR}/* && sudo apt-get install unzip -y && sudo unzip /tmp/build.zip -d ${APP_DIR}' --output text"

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
