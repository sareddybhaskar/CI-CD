node {
def NAME   = 'container.co:550/root/production-stack/cognidesk-auth-app'
def LATEST = "${NAME}:${params.release_number}"

    stage ('SCM checkout'){
            git branch: 'production', credentialsId: 'Bitbucket', url: 'https://user@bitbucket.org/skittergit/cognidesk-auth-app.git'
        }
        sh "git tag ${params.release_number}"
        sh "git push https://user:password@bitbucket.org/skittergit/cognidesk-auth-app.git --tags"

   
properties([[$class: 'JiraProjectProperty'], gitLabConnection(''), [$class: 'BeforeJobSnapshotJobProperty'], parameters([string(defaultValue: 'release_number', description: '', name: '${release_number}', trim: true)])])    

    stage ('Build Image '){
        withDockerRegistry(credentialsId: 'Docker_registry', url: 'http://container.co:') {
        def customImage = docker.build("${LATEST}")

            /* Push the container to the custom Registry */
            customImage.push()
        }
    }
    stage("docker_scan"){
      sh '''
        docker run -d --name db arminc/clair-db
        sleep 15 # wait for db to come up
        docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan
        sleep 1
        DOCKER_GATEWAY=$(docker network inspect bridge --format "{{range .IPAM.Config}}{{.Gateway}}{{end}}")
        wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
        ./clair-scanner --ip="$DOCKER_GATEWAY" gitlab.workativ.co:5650/root/cognidesk/development/cognidesk-settings:develop || exit 0
      '''
       }
    stage ('Production deployment '){  
      sh "ssh ubuntu@localhost sudo docker pull ${LATEST}"
      sh "ssh ubuntu@localhost sudo docker stack rm authenticate"
      sleep time: 8000, unit: 'MICROSECONDS'
      sh "ssh ubuntu@localhost sudo docker stack deploy -c /root/production-stack/auth/authenticate.yml authenticate "
   }
    
  }