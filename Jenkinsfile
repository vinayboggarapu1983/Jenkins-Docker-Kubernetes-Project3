pipeline {
    agent any
	tools {
		maven 'Maven'
	}
	
	environment {
		PROJECT_ID = 'santu0908-365206'
                CLUSTER_NAME = 'autopilot-cluster-1'
                LOCATION = 'us-central1'
                CREDENTIALS_ID = 'kubernetes'		
	}
	
        
        stages{

              stage('Quality Gate Status Check'){
		      agent {
                docker {
                 image 'maven:3-openjdk-11'

                }
            }
                  steps{
                      script{
			      withSonarQubeEnv('sonarserver') { 
			      sh "mvn clean sonar:sonar"
                       	     	}
			      timeout(time: 1, unit: 'HOURS') {
			      def qg = waitForQualityGate()
				      if (qg.status != 'OK') {
					   error "Pipeline aborted due to quality gate failure: ${qg.status}"
				      }
                    		}
		    	   
                 	}
               	 }  
              }	
			       	     	         
    
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build') {
		    steps {
			    sh 'mvn clean package'
		    }
	    }
	    
	    stage('Test') {
		    steps {
			    echo "Testing..."
			    sh 'mvn test'
		    }
	    }
		
		stage('Upload War To Nexus'){
            steps{
                
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'k8sDemo', 
                            classifier: '', 
                            file: "target/Project Build-1.0-AMIT.war", 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus3', 
                    groupId: 'k8sDemo.devops.gcp', 
                    nexusUrl: '35.228.99.220:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: repository-example, 
                    version: "1.0-AMIT"
                    }
            }
	    
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				    myimage = docker.build("vinayboggarapu/jenkinsdevops:${env.BUILD_ID}")
			    }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
			    script {
				    echo "Push Docker Image"
				    withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
            				sh "docker login -u vinayboggarapu -p ${dockerhub}"
				    }
				        myimage.push("${env.BUILD_ID}")
				    
			    }
		    }
	    }
	    
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' serviceLB.yaml"
				sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
			    echo "Start deployment of serviceLB.yaml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'serviceLB.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Start deployment of deployment.yaml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
	    }
            }
	}
