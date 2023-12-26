pipeline {
    agent any
    tools {
        maven "MAVEN"
        jdk "JDK8"
    }
    
    environment {
        SNAPSHOT_REPO = 'bookworl-snapshot'
        RELEASE_REPO = 'bookworld-release'
        CENTRAL_REPO = 'bookworld-maven-central'
        GROUP_REPO = 'bookworld-group'
        NEXUS_USERNAME = 'admin'
        NEXUS_PASSWORD = 'nexus'
        NEXUS_IP = '44.212.43.130'
        NEXUS_PORT = '8081'
        NEXUS_LOGIN = 'nexus' 
        ANSIBLE_LOGIN = 'ansible-login'
        SONARSERVER = 'sonarqube'
        //SONARSCANNER = 'sonarqube-scanner' 
    }
 
    stages {
	    
        stage('Build'){
            steps {
                sh 'cd api/ && mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Archiving the artifact"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test'){
            steps {
                sh 'cd api && mvn -s settings.xml test'
            }
        }
 

        stage('Sonar Analysis') {
            steps {
            	withSonarQubeEnv("${SONARSERVER}") { 
			// You can override the credential to be used 
			        sh 'cd api && mvn clean package sonar:sonar'
            	}
	        }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload War To Nexus"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",
                  groupId: 'bookworld',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  artifacts: [
                    [artifactId: 'bookworld_artifact_id',
                     classifier: '',
                     file: 'api/target/bookworld.war',
                     type: 'war']
                  ]
                )
            }
        }

        stage('Deploy Using Ansible Playbooks'){
            steps{
                ansiblePlaybook(
                inventory: 'ansible/inventory',
                playbook: 'ansible/import.yaml',
                disableHostKeyChecking: true,
		        credentialsId: "${ANSIBLE_LOGIN}",
                extraVars: [
                    nexususername: "${NEXUS_USERNAME}",
                    nexuspassword: "${NEXUS_PASSWORD}",
                    nexusip: "${NEXUS_IP}",
                    repository: "${RELEASE_REPO}",
                    groupId: 'bookworld',
                    artifactId: 'bookworld_artifact_id',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
                ])
            }
        }
    }
} 
