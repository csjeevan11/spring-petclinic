
pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy JAR to Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'tomcat-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        echo "Copying JAR..."
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no \
                        target/spring-petclinic-*.jar \
                        $SSH_USER@172.31.44.10:/home/ubuntu/app.jar

                        echo "Restarting Application..."
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                        $SSH_USER@172.31.44.10 \
                        "nohup java -jar /home/ubuntu/app.jar --server.port=8081 > /home/ubuntu/app.log 2>&1 &"
                    '''
                }
            }
        }
    }
}
