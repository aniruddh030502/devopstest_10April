pipeline {

  agent any

  environment {
    REGISTRY = 'aniruddh030502/devopsjavaimg'
    IMAGE_TAG = "${GIT_COMMIT.take(7)}"
  }

  stages {

    stage('Build & Test') {
      steps {
        script {
          sh 'mvn clean package -DskipTests=false'
        }
      }
    }

    stage('Docker Build & Push') {
      when {
        branch 'develop'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          script {
            sh """
              docker build -t $REGISTRY:$IMAGE_TAG .
              echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
              docker push $REGISTRY:$IMAGE_TAG
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      when {
        branch 'develop'
      }
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-credential-id', variable: 'KUBECONFIG')]) {
          script {
            sh """
              sed -i 's|{{IMAGE}}|$REGISTRY:$IMAGE_TAG|' deployment.yaml
              kubectl apply -f deployment.yaml
              kubectl apply -f service.yaml
            """
          }
        }
      }
    }

  }

}
