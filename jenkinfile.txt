pipeline {
    agent any
 
    tools {
        ansible 'ansible'
    }
 
    stages {
 
        stage('Checkout Ansible Repo') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-classic-id',
                    url: 'https://github.com/sayeeswari-git/Jenkins-repo.git'
            }
        }
 
        stage('Checkout Website Repo') {
            steps {
                dir('web-src') {
                    git branch: 'main',
                        credentialsId: 'github-classic-id',
                        url: 'https://github.com/sayeeswari-git/jenkins-repo2.git'
                }
            }
        }
 
        stage('Deploy with Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'vm2-deploy-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
 
                    echo 'Deploying to multiple web servers using Ansible from two repos...'
 
                    sh 'cp ${SSH_KEY} /tmp/ssh_key'
                    sh 'chmod 600 /tmp/ssh_key'
 
                    ansiblePlaybook(
                        playbook: 'deploy.yml',
                        inventory: 'inventory/hosts.ini',
                        colorized: true
                    )
 
                    sh 'rm -f /tmp/ssh_key'
                }
            }
        }
    }

}
