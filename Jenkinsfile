pipeline {
  agent any
  environment {
    VERSION = '1.1'
    IMAGE = "mshoai306/cw2-server:${VERSION}"
  }

  stages {
    stage('Clone') {
      steps {
        git branch: 'main', url: 'git@github.com:shoaibeagledevs/devops-cw2.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${IMAGE}", "--no-cache .")
        }
      }
    }

    stage('Test Docker Image') {
      steps {
        sh 'docker run -d -p 8081:8081 --name test_container $IMAGE'
        sh 'sleep 5'
        sh 'curl -s http://localhost:8081 || echo "Test failed"'
        sh 'docker rm -f test_container'
      }
    }

    stage('Push to DockerHub') {
      steps {
        withDockerRegistry([credentialsId: 'dockerhub-creds', url: 'https://index.docker.io/v1/']) {
          sh 'docker push $IMAGE'
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sshagent (credentials: ['prod-server-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@54.147.169.172 \
            "kubectl set image deployment/cw2-deploy cw2-server=mshoai306/cw2-server:1.1"
          '''
        }
      }
    }
  }
}
