pipeline {
	agent any
	
	stages {

		stage('Checkout') {
			steps {
				echo 'Pulling code from Github'
			}
		}
		
		stage('Deploy with Ansible') {
			steps {
				sh '''
				ansible-playbook nginx.yml
				'''
			}
		}
	
	}
}
