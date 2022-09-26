pipeline {
  agent none
  stages {
    stage('worker build') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Compiling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker test') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Unit Tests on worker app'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Packaging on worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker docker-package') {
      agent any
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://docker.home.georcon.com/', 'docker.home.georcon.com-secret') {
            def workerImage = docker.build("georcon/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('result build') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compiling result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result test') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on result app'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('result docker-package') {
      agent any
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Packaging result app with docker'
        script {
          docker.withRegistry('https://docker.home.georcon.com/', 'docker.home.georcon.com-secret') {
            def workerImage = docker.build("georcon/result:v${env.BUILD_ID}", "./result")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('vote build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compiling vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote docker-package') {
      agent any
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Packaging vote app with docker'
        script {
          docker.withRegistry('https://docker.home.georcon.com', 'docker.home.georcon.com-secret') {
            def voteImage = docker.build("georcon/vote:v${env.BUILD_ID}", "./vote")
            voteImage.push()
            voteImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('Deploy to dev') {
      steps {
        sh 'docker compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Pipeline for worker is complete...'
      slackSend(channel: 'testing-jenkins-integration', message: "Building... - ${env.JOB_NAME}")
    }

    failure {
      echo 'Something bad happened...'
      slackSend(channel: 'testing-jenkins-integration', message: 'Build Failed - ${env.JOB_NAME}')
    }

    success {
      slackSend(channel: 'testing-jenkins-integration', message: "Build Succeeded! - ${env.JOB_NAME}")
    }

  }
}