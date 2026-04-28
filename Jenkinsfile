pipeline {
    agent none 

    stages {
        stage('Build & Package') {
            agent { label 'Slave-01' } 
            steps {
                git branch: 'main', url: 'https://github.com/knowledgemateit/Project-Maven-Tomcat-Nexus-Sonar-Setup-PollApp.git'
                sh 'mvn clean package -DskipTests'
                
                // Save the WAR file to Jenkins controller memory
                stash includes: 'target/*.war', name: 'app-war'
            }
            post {
                always { cleanWs() } // Clean slave workspace
            }
        }

        stage('Ansible Deploy') {
            agent { label 'built-in' } // Run Ansible from Master
            environment {
                ANSIBLE_HOST_KEY_CHECKING = 'False'
            }
            steps {
                // Pull the WAR file onto the Master's local workspace
                unstash 'app-war'
                
                ansiblePlaybook(
                    playbook: 'deploy-docker.yml',
                    inventory: '/opt/ansible/hosts',
                    installation: 'ansible-latest',
                    colorized: true,
                    extraVars: [
                        war_path: "${WORKSPACE}/target/PollApp.war"
                    ]
                )
            }
            post {
                always { cleanWs() } // Clean master workspace
            }
        }
    }
}
