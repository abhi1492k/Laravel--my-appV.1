pipeline {
    agent any

    environment {
        DEPLOY_PATH = '/var/www/my-appV.1'
        GIT_URL = 'https://github.com/abhi1492k/Laravel--my-appV.1.git'
        BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "main", url: "https://github.com/abhi1492k/Laravel--my-appV.1.git"
            }
        }

        stage('PHP Lint') {
            steps {
                sh 'find . -path ./vendor -prune -o -type f -name "*.php" -print | xargs -P 4 -n 1 php -l'
            }
        }

        stage('Dependencies') {
            steps {
                sh 'composer install --no-interaction --prefer-dist --optimize-autoloader'
                sh 'cp .env.example .env'
                sh 'php artisan key:generate'
            }
        }

        stage('Clear Cache') {
            steps {
                sh 'php artisan cache:clear'
                sh 'php artisan config:clear'
                sh 'php artisan route:clear'
                sh 'php artisan view:clear'
            }
        }

        stage('Test') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }

        stage('Build Assets') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Deploy') {
            steps {
                // Copy files to deploy path
                sh "rsync -avz --exclude='.git' --exclude='node_modules' --exclude='tests' ./ /var/www/my-appV.1/"

                // Permissions
                sh "chown -R www-data:www-data /var/www/my-appV.1"
                sh "find /var/www/my-appV.1 -type d -exec chmod 755 {} \\;"
                sh "find /var/www/my-appV.1 -type f -exec chmod 644 {} \\;"
                sh "chmod -R 775 /var/www/my-appV.1/storage"
                sh "chmod -R 775 /var/www/my-appV.1/bootstrap/cache"

                // Laravel post-deploy
                sh "php /var/www/my-appV.1/artisan config:cache"
                sh "php /var/www/my-appV.1/artisan route:cache"
                sh "php /var/www/my-appV.1/artisan view:cache"
                sh "php /var/www/my-appV.1/artisan migrate --force"
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully.'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}
