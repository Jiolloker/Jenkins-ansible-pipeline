// Jenkinsfile pipeline for ansible playbooks
pipeline {
    agent any
    stages {
        stage('Ansible') {
            steps {
                ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible2', inventory: 'inventory.ini', playbook: 'playbook.yml'
            }
        }
    }
}