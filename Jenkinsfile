pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "eazydia-staging"
        PRODUCTION = "eazydia-prod"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Run container based on built image') {
            agent any
            steps {
                sh '''
                docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT=5000 ${IMAGE_NAME}:${IMAGE_TAG}
                sleep 5s
                '''
            }
        }

        stage('Test image') {
            agent any
            steps {
                sh '''
                curl http://172.18.0.1 | grep -q "Hello world!"
                '''
            }
        }

        stage('Clean container') {
            agent any
            steps {
                sh '''
                docker stop ${IMAGE_NAME}
                docker rm ${IMAGE_NAME}
                '''
            }
        }

        stage('Deploy to Heroku Staging') {
            when {
                branch 'master'
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                sh '''
                heroku container:login
                heroku git:remote -a $STAGING || heroku create $STAGING
                heroku container:push -a $STAGING web
                heroku container:release -a $STAGING web
                '''
            }
        }

        stage('Deploy to Heroku Production') {
            when {
                branch 'master'
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                sh '''
                heroku container:login
                heroku git:remote -a $PRODUCTION || heroku create $PRODUCTION
                heroku container:push -a $PRODUCTION web
                heroku container:release -a $PRODUCTION web
                '''
            }
        }
    }
}
