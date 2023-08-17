pipeline {
    agent any
    environment {
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}"
        ZIP_OUTFILE = "${WORKSPACE}/build-${PACKAGE_VERSION}.zip"
    }
    stages {
        stage ('Build') {
            steps {
                sh '''#!/bin/bash
                python3 -m venv test3
                source test3/bin/activate
                pip install pip --upgrade
                pip install -r requirements.txt
                export FLASK_APP=application
                '''
            }
        }
        stage ('Test') {
            steps {
                sh '''#!/bin/bash
                source test3/bin/activate
                py.test --verbose --junit-xml test-reports/results.xml
                '''
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage ('Deploy') {
            steps {
                input(message: 'Are you ready to deploy?', ok: 'Continue', abort: 'Stop')
            }
        }
        stage ('Packaging the output files') {
            steps {
                zip dir: env.ZIP_OUTFILE
            }
        }
    }
}

