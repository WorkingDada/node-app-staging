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
        git branch: 'main', url: 'https://github.com/WorkingDada/node-app.git'
      }
    }
    stage('Updating .yaml') {
        steps {
            script {
                def fileContentsTemplate = "${WORKSPACE}/deploymentservice.yml"
                String fileContents = readFile(fileContentsTemplate)
                fileContents = fileContents.replaceAll("##VERSION##","params.Version")
                sh "rm -f ${fileContentsTemplate}"
                writeFile file: fileContentsTemplate, text: fileContents
            }
        }
    }
    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
          
        }
      }
    }
  }
}
