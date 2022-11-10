pipeline {
    agent any
	tools {
		maven 'Maven3.8.6'
	}
	
	environment {
		PROJECT_ID = 'santu0908-365206'
                CLUSTER_NAME = 'autopilot-cluster-1'
                LOCATION = 'us-central1'
                CREDENTIALS_ID = 'kubernetes'		
	}
	
        
        stages{

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
		
		
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				    myimage = docker.build("vinayboggarapu/jenkinsdevops:${env.BUILD_ID}")
			    }
		    }
	    }
	      
            }
	}
