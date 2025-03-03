node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }

    stage('Manual Approval') {
      input message: 'Lanjutkan ke tahap Deploy?'
    }

    stage('Deploy') {
        dir("${env.BUILD_ID}") {
            unstash 'compiled-results'
            sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 'pyinstaller -F add2vals.py'"
        }

        withCredentials([sshUserPrivateKey(credentialsId: '13.250.14.198', keyFileVariable: '13.250.14.198', usernameVariable: 'ubuntu')]) {
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: 'My Web Server', 
                        transfers: [
                            sshTransfer(
                                remoteDirectory: '/python-app',  
                                removePrefix: "${env.BUILD_ID}/sources/dist/", 
                                sourceFiles: "${env.BUILD_ID}/sources/dist/add2vals",
                                execCommand: "chmod a+x python-app/add2vals"
                            )
                        ], 
                        verbose: true
                    )
                ]
            )
        }

        sleep time: 1, unit: 'MINUTES'

        archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", fingerprint: true
        sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 'rm -rf build dist'"
    }
}