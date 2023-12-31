pipeline {

    agent any

    tools { 
        maven 'maven-jenkins-lab'
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    echo credentialsId
                    sh 'docker build -t minhdang2001/mavenJenkinsLab .'
                    sh 'docker push minhdang2001/mavenJenkinsLab'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create bridge || echo "this network exists"'
                sh 'docker container stop khalid-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh "docker run --name mysql --rm --network bridge -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Backend to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull minhdang2001/mavenJenkinsLab'
                sh 'docker container stop mavenJenkinsLab || echo "this container does not exist" '
                sh 'docker network create bridge || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name mavenJenkinsLab -p 8081:8080 --network bridge minhdang2001/mavenJenkinsLab'
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
