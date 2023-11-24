
pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    // Using below docker image
                    image 'python:2-alpine'
                }
            }
            steps {

                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Staging') {
            agent {
                docker {
                    // using pytest image for running tests inside docker image
                    image 'qnib/pytest'
                }
            }
            steps {
                // running tests
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deployment') {
                    agent any
                    //This environment block defines two variables which will be used later in the 'Deployment' stage.
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

                    // Send successful build status to JIRA
                    post {
                        always {
                            jiraSendBuildInfo site: 'moosasharieff.atlassian.net', branch: 'EYES-2-CI-CD'
                            }
                        }

        }
    }
}
