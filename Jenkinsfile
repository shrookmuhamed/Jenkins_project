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
                        docker run -d -p 3002:3000 \
                            -e RDS_HOSTNAME='${RDS_HOSTNAME}' \
                            -e RDS_USERNAME='${RDS_USERNAME}' \
                            -e RDS_PASSWORD='${RDS_PASSWORD}' \
                            -e RDS_PORT='${RDS_PORT}' \
                            -e REDIS_HOSTNAME='${REDIS_HOSTNAME}' \
                            -e REDIS_PORT='${REDIS_PORT}' \
                            ${USER}/nodeapp:latest
                        """
                }
            }
        }
        stage('Debug Environment Variables') {
            steps {
                script {
                    echo "RDS_HOSTNAME: ${env.RDS_HOSTNAME}"
                    echo "RDS_USERNAME: ${env.RDS_USERNAME}"
                    echo "REDIS_HOSTNAME: ${env.REDIS_HOSTNAME}"
                    // Add additional variables as necessary
                }
            }
        }
    }
}
