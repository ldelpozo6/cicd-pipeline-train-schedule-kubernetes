node {
        def DOCKER_IMAGE_NAME="ldelpozo/train-schedule"
        stage('Build') {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
        }
        stage('Build Docker Image') {
            if (env.BRANCH_NAME == 'master') {
                app = docker.build("${DOCKER_IMAGE_NAME}")
                app.inside {
                    sh 'echo Hello, World!'
                }
            }
        }
        stage('Push Docker Image') {
            if (env.BRANCH_NAME == 'master') {
                docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
            }
            }
        }
        stage('DeployToProduction') {
            if (env.BRANCH_NAME == 'master') {
                input 'Deploy to Production?'
                milestone(1)
                def remote = [:]
                remote.name = "kubemaster"
                remote.host = "$kubemaster_ip"
                remote.allowAnyHosts = true
                def vars= """
                DOCKER_IMAGE_NAME="${DOCKER_IMAGE_NAME}"
                BUILD_NUMBER=${env.BUILD_NUMBER}"
                """
                
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    remote.user = USERNAME
                    remote.password = USERPASS
                    echo "### Puts a configuration file from the current workspace to remote node ####"
                    sshPut remote: remote, from: 'train-schedule-kube.yml', into: '.'
                    echo "### Deploy configuration file on kubernetes ####"
                    writeFile file: 'vars', text: "$vars"
                    sshPut remote: remote, from: 'vars', into: '.'
                    //sshCommand remote: remote, command: "echo -e \"DOCKER_IMAGE_NAME=\"${DOCKER_IMAGE_NAME}" \nBUILD_NUMBER=\"${env.BUILD_NUMBER}\"" "
                    sshCommand remote: remote, command: "source vars; kubectl apply -f train-schedule-kube.yml"
                }
            }
        }
}
