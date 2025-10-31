pipeline {
  agent any

  environment {
    APP_ID = "3DNklqpUFcPMbbacw9IQl" 
  }

  parameters {
    string(name: 'DEPLOY_URL_CRED_ID', defaultValue: 'DEPLOY_URL', description: 'Credentials ID for the deployment URL (Secret Text)')
    string(name: 'DEPLOY_KEY_CRED_ID', defaultValue: 'DEPLOY_KEY', description: 'Credentials ID for the deployment API key (Secret Text)')
    string(name: 'VITE_CANDIDATES_ENDPOINT', defaultValue: 'VITE_CANDIDATES_ENDPOINT', description: 'Endpoint for candidates API used by the frontend (exported into .env)')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup .env') {
      steps {
        sh "echo 'VITE_CANDIDATES_ENDPOINT=${params.VITE_CANDIDATES_ENDPOINT}' > .env"
        sh 'echo ".env created with VITE_CANDIDATES_ENDPOINT"'
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'node -v && npm --version && (npm ci || npm install)'
      }
    }

    // Skipping explicit install of test libraries because they are already in devDependencies

    stage('Run tests') {
      steps {
        sh 'npm test -- --run'
      }
    }
  }  
  post {
    success {
      echo "✅ Tests passed, triggering deployment API..."
      withCredentials([
        string(credentialsId: params.DEPLOY_URL_CRED_ID, variable: 'DEPLOY_URL'),
        string(credentialsId: params.DEPLOY_KEY_CRED_ID, variable: 'DEPLOY_KEY')
      ]) {
        sh '''
          json_payload=$(printf '{"applicationId":"%s"}' "$APP_ID")
          curl -fS -X POST \
            "$DEPLOY_URL" \
            -H 'accept: application/json' \
            -H 'Content-Type: application/json' \
            -H "x-api-key: $DEPLOY_KEY" \
            --data-binary "$json_payload" \
            -w "\nHTTP %{http_code}\n"
        '''
      }
      // Send success email notification
      mail to: 'xaioene@gmail.com',
           subject: "Jenkins Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) has succeeded.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Deployment API was triggered successfully.

Regards,
Jenkins
"""
    }

    failure {
      echo "❌ Pipeline failed, sending error email..."
      mail to: 'xaioene@gmail.com',
           subject: "Jenkins Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) has failed.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Please check the console output for details.

Regards,
Jenkins
"""
    }
  }
}