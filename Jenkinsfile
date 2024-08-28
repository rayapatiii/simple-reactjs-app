pipeline {
    agent any

    environment {
        NODE_HOME = '/usr/bin/node'
        NPM_HOME = '/usr/bin/npm'
        PATH = "$NODE_HOME:$NPM_HOME:$PATH"
        APP_DIR = '/var/www/reactjs-app'  // Directory where the app will be deployed
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
                    // Update the homepage in package.json
                    sh '''
                    sed -i 's#"homepage": ".*"#"homepage": "/"#' package.json
                    cat package.json
                    '''
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
                    // Clean the target directory if it already exists
                    sh """
                    mkdir -p ${APP_DIR}
                    rm -rf ${APP_DIR}/*
                    """

                    // Copy the build files to the target directory
                    sh """
                    cp -r build/* ${APP_DIR}/
                    """
                }
            }
        }

        stage('Configure Nginx') {
            steps {
                script {
                    // Create an Nginx configuration for the React app
                    sh """
                    bash -c "cat > /etc/nginx/sites-available/reactjs-app" <<EOL
server {
    listen 80;
    server_name localhost;

    root ${APP_DIR};
    index index.html;

    location / {
        try_files \\$uri /index.html;
    }
}
EOL
                    """

                    // Enable the site and restart Nginx
                    sh """
                    ln -sf /etc/nginx/sites-available/reactjs-app /etc/nginx/sites-enabled/
                    nginx -t && systemctl restart nginx
                    """
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
