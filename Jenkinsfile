pipeline {
    agent {
        label 'k8s-slave'
    }
    environment {
        APPLICATION_NAME = "Eureka"
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    stages {
        stage ('Build') {
            // application build happens here
            steps{
                echo "Building the ${env.APPLICATION_NAME} application"
                sh "mvn clean package -DskipTests=true"
                 
            }
        }
        stage ('Unit Tests') {
            steps {
                echo "Performing Unit tests for ${env.APPLICATION_NAME} application"
                sh "mvn test"
            }
        }
    }
}
