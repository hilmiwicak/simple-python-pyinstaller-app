node {
    checkout scm
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
    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?'
    }
    stage('Deploy') {
        try {
            dir(path: env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sshPublisher(
                    publishers : [
                        sshPublisherDesc(
                            configName: 'ec2-python-add2vals',
                            verbose: true,
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'sources/**',
                                    removePrefix: 'sources/',
                                ),
                                sshTransfer(execCommand: 'cd ~/add2vals && docker run --rm -v $(pwd):/src cdrx/pyinstaller-linux:python2 "pyinstaller -F add2vals.py" && ./dist/add2vals 1 2'),
                            ]
                        )
                    ],
                    continueOnError: false,
                    failOnError: true,
                )
            }
        } catch (err) {
            currentBuild.result = 'FAILED'
            throw err
        } finally {
            def currentResult = currentBuild.result ?: 'SUCCESS'
            if (currentResult == 'SUCCESS') {
                // archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                sleep(time: 1, unit: 'MINUTES')
                sshPublisher(
                    publishers : [
                        sshPublisherDesc(
                            configName: 'ec2-python-add2vals',
                            verbose: true,
                            transfers: [
                                sshTransfer(execCommand: 'cd ~/add2vals && docker run --rm -v $(pwd):/src cdrx/pyinstaller-linux:python2 "rm -rf build dist" && rm -rf ./*'),
                            ]
                        )
                    ],
                    continueOnError: false,
                    failOnError: true,
                )
            } else {
                sh 'echo "Build failed, not deploying"'
            }
        }
    }
}
