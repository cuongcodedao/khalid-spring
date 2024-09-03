pipeline {

    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
        // Assuming 'mysql-root-login' has a password stored as 'password'
        MYSQL_ROOT_PASSWORD = "${MYSQL_ROOT_LOGIN_PSW}"
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {
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
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop cuongcodedao-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune'
                sh 'docker volume rm cuongcodedao-mysql-data || echo "no volume"'

                sh "docker run --name cuongcodedao-mysql --rm --network dev -v cuongcodedao-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} -e MYSQL_DATABASE=db_example -d mysql:8.0"
                sh 'sleep 20'
                sh "docker exec -i cuongcodedao-mysql mysql --user=root --password=${MYSQL_ROOT_PASSWORD} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull cuongcodedao/springboot'
                sh 'docker container stop cuongcodedao-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune'

                sh 'docker container run -d --rm --name cuongcodedao-springboot -p 8081:8080 --network dev cuongcodedao/springboot'
            }
        }
 
    }
    post {
        always {
            cleanWs()
        }
    }
}
