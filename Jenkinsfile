pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "ldelpozo/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                /*
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )*/
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    def remote = [:]
                    remote.name = "kubemaster"
                    remote.host = "$kubemaster_ip"
                    remote.user = USERNAME
                    remote.password = USERPASS
                    remote.allowAnyHosts = true
                        try {
                            echo "### Puts a configuration file from the current workspace to remote node ####"
                            sshPut remote: remote, from: 'train-schedule-kube.yml', into: '.'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        echo "### Deploy configuration file on kubernetes ####"
                        sshCommand remote: remote, command: "kubectl apply -f train-schedule-kube.yml"

                }
            }
        }
    }
}
