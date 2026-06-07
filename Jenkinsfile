pipeline {
    agent any
    
    tools {
        maven 'Maven'
    }

    environment {
        SONARQUBE = 'SonarQube'
        NEXUS_URL = 'http://172.31.85.18:8081'
        APP_SERVER = '172.31.90.5'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/csjeevan11/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests -Dnohttp.check.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=petclinic \
                    -Dsonar.host.url=http://172.31.88.60:9000
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload To Nexus') {
            steps {
                sh 'mvn deploy -DskipTests -Dnohttp.check.skip=true'
            }
        }

        stage('Deploy To App Server') {
            steps {
                sshagent(['app-server-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$APP_SERVER "
                    pkill -f petclinic || true
                    wget -O app.jar \
                    $NEXUS_URL/repository/maven-releases/org/springframework/samples/spring-petclinic/3.5.0/spring-petclinic-3.5.0.jar
                    nohup java -jar app.jar \
                    > app.log 2>&1 &
                    "
                    '''
                }
            }
        }

    }

    post {
        success {
            echo 'Application deployed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
