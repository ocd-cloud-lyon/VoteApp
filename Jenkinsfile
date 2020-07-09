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
        // --network=host added to ensure that container has internet access while being build
        sh 'docker build -t ocd-cloud-lyon/result ./result --network=host'
      }
    } 
    

    stage('Build vote') {
      steps {
        // --network=host added to ensure that container has internet access while being build
        sh 'docker build -t ocd-cloud-lyon/vote ./vote --network=host'
      }
    }
    
    stage('Build worker') {
      steps {
        // --network=host added to ensure that container has internet access while being build
        sh 'docker build -t ocd-cloud-lyon/worker ./worker --network=host'
      }
    }

    stage('Build db') {
      steps {
        // --network=host added to ensure that container has internet access while being build
        sh 'docker build -t ocd-cloud-lyon/db ./db --network=host'
      }
    }
    
    stage('Build redis') {
      steps {
        // --network=host added to ensure that container has internet access while being build
        sh 'docker build -t ocd-cloud-lyon/redis ./redis --network=host'
      }
    }
    
    stage ('Scan_Prisma'){
      steps {
        script { 
          FAILED_STAGE=env.STAGE_NAME
          echo "Scan Prisma"
          Prisma_Scan_launched = 1
        }
        
        prismaCloudScanImage  ca: '',
                              cert: '',
                              dockerAddress: 'unix:///var/run/docker.sock',
                              ignoreImageBuildTime: true,
                              image: 'ocd-cloud-lyon/*',
                              key: '',
                              logLevel: 'info',
                              podmanPath: '',
                              project: '',
                              resultsFile: 'prisma-cloud-scan-results.json'

        echo "scan completed"
        script {
          Prisma_Scan_Done = 1
        }
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
    stage('Push worker image') {
      steps {
        script{
          docker.withRegistry(registry, registryCredential) {
            docker.image('ocd-cloud-lyon/worker').push('latest')
            docker.image('ocd-cloud-lyon/worker').push("${env.BUILD_NUMBER}")
          }
        } 
      }
    }

    stage('Push db image') {
      steps {
        script{
          docker.withRegistry(registry, registryCredential) {
            docker.image('ocd-cloud-lyon/db').push('latest')
            docker.image('ocd-cloud-lyon/db').push("${env.BUILD_NUMBER}")
          }
        } 
      }
    }

    stage('Push redis image') {
      steps {
        script{
          docker.withRegistry(registry, registryCredential) {
            docker.image('ocd-cloud-lyon/redis').push('latest')
            docker.image('ocd-cloud-lyon/redis').push("${env.BUILD_NUMBER}")
          }
        } 
      }
    }

    stage('deploy') {
      steps {
        script{
          FAILED_STAGE=env.STAGE_NAME
          echo "delete docker image"
        }

       
        script {
          withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig-file', namespace: '', serverUrl: '') {
            sh '/usr/local/bin/helm upgrade -i voteapp VoteApp'
          }
        }
      }
    }
  }
  post {
    always {
      script {
        if (Prisma_Scan_Done == 1){
          script {
            echo "scan lancé maintenant on pousse les resultats"        
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
            echo "published completed"
          }
        }
      }
    }
  }
}
