// node {
//   stage('Build') {
//     docker.image('python:2-alpine').inside {
//       sh 'python -m py_compile sources/add2vals.py sources/calc.py'
//     }      
//   }
//   stage('Test') {
//     docker.image('qnib/pytest').inside {
//       sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
//       junit 'test-reports/results.xml'
//     } 
//   }
//   stage('Deploy') {
//     def VOLUME = "${pwd()}/sources:/src"
//     def IMAGE = 'cdrx/pyinstaller-linux:python2'

//     dir("${env.BUILD_ID}") {
//       sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
//     }

//     archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
//     sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'" 
    

//     // post {
//     //   success {
//     //     archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", fingerprint: true
//     //     sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"
//     //   }
//     // }
//   }
// }

pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') { 
            agent any
            environment { 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results') 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}