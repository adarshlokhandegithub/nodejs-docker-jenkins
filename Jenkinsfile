node {
  try {
    stage('Checkout') {
      checkout scm
    }
    stage('Environment') {
      sh 'git --version'
      echo "Branch: ${env.BRANCH_NAME}"
      sh 'docker -v'
      sh 'printenv'
    }
    stage('Build'){
      if(env.BRANCH_NAME == 'master'){
        sh 'docker build -t my-app --no-cache .'
        sh 'docker tag my-app localhost:5000/my-app'
        sh 'docker push localhost:5000/my-app'
      }
    }
    stage('Deploy'){
      if(env.BRANCH_NAME == 'master'){
        sh 'docker pull localhost:5000/my-app'
        sh 'docker run -d -p 3000:30000 --name my-app localhost:5000/my-app:latest'
        sh 'docker rmi -f app localhost:5000/my-app'
      }
    }
  }
  catch (err) {
    throw err
  }
}
