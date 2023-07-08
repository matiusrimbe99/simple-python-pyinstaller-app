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
//     docker.image('cdrx/pyinstaller-linux:python2').inside {
//       sh 'pyinstaller --onefile sources/add2vals.py'
//       archiveArtifacts 'dist/add2vals'
//     }
//   }
// }

node {
    options {
        skipStagesAfterUnstable()
    }

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

    stage('Deliver') {
        dir("${env.BUILD_ID}") {
            unstash 'compiled-results'
            sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 pyinstaller -F add2vals.py"
        }
        archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", fingerprint: true
        sh "docker run --rm -v ${pwd()}/sources:/src cdrx/pyinstaller-linux:python2 rm -rf build dist"
    }
}
