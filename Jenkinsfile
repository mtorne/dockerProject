pipeline {

  agent any
      environment {
        PROJECT_ID = 'emerald-trilogy-313008'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'europe-west6-a'
        CREDENTIALS_ID = 'emerald-trilogy-313008'
    }

  stages {

    stage('Checkout Source') {
      steps {
        git url:'https://github.com/mtorne/dockerProject.git', branch:'main'
      }
    }
    
      stage("Build image") {
            steps {
                script {
                    myapp = docker.build("mtorne/hellowhale:${env.BUILD_ID}")
                }
            }
        }
    
      stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('List pods') {
          steps {
            withKubeConfig([credentialsId: env.CREDENTIALS_ID]) {
              
                sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
                sh 'chmod u+x ./kubectl'
                sh 'echo $PATH'
                sh '/usr/local/bin/kubectl -version'
            }
          } 
        }

        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hellowhale:latest/hellowhale:${env.BUILD_ID}/g' hellowhale.yml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'hellowhale.yml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    


  }

}