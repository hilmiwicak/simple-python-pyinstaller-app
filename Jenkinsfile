node {
    stage('Build') {
        docker.image('python:3.12.1-alpine3.19').inside {
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
    stage('Deploy') {
        withEnv(['VOLUME=$(pwd)/sources:/src',
                 'IMAGE=cdrx/pyinstaller-linux:python2']) {
            try {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh 'docker run --rm -v $VOLUME $IMAGE "pyinstaller -F add2vals.py"'
                }
            } catch (err) {
                currentBuild.result = 'FAILED'
                throw err
            } finally {
                def currentResult = currentBuild.result ?: 'SUCCESS'
                if (currentResult == 'SUCCESS') {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                } else {
                    sh 'echo "Build failed, not deploying"'
                }
            }
        }
    }
}
