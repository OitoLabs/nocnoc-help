def release=""
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""
    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.rawBuild.changeSets
    echo "${changeLogSets}"
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            echo "${changeLogSets[0].items}"
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += "  ${j+1} - ${truncated_msg} <br>"
        }
    }

    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}
pipeline {
	
    agent any
           parameters {
choice(choices: 'None\nQA\nPrepord\nProduction', description: 'Nocnoc enviorment-value', name: 'NocNocEnv')
}
  	options { buildDiscarder(logRotator(numToKeepStr: '10')) }

    stages {
        stage('Checkout'){
            steps {
                echo 'Checking out code from scm...'
                checkout scm
            }
        }
        stage('Deploy - Content'){
            steps{
            	script{
			  if("${params.NocNocEnv}" == "Production") {
           sh '''
           aws s3 sync . s3://production-nocnoc-static-content/help --exclude ".git/*" --acl public-read
           '''
		}
                 }
            }
        }
    }
                post {
    success {
          sh 'echo slack notification and email'
          script{
            if("${params.NocNocEnv}" == "Production") {
                	
          slackSend channel: "#production_release", message: "STATUS: *Deployed* :thumbsup_all:\nJob Name: *${env.JOB_NAME}* \nBUILD: *[${env.BUILD_NUMBER}]* \nURL: (${env.BUILD_URL})\nGit-Commits List -\n${getChangeString().replace('<br>', '\n')}" ,color: "#0fc64f"
    }}}
	 always { 
            cleanWs()
        }
}
}
