#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
    stage("Create script with exit code 1"){
            // script path
            SCRIPT_PATH = "~/exit_with_1.sh"

            // create the script
            sh "echo '# This script exits with 1' > ${SCRIPT_PATH}"
            sh "echo 'exit 1'                    >> ${SCRIPT_PATH}"

            // print it, just in case
            sh "cat ${SCRIPT_PATH}"

            // grant run permissions
            sh "chmod +x ${SCRIPT_PATH}"
    }
    stage("Copy file")  {
        // script path
        SCRIPT_PATH = "~/exit_with_1.sh"

       // invoke script, and save exit code in "rc"
        echo 'Running the exit script...'
        rc = sh(script: "${SCRIPT_PATH}", returnStatus: true)

        // check exit code
        sh "echo \"exit code is : ${rc}\""

        if (rc != 0) 
        { 
            sh "echo 'exit code is NOT zero'"
        } 
        else 
        {
            sh "echo 'exit code is zero'"
        }
    }
    post {
        always {
            // remove script
            sh "rm ${SCRIPT_PATH}"
        }
    }
    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Deploye Code') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

			println rc
			
			// need to pull out assigned username
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}else{
			   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}
			  
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
        }
    }
}
