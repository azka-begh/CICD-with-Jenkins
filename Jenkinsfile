pipeline {
	options {
		buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
		//skipDefaultCheckout() 
		disableConcurrentBuilds()
	}
	agent { label 'agent1' }
	parameters {
		booleanParam(name: "Deploy", defaultValue: false, description: "Deploy the Build to EKS cluster")
    }	
    environment {
	ecrRepo = '674583976178.dkr.ecr.us-east-2.amazonaws.com/teamimagerepo'
        //ecrCreds = 'awscreds'
	dockerImage = "${env.ecrRepo}:${env.BUILD_ID}"
    }
	
    stages{
	    stage('Docker Image Build') {
		    steps {
			    sh 'docker build -t ${env.ecrRepo}:latest ./docker/'
			    sh 'docker tag ${env.ecrRepo} ${env.dockerImage} '
		    }}
	    stage('Push Image to ECR'){
		    steps {
			    script {
				    docker push ${env.ecrRepo}:latest
				    def exit1 = sh script: 'echo $?'
                                    if (exit1 != 0){
                                    sh 'echo $(aws ecr get-authorization-token --region us-east-2 --output text --query 'authorizationData[].authorizationToken' | base64 -d | cut -d: -f2) | docker login -u AWS 674583976178.dkr.ecr.us-east-2.amazonaws.com --password-stdin'
				    docker push 674583976178.dkr.ecr.us-east-2.amazonaws.com/teamimagerepo:latest
				    }
				    sh 'docker push ${env.dockerImage}'
			    }}
		    post { success { sh 'docker builder prune --all -f' } }
	    }
	    stage('Deploy to EKS'){
                 when { expression { 
			 return params.Deploy }}
            steps {
		script{
		 sh 'eksctl get cluster --region us-east-2'
		 def exit2 = sh script: 'echo $?'
		 if (exit2 != 0){
			sh './k8s/cluster.sh'
		 }
                 sh '''
		 kubectl apply -f ./k8s/eksdeploy.yml
		 kubectl get deployments  
                 sleep 5
                 kubectl get svc
                 '''   }}
		    post {
			    always { cleanWs() } }
	    }
    }
	post { always { cleanWs() } }
}
