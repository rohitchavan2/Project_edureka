pipeline {
    agent {
        label 'env-test'
    }

    stages {
        stage('Install and Configure Puppet Agent') {
            steps {
                script {
                    // Install Puppet Agent
                    sh 'curl -O https://apt.puppetlabs.com/puppet6-release-bionic.deb'
                    sh 'sudo dpkg -i puppet6-release-bionic.deb'
                    sh 'sudo apt-get update'
                    sh 'sudo apt-get install -y puppet-agent'

                    // Configure Puppet Agent
                    sh 'echo "server = puppetmaster.example.com" | sudo tee -a /etc/puppetlabs/puppet/puppet.conf'
                    sh 'sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true'
                }
            }
        }

        stage('Push Ansible configuration for Docker installation on test server') {
            steps {
                script {
                    // Replace 'ubuntu@ip-172-31-1-182' with the actual name or IP address of your test server
                    def testServer = 'ubuntu@ip-172-31-1-182'

                    // Replace 'https://github.com/rohitchavan2/project_edureka.git' with your Ansible playbook Git repository URL
                    def ansibleRepo = 'https://github.com/rohitchavan2/project_edureka.git'

                    // Clone the Ansible playbook repository
                    sh "git clone ${ansibleRepo} /tmp/ansible-repo"

                    // Copy Ansible playbook to the test server
                    sh "scp -r /tmp/ansible-repo/* ${testServer}:/home/ubuntu/"

                     // Run Ansible playbook on the test server
                    sh "ssh ${testServer} 'ansible-playbook /home/ubuntu/docker-installation.yml'"

                    // Clean up cloned repository on the Jenkins node
                    sh "rm -rf /tmp/ansible-repo"
                }
            }
        }

        stage('Build and deploy PHP Docker container') {
            steps {
                script {
                    // Replace 'https://github.com/rohitchavan2/project_edureka.git' with your Git repository URL
                    def gitRepo = 'https://github.com/rohitchavan2/project_edureka.git'

                    // Clone the Git repository
                    sh "git clone ${gitRepo} /tmp/php-web-app"

                    // Move to the directory with PHP website source code and Dockerfile
                    sh "cd /tmp/php-web-app"  // Adjust this line based on your directory structure

                    // Build the Docker image
                    sh "docker build -t php-web-app ."

                    // Run the Docker container on port 8000
                    sh "docker run -d --name php-container -p 8000:80 php-web-app"
                }
            }
        }
    }

    post {
        success {
            // Send email on success
            emailext subject: "Job Succeeded: ${currentBuild.fullDisplayName}",
                      body: "The build of ${currentBuild.fullDisplayName} succeeded.",
                      to: "rohit.chavan060898@gmail.com",
                      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
        }

        failure {
            // Send email on failure
            emailext subject: "Job Failed: ${currentBuild.fullDisplayName}",
                      body: "The build of ${currentBuild.fullDisplayName} failed.",
                      to: "rohit.chavan060898@gmail.com",
                      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
        }
    }
}
