pipeline {
    agent { label 'aws_pub' }

    environment {
        // Assuming these are not sensitive and static
        REDIS_PORT = '6379'
        RDS_PORT = '3306'
    }

    stages {
        stage('Preparation') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shrookmuhamed/Jenkins_project.git'
            }
        }
        stage('Build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh  """
                        docker build -t ${USER}/nodeapp:latest -f Dockerfile .
                        docker login -u ${USER} -p ${PASS}
                        docker push ${USER}/nodeapp:latest
                        """
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'docker', passwordVariable: 'PASS', usernameVariable: 'USER'),
                    string(credentialsId: 'rds_hostname', variable: 'RDS_HOSTNAME'),
                    string(credentialsId: 'rds_username', variable: 'RDS_USERNAME'),
                    string(credentialsId: 'rds_password', variable: 'RDS_PASSWORD'),
                    string(credentialsId: 'redis_hostname', variable: 'REDIS_HOSTNAME')
                ]) {
                    sh  """
                        docker login -u ${USER} -p ${PASS}
                        docker run -d -p 3001:3000 \
                            -e RDS_HOSTNAME= 'mydb' \
                            -e RDS_USERNAME='master' \
                            -e RDS_PASSWORD='yK_U6F36T6BwtIDF' \
                            -e RDS_PORT='${RDS_PORT}' \
                            -e REDIS_HOSTNAME='cluster-redis' \
                            -e REDIS_PORT='${REDIS_PORT}' \
                            ${USER}/nodeapp:latest
                        """
                }
            }
        }
        
    }
}
