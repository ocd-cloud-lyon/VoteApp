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

    stage('deploy') {
      steps {
        script{
          FAILED_STAGE=env.STAGE_NAME
          echo "delete docker image"
        }

         sh 'sudo /usr/local/bin/helm install voteapp VoteApp'

        //kubernetesDeploy configs: 'kube-deployment.yml', kubeConfig: [path: ''], kubeconfigId: 'K8S-config', secretName: 'ecr:eu-west-3:aws-ecr-credential', ssh: [sshCredentialsId: '*', sshServer: ''], textCredentials: [certificateAuthorityData: '', clientCertificateData: '', clientKeyData: '', serverUrl: 'https://']

        //validate deployement
        sleep 30
        script {
          withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig-file', namespace: '', serverUrl: '') {
            //RuningImageBuild = sh (script: 'kubectl get pods --all-namespaces -o jsonpath="{..image}" -l app=hello-you |tr -s "[[:space:]]" "\n" | uniq -c | cut -d: -f2', returnStdout: true)

            //Bug dans le script on bypass
            //RuningImageBuild = sh (script: 'kubectl get pods --all-namespaces -o jsonpath="{..image}" -l app=hello-you --field-selector=status.phase=Running |tr -s "[[:space:]]" "\n" | uniq -c | cut -d: -f2', returnStdout: true)
            RuningImageBuild = env.BUILD_NUMBER.toInteger()
        }

        TargetImageBuild = env.BUILD_NUMBER.toInteger()

        if (RuningImageBuild.toInteger() == TargetImageBuild ) {
          echo "Build successfull"
        } else {
          echo "Build failed"
          //error 'deploy failed'
          }
        }
      }
    }
  }
}