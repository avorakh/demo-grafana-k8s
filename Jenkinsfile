pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: helm
    image: alpine/helm:3.11.1
    command: ['cat']
    tty: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['cat']
    tty: true
"""
        }
    }

    environment {
        SLACK_CHANNEL = credentials('SLACK_CICD_CHANNEL') 
        SLACK_CREDENTIAL_ID = 'SLACK_TOKEN'
    }
    

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage('Create ConfigMap') {
            steps {
                container('kubectl') {
                    sh 'kubectl get nodes'
                    // sh '''
                    // kubectl create configmap demo-dashboard \
                    // --from-file=k8s-cluster-dashboard.json \
                    // -n demo-metrics \
                    // --dry-run=client -o yaml | kubectl apply -f -
                    // '''
                }
            }
        }

        stage('Install Grafana') {
            steps {
                container('helm') {
                        sh 'helm repo add bitnami https://charts.bitnami.com/bitnami'
                        sh 'helm repo update'
                }
            }
        }
    }

    post {
        success {
            script {
                def branchName = env.GIT_BRANCH ?: 'N/A'
                def duration = ((System.currentTimeMillis() - currentBuild.startTimeInMillis) / 1000) + 's'
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: 'good',
                    message: """
                        :white_check_mark: *Grafana Installation Successful*
                        *Branch:* `${branchName}`
                        *Job:* ${env.JOB_NAME} [${env.BUILD_NUMBER}]
                        *Duration:* ${duration}
                    """
                )
            }
        }

        failure {
            script {
                def branchName = env.GIT_BRANCH ?: 'N/A'
                def duration = ((System.currentTimeMillis() - currentBuild.startTimeInMillis) / 1000) + 's'
                def errorDetails = currentBuild.description ?: 'No specific error details available.'
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: 'danger',
                    message: """
                        :x: *Grafana Installation Failed*
                        *Branch:* `${branchName}`
                        *Job:* ${env.JOB_NAME} [${env.BUILD_NUMBER}]
                        *Duration:* ${duration}
                        *Error Details:* `${errorDetails}`
                    """
                )
            }
        }
    }
}