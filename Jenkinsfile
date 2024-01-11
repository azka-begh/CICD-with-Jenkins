//To build Docker Images and push them to ECR without using plugins
pipeline {
	options {
		buildDiscarder(logRotator(numToKeepStr: '3'))
		disableConcurrentBuilds()
	}
	agent { label 'agent1' }
	parameters {
		booleanParam(name: "Deploy", defaultValue: false, description: "Deploy the Build to EKS cluster")
	}	
	environment {
	ecrRepo = "674583976178.dkr.ecr.us-east-2.amazonaws.com/teamimagerepo"
	dockerImage = "${env.ecrRepo}:${env.BUILD_ID}"
	}
	stages{
		stage('Docker Image Build') {
			steps {
			    sh 'docker build -t $dockerImage ./docker/'
			    sh 'docker tag $ecrRepo $ecrRepo:latest'
		    }}
		stage('Push Image to ECR'){
			steps {
				script {
				    def status = sh(returnStatus: true, script: 'docker push $ecrRepo:latest')
                                    if (status != 0){
					    sh "aws ecr get-authorization-token --region us-east-2 --output text --query 'authorizationData[].authorizationToken' | base64 -d | cut -d: -f2 > ecr.txt"
                                            sh 'cat ecr.txt | docker login -u AWS 674583976178.dkr.ecr.us-east-2.amazonaws.com --password-stdin'
					    sh 'docker push $ecrRepo:latest'
				    }
				    sh 'docker push $dockerImage'
				    sh 'rm -f ecr.txt'
			    }}
		    post { success { sh 'docker builder prune --all -f' } }
		}
		stage('Deploy to EKS'){
			when { expression { return params.Deploy }}
			steps {
				script{
					def output = sh(script: "echo \$(eksctl get cluster --region us-east-2)", returnStdout: true)
					//def countCluster = sh(script: 'eksctl get cluster --region us-east-2')
					if (output == "No clusters found"){
						echo "${output}... Creating cluster(s)"
						sh './k8s/cluster.sh' }
					sh '''kubectl apply -f ./k8s/eksdeploy.yml
                                        kubectl get deployments && sleep 5
                                        kubectl get svc
					'''   }}
			post { always { cleanWs() } }
		}
	}
}
