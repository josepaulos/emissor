pipeline {
    agent any
    stages {
        stage('Verificar Repositório') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], useRemoteConfigs: [[url: 'https://github.com/josepaulos/emissor.git']]])
            }
        }

        stage('Instalar Dependências') {
            steps {
                script {
                    // Atualiza o PATH se necessário
                    env.PATH = "/usr/bin:$PATH"
                    // Instalar as dependências Maven antes de compilar o projeto
                    bat 'mvn clean install'  // Instala as dependências do Maven
                }
            }
        }

        stage('Análise SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {  // 'SonarQubeServer' é o nome da instância configurada no Jenkins
                        bat 'mvn clean verify sonar:sonar -Dsonar.projectKey=projeto-Dsonar.projectName=cadastro-app'
                    }
                }
            }
        }

        stage('Construir Imagem Docker') {
            steps {
                script {
                    def appName = 'cadastro-app'
                    def imageTag = "${appName}:${env.BUILD_ID}"

                    // Construir a imagem Docker
                    bat "docker build -t ${imageTag} ."
                }
            }
        }

        stage('Fazer Deploy') {
            steps {
                script {
                    def appName = 'cadastro'
                    def imageTag = "${appName}:${env.BUILD_ID}"

                    // Parar e remover o container existente, se houver
            		bat "docker stop ${appName} || exit 0"
            		bat "docker rm -v ${appName} || exit 0"  // Remover o container e os volumes associados

            		// Parar e remover os containers do mysql e rabbitmq, se estiverem em uso
            		bat "docker stop mysql rabbitmq || exit 0"
            		bat "docker rm -v mysql rabbitmq || exit 0"

                    // Executar o novo container
                    bat "docker-compose up -d --build"
                }
            }
        }
    }
    post {
        success {
            echo 'Deploy realizado com sucesso!'
        }
        failure {
            echo 'Houve um erro durante o deploy.'
        }
    }
}