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
                sh "sed -i 's|\$DOCKER_IMAGE_NAME:\$BUILD_NUMBER|${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}|g' ${WORKSPACE}/train-schedule-kube.yml"
                sh "cat ${WORKSPACE}/train-schedule-kube.yml"
                
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    remote.user = USERNAME
                    remote.password = USERPASS
                    echo "### Puts a configuration file from the current workspace to remote node ####"
                    sshPut remote: remote, from: 'train-schedule-kube.yml', into: '.'
                    echo "### Deploy configuration file on kubernetes ####"
                    //sshCommand remote: remote, command: """sed -i 's/\$DOCKER_IMAGE_NAME:\$BUILD_NUMBER/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}/g' train-schedule-kube.yml"""
                    sshCommand remote: remote, command: "kubectl apply -f train-schedule-kube.yml"
                }
            }
        }
}
