// Mandatory environment variables:
// DOCKER_REGISTRY (without http://, no ending /)
// k8s_username
// k8s_password
// k8s_tenant
// k8s_name
// k8s_resourceGroup

// You need to create 2 Jenkins Credentials
// * 1 type 'Secret file', with ID 'kuby'
// * 1 type 'username with password' with ID 'registry'

def buildVersion = null
def short_commit = null
echo "Building ${env.BRANCH_NAME}"

pipeline {
    agent { docker 'maven:3.3.3' }
    stages {
        stage('build') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
