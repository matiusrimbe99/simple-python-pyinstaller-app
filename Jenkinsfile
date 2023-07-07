node {
  stage('Build') {
    docker.image('python:2-alpine').inside {
      sh 'python -m py_compile sources/add2vals.py sources/calc.py'
    }      
  }
  stage('Test') {
    docker.image('qnib/pytest').inside {
      sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
      junit 'test-reports/results.xml'
    } 
  }
  stage('Deploy') {
    def VOLUME = "${pwd()}/sources:/src"
    def IMAGE = 'cdrx/pyinstaller-linux:python2'

    dir("${env.BUILD_ID}") {
      sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"
    }

    // post {
    //   success {
    //     archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", fingerprint: true
    //     sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
    //   }
    // }
  }
}