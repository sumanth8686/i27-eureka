// This Jenkinsfile is for the eureka deployment
pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    APPLICATION_NAME = "eureka"
    POM_VERSION = readMavenPom().getVersion()
    POM_PACKAGING = readMavenPom().getPackaging()
  }
  tools {
    maven 'Maven-3.8.8'
    jdk 'JDK-17'
  }
  stages {
    stage ('Build') {
      steps {
        echo "buiding the ${env.APPLICATION_NAME} application"
        sh 'mvn clean package -DskipTests=true'  
        
      }
    }
    stage ('unit-tests') {
      steps {
        echo "Performing unit test for ${env.APPLICATION_NAME} application"
        sh 'mvn test'
      }
      post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
      }
    }

    stage ('Docker-Format') {
      steps {
        echo "JAR Source: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
      }
    }

    stage ('test') {
      steps {
        echo "hello_world"
      }
    }
    
    
  }
}
