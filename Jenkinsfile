pipeline {
    agent any

    tools {
        jdk 'JDK 17'
        maven 'Maven'
    }

    // Define uma variável para o repositório Maven local dentro do workspace
    environment {
        MAVEN_LOCAL_REPO = "${workspace}/.m2/repository"
    }

    stages {
        stage('Checkout Application SCM') {
            steps {
                echo 'Baixando o código-fonte do projeto...'
                cleanWs() 
                checkout scm
            }
        }

        stage('Build and Install ACE Maven Plugin') {
            steps {
                echo 'Clonando a branch feature/java17 e construindo o ace-maven-plugin...'
                
                dir('ace-maven-plugin-build') {
                    git url: 'https://github.com/ot4i/ace-maven-plugin.git', branch: 'feature/java17'
                    
                    dir('ace-maven-plugin') {
                        // CORREÇÃO: Força a instalação no repositório local do workspace
                        bat "\"%MAVEN_HOME%\\bin\\mvn\" clean install -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                dir('Backup') {
                    echo 'Executando o build do projeto Backup...'
                    // CORREÇÃO: Força a leitura do repositório local do workspace
                    bat "\"%MAVEN_HOME%\\bin\\mvn\" clean install -U -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build concluído com sucesso!'
            archiveArtifacts artifacts: 'Backup/target/*.bar', fingerprint: true
        }
        failure {
            echo 'O build falhou. Verifique os logs para detalhes.'
        }
        always {
            echo 'Limpando o workspace...'
            cleanWs()
        }
    }
}
