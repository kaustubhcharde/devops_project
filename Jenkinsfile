pipeline{
	agent any
	tools {
        maven 'maven3'
    }
    environment {
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
	stages{
		stage('git clone'){
			steps{
				git branch: 'master', credentialsId: 'git', url: 'https://github.com/kaustubhcharde/devops_project.git'
			}
		}
		stage('code build'){
			steps{
				sh "mvn clean package"
			}
		}
		stage('Docker Build'){
            steps{
                sh "docker build . -t dockerkbc/testapp:${DOCKER_TAG}"
            }
        }
        stage('Docker push'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerpasswd', usernameVariable: 'dockeruser')]) {
                    sh "docker login -u ${dockeruser} -p ${dockerpasswd}"
                }
                sh "docker push dockerkbc/testapp:${DOCKER_TAG}"
            }
        }
        stage('Docker deploy'){
            steps{
                sh "ansible-galaxy collection install community.docker"
                ansiblePlaybook credentialsId: 'ansible-creds', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
	}
	post {
        	success {
        	sh "docker rmi -f \$(docker images -aq)"
    	}
    }
}
