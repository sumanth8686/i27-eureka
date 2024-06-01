// This Jenkinsfile is for the eureka deployment
pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    APPLICATION_NAME = "eureka"
    POM_VERSION = readMavenPom().getVersion()
    POM_PACKAGING = readMavenPom().getPackaging()
    DOCKER_HUB = "docker.io/sumanth9677"
    DOCKER_CREDS = credentials('sumanth9677_docker_creds')
    SONAR_URL = "http://34.125.222.46:9000"
    SONAR_TOKEN = credentials('sonar_creds')
    
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

    stage ('sonar') {
        steps {
            sh """
            echo "starting sonar scan"
            mvn clean verify sonar:sonar \
             -Dsonar.projectKey=i27-eureka \
             -Dsonar.host.url=${env.SONAR_URL} \
             -Dsonar.login=${SONAR_TOKEN}
            """
        }
    }

    stage ('Docker Format') {
      steps {
        echo "Actual format: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
      }
    }

    stage ('test') {
      steps {
        echo "Custom format: ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
      }
    }
    
    stage ('docker build') {
        steps {
            sh """
              ls -la
              cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
              ls -la ./.cicd
              echo "*********************** Build Docker Image *******************************************"
              docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
              echo "*********************** Docker login *******************************************"
              docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
              echo "*********************** Docker push *******************************************"
              docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
              """
        }
    }
    
    
  }
}
