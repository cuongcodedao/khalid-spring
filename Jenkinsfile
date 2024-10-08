pipeline {
    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN_PSW = credentials('mysql-root-login') // Assuming 'mysql-root-login' contains password
    }
    stages {
        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing Image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t cuongcodedao/springboot .'
                    sh 'docker push cuongcodedao/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network inspect dev || docker network create dev'
                sh 'docker container stop khalid-mysql || echo "this container does not exist"'
                sh 'docker container prune -f'
                sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh "docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example -d mysql:8.0"
                sh 'sleep 20'
                sh "docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script" // Ensure correct path
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull cuongcodedao/springboot'
                sh 'docker container stop khalid-springboot || echo "this container does not exist"'
                sh 'docker container prune -f'

                sh 'docker container run -d --rm --name khalid-springboot -p 8081:8080 --network dev cuongcodedao/springboot'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
