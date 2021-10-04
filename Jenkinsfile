pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile.dev'
    }
  }
  environment {
    PROXY_TEST_USERNAME     = credentials('proxy-test-username')
    PROXY_TEST_PASSWORD = credentials('proxy-test-password')
  }
  stages {
    stage('Unit Test') {
      steps {
        sh 'go test -coverpkg=./... -coverprofile=coverage.out ./... -timeout 100s -parallel 4'
      }
    }

    stage('Coverage') {
      steps {
        sh 'go tool cover -html=coverage.out -o coverage.html'
        archiveArtifacts '*.html'
        sh 'echo "Coverage Report: ${BUILD_URL}artifact/coverage.html"'
        sh '''t=$(go tool cover -func coverage.out | grep total | tail -1 | awk \'{print substr($3, 1, length($3)-1)}\')
if [ "${t%.*}" -lt 80 ]; then 
    echo "Coverage failed ${t}/80"
    exit 1
fi'''
      }
    }
    stage('Main Race Condition') {
      steps {
        sh 'go run --race main.go -t https://proxytest.ddosifytech.com/ -d 1 -n 1500 -a ${PROXY_TEST_USERNAME}:${PROXY_TEST_PASSWORD} -p https'
      }
    }

  }
  post {
    unstable {
      slackSend(channel: '#jenkins', color: 'danger', message: "${currentBuild.currentResult}: ${currentBuild.fullDisplayName} - ${BUILD_URL}")
    }

    failure {
      slackSend(channel: '#jenkins', color: 'danger', message: "${currentBuild.currentResult}: ${currentBuild.fullDisplayName} - ${BUILD_URL}")
    }

  }
}
