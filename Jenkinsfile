pipeline {
    environment {
        IMAGEN = "aleromero/jenkins:v1"
        LOGIN = 'DOCKER_HUB_IVANR'
    }
    agent any
    stages {
        stage("Bajar_imagen") {
            agent {
                docker {
                    image "python:3"
                    args '-u root:root'
                }
            }
            stages {
                stage('Repositorio') {
                    steps {
                        git branch:'main',url:'https://github.com/AlexRomero10/prueba_icdc.git'
                    }
                }
                stage('Requirements') {
                    steps {
                        sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'cd app && pytest test_app.py'
                    }
                }

            }
        }
        stage("Gen_imagen") {
            agent any
            stages {
                stage('build') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:latest"
                        }
                    }
                }
                stage('Subir') {
                    steps {
                        script {
                            docker.withRegistry( '', LOGIN ) {
                                newApp.push()
                            }
                        }
                    }
                }
                stage('Borrar') {
                    steps {
                        sh "docker rmi $IMAGEN:latest"
                    }
                }
            }
        }
        stage('VPS') {
        agent any
        steps {
            sshagent(credentials: ['SSH']) {
                sh '''
                ssh -o StrictHostKeyChecking=no alejandro@art <<EOF
                    cd ~/prueba_icdc || exit
                    docker-compose down
                    docker rmi -f aleromero/jenkins:v1
                    docker-compose up -d --force-recreate
                '''
            }
        }
    }
    }
    post {
        always {
            mail to: 'aletromp00@gmail.com',
            subject: "Pipeline IC: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}. Si ha tenido exito, se ha creado y subido la imagen $IMAGEN:latest."
        }
    }
}
