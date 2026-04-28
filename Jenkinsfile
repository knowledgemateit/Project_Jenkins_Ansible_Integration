pipeline {
    agent none 
    stages {
        stage('Build') {
            agent { label 'Slave-01' }
            steps {
                git branch: 'main', url: 'https://github.com/knowledgemateit/Project-Maven-Tomcat-Nexus-Sonar-Setup-PollApp.git'
                sh 'mvn clean package -DskipTests'
                stash includes: 'target/*.war', name: 'app-war'
            }
        }

        stage('Infrastructure Setup') {
            agent { label 'built-in' }
            steps {
                // Only run this if the container is down or config changed
                ansiblePlaybook(
                    playbook: 'ansible/playbooks/setup-tomcat.yml',
                    inventory: 'ansible/inventory/hosts.ini',
                    installation: 'ansible-latest'
                )
            }
        }

        stage('App Deployment') {
            agent { label 'built-in' }
            steps {
                unstash 'app-war'
                ansiblePlaybook(
                    playbook: 'ansible/playbooks/deploy-war.yml',
                    inventory: 'ansible/inventory/hosts.ini',
                    installation: 'ansible-latest',
                    extraVars: [ war_path: "${WORKSPACE}/target/PollApp.war" ]
                )
            }
        }
    }
}
