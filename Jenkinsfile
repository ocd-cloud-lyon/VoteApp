 pipeline {
    options {
        // Build auto timeout
        timeout(time: 60, unit: 'MINUTES')
    }
     // Some global default variables
    environment {
    registry = "https://573329840855.dkr.ecr.eu-west-3.amazonaws.com"
    registryCredential = 'ecr:eu-west-3:aws-ecr-credential'
        dockerImage = ''
        NomProjet = 'VoteApp'
        NameSpace = 'VoteApp-NS'
    //DeployName = 'VoteApp-NS'
    //ServiceName = 'hello-you-svc'
    RuningImageBuild = 0
    TargetImageBuild = 0
    }

    agent any

     stages {
        stage('Clone repository') {
            /* Let's make sure we have the repository cloned to our workspace */
             steps {
                //checkout scm
              script{
                  FAILED_STAGE=env.STAGE_NAME
                  echo "Clone repository"
              }
              git credentialsId: 'github-credential', url: 'https://github.com/ocd-cloud-lyon/VoteApp/'
             }

        }   
        stage('Build result') {
          steps {
            sh 'docker build -t ocd-cloud-lyon/result ./result --network=host'
          }
        } 
        stage('Build vote') {
          steps {
            sh 'docker build -t ocd-cloud-lyon/vote ./vote --network=host'
          }
        }
        stage('Build worker') {
          steps {
            sh 'docker build -t ocd-cloud-lyon/worker ./worker --network=host'
          }
        }
    stage('Push result image') {
      steps {
        script{
          docker.withRegistry(registry, registryCredential) {
            docker.image('ocd-cloud-lyon/result').push('latest')
            docker.image('ocd-cloud-lyon/result').push("${env.BUILD_NUMBER}")
                }
        }
        
      }
    }
    stage('Push vote image') {
      steps {
        script{
          docker.withRegistry(registry, registryCredential) {
            docker.image('ocd-cloud-lyon/vote').push('latest')
            docker.image('ocd-cloud-lyon/vote').push("${env.BUILD_NUMBER}")
                }
        }
        
      }
    }
      /*stage('Push vote image') {
      when {
        branch 'master'
      }
      steps {
        withDockerRegistry(credentialsId: 'dockerbuildbot-index.docker.io', url:'') {
          sh 'docker push dockersamples/vote'
        }
      }
    }*/
    /*stage('Push worker image') {
      when {
        branch 'master'
      }
      steps {
        withDockerRegistry(credentialsId: 'dockerbuildbot-index.docker.io', url:'') {
          sh 'docker push dockersamples/worker'
        }
      }
    }*/
  }
}
