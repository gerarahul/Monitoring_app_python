pipeline {
  agent any

  environment {
    NAME = "monitoring-app"
    VERSION = "v1.${env.BUILD_ID}"
    IMAGE_REPO = "rgera0901"
    GITHUB_TOKEN = credentials('GITHUB_TOKEN')
  }
  
  stages {
    stage("Git Checkout") {
       steps{
           git branch: 'main', url: 'https://github.com/gerarahul/Monitoring_app_python.git'
           echo "checkout successfull"
      }
    }
    
    stage('Unit Tests') {
      steps {
        echo 'Implement unit tests if applicable.'
        echo 'This stage is a sample placeholder'
      }
    }

    stage('Build Image') {
      steps {
        sh "docker build -t ${NAME} ."
        sh "docker tag ${NAME}:latest ${IMAGE_REPO}/${NAME}:${VERSION}"
      }
    }

    stage('Push Image') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh "docker push ${IMAGE_REPO}/${NAME}:${VERSION}"
        }
      }
    }

 // in this step we are cloning the feature branch of manifest repo not the main branch which will be merge into main after PR approval"
    stage('Clone/Pull k8s manifests Repo') {
      steps {
        script {
          if (fileExists('Monitoring-app')) {

            echo 'Cloned repo already exists - Pulling latest changes'

            dir("Monitoring-app") {
              sh 'git pull'
            }

          } else {
            echo 'Repo does not exists - Cloning the repo'
            sh 'git clone -b feature-branch https://github.com/gerarahul/gitops-argocd.git'
          }
        }
      }
    }
    
// In below stage we are modifying the image in deployment.yaml file
    stage('Update Manifest') {
      steps {
        dir("gitops-argocd/Monitoring-app") {
          sh 'sed -i "s|image:.*|image: ${IMAGE_REPO}/${NAME}:${VERSION}|g" deployment.yaml'
          sh 'cat deployment.yaml'
        }
      }
    }

    stage('Commit & Push') {
      steps {
        dir("gitops-argocd/jenkins-demo") {
          sh 'git config --global user.name "Rahul"'
          sh 'git config --global user.email "rahul@testemail.com"'
          sh 'git remote set-url origin https://$GITHUB_TOKEN@github.com/gerarahul/argocd.git'
          sh 'git checkout feature-branch'
          sh 'git add -A'
          sh "git commit -m 'Updated image version for Build - ${VERSION}'"
          sh 'git push origin feature-branch'
        }
      }
    }
    stage('Send Email Notification') {
      steps {
        emailext subject: 'Build and Deployment Complete',
                  body: 'The build and deployment process has been completed successfully. Now, you can raise the PR',
                  to: 'rgera0901@gmail.com'
      }
    }
  }
}