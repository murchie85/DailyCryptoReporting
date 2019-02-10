pipeline {
    agent any

    triggers {
        //pollSCM('*/5 * * * 1-5')
        cron('0 * * * *')
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
        stage('Test environment & pull metrics') {
            steps {
                sh '''source activate ${BUILD_TAG} 
                      pip list
                      which pip
                      which python
                      python3 gbpReport.py
                    '''
            }
        }
        stage('Generate Reports') {
            steps {
                build 'BASEGBP-REPORT'
            }
        }
        stage('Static code metrics') {
            steps {
                echo "Raw metrics"
                sh  ''' source activate ${BUILD_TAG}
                        radon raw --json testTargets/ > raw_report.json
                        radon cc --json testTargets/ > cc_report.json
                        radon mi --json testTargets/ > mi_report.json
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