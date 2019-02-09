pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * 1-5')
    }
    options {
        skipDefaultCheckout(true)
        // Keep the 10 most recent builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }
    environment {
      PATH="/users/shared/jenkins/miniconda3/bin:$PATH"
    }

    stages {

        stage ("Code pull"){
            steps{
                checkout scm
            }
        }
        stage('Build environment') {
            steps {
                sh '''conda create --yes -n ${BUILD_TAG} python
                      source activate ${BUILD_TAG} 
                      pip install -r requirements.txt
                    '''
            }
        }
        stage('Test environment') {
            steps {
                sh '''source activate ${BUILD_TAG} 
                      pip list
                      which pip
                      which python
                      python3 gbpReport.py
                      python3 graphGBP.py
                    '''
            }
        }
        stage('Static code metrics') {
            steps {
                echo "Raw metrics"
                sh  ''' source activate ${BUILD_TAG}
                        radon raw --json irisvmpy/ > raw_report.json
                        radon cc --json irisvmpy/ > cc_report.json
                        radon mi --json irisvmpy/ > mi_report.json
                        //TODO: add conversion and HTML publisher step
                    '''
            }
        }
    }
    post {
        always {
            sh 'conda remove --yes -n ${BUILD_TAG} --all'
        }

    }
}