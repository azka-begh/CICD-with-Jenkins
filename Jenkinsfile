pipeline {
    options {
      buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
      //skipDefaultCheckout() 
  }
    agent any
    parameters {
	    //choice(choices: ["mvn clean install", "mvn clean install -DskipTests"], name: "goal", description: "Maven Goal")
	    booleanParam(name: "deploy", defaultValue: false, description: "Deploy the Build")
	    booleanParam(name: "SonarQube", defaultValue: false, description: "ByPass SonarQube Scan")
    }	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"	    
        NEXUS_URL = "172.31.18.80:8081"
        NEXUS_REPOSITORY = "team-artifacts"
	NEXUS_REPO_ID    = "team-artifacts"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
	ecr_repo = '674583976178.dkr.ecr.us-east-2.amazonaws.com/teamimagerepo'
        ecrCreds = 'awscreds'
        image = ''
    }
	
    stages{
	/*stage('Clone Source Code'){
         steps {
           git branch: 'Test-Anisble', url: 'https://github.com/azka-begh/Jenkins.git'
      }
    } */	    
	stage("Fetch from Nexus & Deploy using Ansible"){
		 agent { label 'agent1' }
                 when {
                   expression {
                       return params.deploy  
                }
            }
		steps{
			echo "Stage: Fetch from Nexus & Deploy using Ansible"
			echo "${params.deploy}"
			//sh 'git clone https://github.com/azka-begh/Jenkins.git && git checkout Anisble-Test'
			//sh 'cd Jenkins && ls -l'
			sh 'cd ansible && ls -l'
			sh 'ansible-playbook deployment.yml --extra-vars PASS=${NEXUS_CREDENTIAL_ID} BUILD_ID=${BUILD_ID}'
            }
        }
    }}
	post {
          always {
            cleanWs()
        }
    }   
}
