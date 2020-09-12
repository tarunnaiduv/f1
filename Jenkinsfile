pipeline {
    agent any
    parameters {
		choice (
			name: 'Test_Permission',
			choices: ['Proceed' , 'Stop'],
			description: ''
		)
		choice (
			name: 'Deploy_Permission',
			choices: ['Proceed' , 'Stop'],
			description: ''
		)
	}
    stages {
		stage('Clean WS'){
			steps {
				cleanWs()
				sh 'ls'
			}
		}
		stage('SCM'){
			steps {
				checkout scm
				sh 'ls'
			}
		}
		stage('build') {
			steps {
				withEnv(["HOME=${env.WORKSPACE}"]) {
					echo '#####-Beginnig Build-#####'
					sh 'python3 -m virtualenv my_env'   // Setting Up Python Virtual Environment
					sh 'source my_env/bin/activate'   // Activating Python Virtual Environment
					sh 'pip3 install --user -r requirements.txt'   // Installing Required Python Modules
					echo '#####-Build Complete-#####'
				}
			}
		}
		stage('test') {
			when{
				expression {
					params.Test_Permission == 'Proceed'
				}
			}
			steps {
				withEnv(["HOME=${env.WORKSPACE}"]) {
					echo '#####-Beginnig Test-#####'
					sh 'python3 lint.py'   // Checking the code quality by linting
					sh 'python3 -m pytest'   // Testing the Functions
					echo '#####-Test Complete-#####'
				}
			}   
		}
		stage('deploy') {
			when{
				expression {
					params.Deploy_Permission == 'Proceed'
				}
			}
			steps([$class: 'BapSshPromotionPublisherPlugin']) {
				sshPublisher (
					continueOnError: false, failOnError: true,
					publishers: [
						sshPublisherDesc (
							configName: "21046-abhishek",
							verbose: true,
							transfers: [
								sshTransfer(execCommand: "kill -9 $(lsof -t -i:5000)"),
								sshTransfer(execCommand: "python3 -m virtualenv env"),
								sshTransfer(execCommand: "source env/bin/activate"),
								sshTransfer(execCommand: "pip3 install --user -r requirements.txt"),
								sshTransfer(execCommand: "/bin/python3 app.py &")
							]
						)
					]
				)
			}
		}
    }
}

