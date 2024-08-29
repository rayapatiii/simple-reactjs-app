pipeline {
    agent any

    environment {
        NODE_HOME = '/usr/bin/node'
        NPM_HOME = '/usr/bin/npm'
        PATH = "$NODE_HOME:$NPM_HOME:$PATH"
        APP_DIR = '/var/www/reactjs-app'  // Directory on the EC2 instance where the app will be deployed
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from the dev branch
                git branch: 'dev', url: 'https://github.com/rayapatiii/simple-reactjs-app.git'
            }
        }

        stage('Update Homepage in package.json') {
            steps {
                script {
                    // Corrected sed command to update the homepage in package.json
                    sh 'sed -i "s/\\"homepage\\": \\".*\\"/\\"homepage\\": \\"/\\"/" package.json'
                    sh 'cat package.json'
                }
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

        stage('Deploy to Local Directory') {
            steps {
                script {
                    // Create the app directory and remove old build files
                    sh 'mkdir -p ${APP_DIR}'
                    sh 'rm -rf ${APP_DIR}/build'

                    // Copy new build files to the app directory
                    sh 'cp -r build/* ${APP_DIR}/'
                }
            }
        }

        stage('Configure Nginx') {
            steps {
                script {
                    // Configuration for Nginx to serve the React app
                    sh '''
                        sudo tee /etc/nginx/sites-available/reactjs-app << EOF
                        server {
                            listen 80;
                            server_name _;

                            root /var/www/reactjs-app;
                            index index.html index.htm;

                            location / {
                                try_files $uri /index.html;
                            }
                        }
                        EOF
                    '''
                    // Create a symlink to enable the Nginx configuration
                    sh 'sudo ln -sf /etc/nginx/sites-available/reactjs-app /etc/nginx/sites-enabled/'
                    // Test Nginx configuration
                    sh 'sudo nginx -t'
                    // Restart Nginx
                    sh 'sudo systemctl restart nginx'
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
