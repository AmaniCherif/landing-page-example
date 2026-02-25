pipeline {
  agent any

  environment {
    ALWAYS_HOST = 'ssh-amani.alwaysdata.net'
    ALWAYS_PATH = '~/www'
    SSH_CRED_ID = 'alwaysdata-ss' 
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/AmaniCherif/landing-page-example.git'
      }
    }

    stage('CI - Validate source') {
      steps {
        sh '''
          set -e
          ls -la
          test -f index.html
          grep -n -i '<!DOCTYPE html' index.html
        '''
      }
    }

    stage('CD - Deploy to AlwaysData (SCP)') {
      steps {
        withCredentials([sshUserPrivateKey(
          credentialsId: "${SSH_CRED_ID}",
          keyFileVariable: 'SSH_KEY',
          usernameVariable: 'SSH_USER'
        )]) {
          sh '''
            set -euo pipefail

            mkdir -p ~/.ssh
            chmod 700 ~/.ssh
            ssh-keyscan -H "$ALWAYS_HOST" >> ~/.ssh/known_hosts

            SSH_OPTS="-o IdentitiesOnly=yes -o StrictHostKeyChecking=yes -i $SSH_KEY"

            ssh-keygen -y -f "$SSH_KEY" >/dev/null

            ssh $SSH_OPTS "$SSH_USER@$ALWAYS_HOST" "mkdir -p $ALWAYS_PATH"
            scp $SSH_OPTS -p index.html "$SSH_USER@$ALWAYS_HOST:$ALWAYS_PATH/"
          '''
        }
      }
    }
  }
}
