// This Jenkinsfile is for the eureka deployment
pipeline {
  agent {
    label 'k8s-slave'
  }

  parameters {
    choice(name: 'buildOnly', 
      choices: 'no\nyes',
      description: 'this will only build the application'
    )
    
    choice(name: 'scanOnly',
      choices: 'no\nyes',
      description: 'this will only scan the application'
    )
    
    choice(name: 'dockerPush' ,
      choices: 'no\nyes',
      description: 'this will only scan the application'
    )
    choice(name: 'deploytoDev',
      choices: 'no\nyes',
      description: 'this will only scan the application'
    )
    
    choice(name: 'deploytoTest',
      choices: 'no\nyes',
      description: 'this will only scan the application'
    )
    choice(name: 'deploytoStage',
      choices: 'no\nyes',
      description: 'this will only scan the application'
    )
    
  }


  environment {
    APPLICATION_NAME = "eureka"
    POM_VERSION = readMavenPom().getVersion()
    POM_PACKAGING = readMavenPom().getPackaging()
    DOCKER_HUB = "docker.io/sumanth9677"
    DOCKER_CREDS = credentials('sumanth9677_docker_creds')
    SONAR_URL = "http://34.125.200.191:9000/"
    SONAR_TOKEN = credentials('sonar_creds')
    
  }
  tools {
    maven 'Maven-3.8.8'
    jdk 'JDK-17'
  }
  stages {
    stage ('Build') {
      when {
        anyOf {
          expression {
            params.buildOnly == 'yes'
            //params.dockerPush == 'yes'
          }
        }
      }
      steps {
        echo "buiding the ${env.APPLICATION_NAME} application"
        sh 'mvn clean package -DskipTests=true'  
        
      }
    }
    stage ('unit-tests') {
       when {
        anyOf {
          expression {
            params.buildOnly == 'yes'
            params.dockerPush == 'yes'
          }
        }
      }
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
      when {
        expression {
          params.scanOnly == 'yes'
        }
       
      }
       steps {
         echo "Starting SonarQube with QualityGates"
         withSonarQubeEnv('SonarQube'){ // this name should be same as manage jenkins > system details
           sh """
             mvn clean verify sonar:sonar \
               -Dsonar.projectKey=i27-eureka \
               -Dsonar.host.url=${env.SONAR_URL} \
               -Dsonar.login=${SONAR_TOKEN}

         """
        }
        timeout (time: 2, unit: 'MINUTES') { // wait for 2min
          script {
            waitForQualityGate abortPipeline: true // if no respose, fail the gate
          }
         }
    }
        
  }
      


    /*stage ('Docker Format') {
      steps {
        echo "Actual format: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
      }
    }*/

    stage ('test') {
      steps {
        echo "Custom format: ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
      }
    }
    
    stage ('docker build & push') {
      when {
        anyOf {
          expression {
            params.dockerPush == 'yes'
          }
        
        }
      }
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

    stage ('deploying to dev') {
      when {
          expression {
             params.deploytoDev == 'yes'
          }
         
        }
      
          steps {
            script {
              dockerDeploy('dev', '5761', '8761').call()
              echo "************deployed to dev successfully***************"


            }
          }
        
    }
    

    stage ('deploying to test') {
      when {
        expression {
          params.deploytoTest == 'yes'
        }
          
      }
        
        steps {
          script{
            echo "*************entering into tst env*************"
            dockerDeploy('tst', '6761', '8761').call()
     }
    }
   }
    stage ('deploying to stage') {
      when {
        expression {
           params.deploytoStage == 'yes'
       }
      }
        steps {
          script{
            dockerDeploy('stage', '7761', '8761').call()
    }
   }
  }

    stage ('deploying to prod') {
      when {
        expression {
          params.deploytoProd == 'yes'

        }
        
      }
        steps {
          script{
            dockerDeploy('prod', '8761', '8761').call()
      }
    }
  }
  stage (clean) {
    steps {
      cleanWs()

    }


  }
}
}




def dockerDeploy(envDeploy, hostPort, contPort) {
      return {
      echo "*********************** Deploying to $envDeploy Env *******************************************"
      withCredentials([usernamePassword(credentialsId: 'maha_dockerenv_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
      //sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} hostname -i"

      script {
        //pull container
        sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"

        try {
          //stop container  //when there is a code change, we cannot create container with same name
          echo "stopping the container"
          sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker stop ${env.APPLICATION_NAME}-$envDeploy"

          //remove container
          echo "removing the container"
          sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker rm ${env.APPLICATION_NAME}-$envDeploy"



        } catch(err) {
          echo "Caught the error: $err"
        }



      } 
      
      
      //now we will create a container, eureka runs at 8761 port defined by developer
      //we will configure env's such that dev=>5761(host port),test=>6761,stage=>7761,prod=>8761
      //creating container
      echo "**********creating container***********"
      sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker run -d -p $hostPort:$contPort --name ${env.APPLICATION_NAME}-$envDeploy  ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
          
    } 
        

    }
}
            


        

         



       
      
