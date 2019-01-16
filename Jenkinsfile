/* 
* Copyright (c) 2017 and Confidential to Pegasystems Inc. All rights reserved11.  
*/ 

pipeline {
    agent any

    options {
      timestamps()
      timeout(time: 15, unit: 'MINUTES')
      withCredentials([
        usernamePassword(credentialsId: 'PDM_JFrogRepository', 
            passwordVariable: 'P@ssw0rd_pega', 
            usernameVariable: 'pega_admin')
        /*usernamePassword(credentialsId: 'puneeth_export', 
            passwordVariable: 'rules', 
            usernameVariable: 'puneeth_export')*/
        ])
    }

    stages {

        stage('Check for merge conflicts'){
            steps {
                echo ('Clear workspace')
                dir ('build/export') {
                    deleteDir()
                }

                echo 'Determine Conflicts'
                sh "./gradlew"
		sh "./gradlew getConflicts -PtargetURL=${PEGA_DEV} -Pbranch=TestDevOps4 -PpegaUsername=puneeth_export -PpegaPassword=rules"
            }
        }

        stage('Run unit tests'){
          steps {
            echo 'Execute tests'

            withEnv(['TESTRESULTSFILE="TestResult.xml"']) {
              sh "./gradlew executePegaUnitTests -PtargetURL=${PEGA_DEV} -PpegaUsername=puneeth_export -PpegaPassword=rules -PtestResultLocation=${WORKSPACE} -PtestResultFile=${TESTRESULTSFILE}"
                    
             // junit(allowEmptyResults: true, testResults: "${env.WORKSPACE}/${env.TESTRESULTSFILE}")

              script {
                if (currentBuild.result != null) {
                  input(message: 'Ready to share tests have failed, would you like to abort the pipeline?')
                }
              }
            }
          }
       }
							  }
  }

  post {
  always {
          junit '**/reports/junit/*.xml'
	        }
    failure {
      mail (
          subject: "${JOB_NAME} ${BUILD_NUMBER} merging branch  has failed",
          body: "Your build ${env.BUILD_NUMBER} has failed.  Find details at ${env.RUN_DISPLAY_URL}", 
          to: "puneeth.in@gmail.com"
      )
    }   
  }
}
