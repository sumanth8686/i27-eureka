pipeline {
    agent {
        label 'k8s-slave'
    }
    environment {
        APPLICATION_NAME: "Eureka"
    }
    tools {
        Maven 'Maven-3.8.8'
        Jdk 'JDK-17'
    }
    stages {
        stage ('Build') {
            // application build happens here
            steps{
                echo "Building the ${env.APPLICATION_NAME} application"
                sh "mvn clean package"
                 
            }
        }
    }
}
