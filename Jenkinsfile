pipeline {
  parameters{
        string(name: 'Version', description: '', defaultValue: '', trim: true)
  }
  environment {
    dockerimagename = "pttzx/nodeapp"
    dockerImage = ""
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        echo "--- Checkout Source ---"
        dir("${WORKSPACE}/app-code") {
          git branch: "main", url: "https://github.com/WorkingDada/node-app-staging.git"
        }
      }
    }

    stage('Download and Create Configmap'){
      steps{
        echo "--- Downloading Configmap ---"
        script {
          dir("${WORKSPACE}/app-config") {
            sh "rm -f staging.properties || true"
            git branch: "main", url: "https://github.com/WorkingDada/techx-trainee-app-config.git"
          }
          withKubeConfig([credentialsId: 'KubeConfigCred', serverUrl: 'https://minikubeCA:58431']) {
            sh 'kubectl create configmap appconfig --from-file=${WORKSPACE}/app-config/staging.properties -n techx-trainee-staging -o yaml --dry-run=client | kubectl apply -f -'
          }
        }
      }
    }   

    stage('Updating .yaml') {
        steps {
            script {
                def fileContentsTemplate = "${WORKSPACE}/deploymentservice.yml"
                String fileContents = readFile(fileContentsTemplate)
                fileContents = fileContents.replaceAll("##VERSION##","${params.Version}")
                sh "rm -f ${fileContentsTemplate}"
                writeFile file: fileContentsTemplate, text: fileContents
            }
        }
    }
    stage('Deploying App to Kubernetes') {
      steps {
        script {
          echo "--- Deploying ---"
          withKubeConfig([credentialsId: 'KubeConfigCred', serverUrl: 'https://minikubeCA:58431']) {
            sh 'kubectl apply -f ${WORKSPACE}/app-code/deploymentservice.yml -n techx-trainee-staging'
          }
        }
      }
    }
  }
}
