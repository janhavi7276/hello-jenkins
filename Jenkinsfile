def run(cmd) { if (isUnix()) { sh cmd } else { bat cmd } }

def awsRun(cmd) {
  withCredentials([
    string(credentialsId: 'aws-key-id', variable: 'AKIARWOVJUTRPG52BLFH'),
    string(credentialsId: 'aws-secret', variable: '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')
  ]) { run(cmd) }
}

def ecrLogin() {
  awsRun "aws ecr get-login-password --region ${env.REGION} | " +
         "docker login --username AWS --password-stdin ${env.REGISTRY}"
}

def deploy() {
  awsRun "aws ssm send-command --region ${env.REGION} " +
         "--instance-ids ${env.INSTANCE} " +
         "--document-name AWS-RunShellScript --parameters file://deploy.json"
}

pipeline {
  agent any
  environment {
    REGION   = 'ap-south-1'
    ACCOUNT  = '123456789012'
    REPO     = 'hello-jenkins'
    INSTANCE = 'none'
    REGISTRY = "${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com"
    IMG      = "${REPO}:${BUILD_NUMBER}"
    REMOTE   = "${REGISTRY}/${REPO}:latest"
  }
  stages {
    stage('Build') { steps { script { run "docker build -t ${IMG} ." } } }
    stage('Test')  { steps { script { run "docker run --rm ${IMG} nginx -t" } } }
    stage('Push') {
      steps { script {
        ecrLogin()
        run "docker tag ${IMG} ${REMOTE}"
        run "docker push ${REMOTE}"
      } }
    }
    stage('Deploy') {
      when { not { environment name: 'INSTANCE', value: 'none' } }
      steps { script { deploy() } }
    }
  }
}
