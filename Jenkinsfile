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


    // Extract the version number from the pom.xml file
    unstash 'pom'
    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
    if (matcher) {
        buildVersion = matcher[0][1]
        echo "Release version: ${buildVersion}"
    }
    matcher = null
    
    def mobileDepositApiImage
    stage('Build Docker Image') {
      //unstash Spring Boot JAR and Dockerfile
      dir('target') {
        unstash 'jar-dockerfile'
        mobileDepositApiImage = docker.build "${DOCKER_REGISTRY}/mobile-deposit-api:${dockerTag}"
      }
    }
      
    stage('Publish Docker Image') {
      withDockerRegistry([url: "https://${DOCKER_REGISTRY}/v2", credentialsId: 'registry']) { 
        mobileDepositApiImage.push()
      }
    }
  }
}
    
//set checkpoint before deployment
checkpoint 'Build Complete'

stage('Deploy to Prod') {

  docker.image('jcorioland/devoxx2017attendee').inside('-v /data:/data') {

    // Load the credentials needed to use the kubectl commandline
    withCredentials([file(credentialsId: 'kuby', variable: 'KUBERNETES_SECRET_KEY')]) {
        unstash 'deployment.yml'

        // Execute this sh script
        sh """
          az login --service-principal -u ${env.k8s_username} -p ${env.k8s_password} --tenant ${env.k8s_tenant}
          
          # Install the Kubenetes secret key
          mkdir -p ~/.ssh
          (cat ${KUBERNETES_SECRET_KEY}; echo '\n') > ~/.ssh/id_rsa
          # Ask Azure CLI to install the Kubenetes credentials
          az acs kubernetes get-credentials -n ${env.k8s_name} -g ${env.k8s_resourceGroup}
          # Check if the Kubernetes credentials have been correctly installed
          kubectl version
          
          # Update the deployment.yml with the latest versions of the app
          sed -i 's/REGISTRY_NAME/${env.DOCKER_REGISTRY}/g' ./deployment.yml
          sed -i 's/IMAGE_TAG/${dockerTag}/g' ./deployment.yml
          # Deploy the application
          kubectl apply -f ./deployment.yml
          # Display the installed services (may also display the external IP if the service has been exposed)
          kubectl get services
          
          """
          //send commit status to GitHub
          step([$class: 'GitHubCommitStatusSetter', contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'Jenkins'], statusResultSource: [$class: 'ConditionalStatusResultSource', results: [[$class: 'BetterThanOrEqualBuildResult', message: 'Pipeline completed successfully', result: 'SUCCESS', state: 'SUCCESS']]]])
          
          currentBuild.result = "success"
    }
  }
}
