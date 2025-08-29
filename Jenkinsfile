pipeline {
    agent any

    tools {
        jdk 'JDK 17'
        maven 'Maven'
    }

    environment {
        // Define um repositório Maven local isolado para este build
        MAVEN_LOCAL_REPO = "${workspace}/.m2/repository"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Limpando o workspace e baixando o código-fonte da aplicação...'
                cleanWs() 
                // Faz o checkout do seu repositório que contém o projeto 'Backup'
                checkout scm 
            }
        }

        stage('Build and Install ACE Plugin') {
            steps {
                echo 'Clonando e instalando o ace-maven-plugin...'
                // Cria um diretório temporário para o build do plugin
                dir('ace-maven-plugin-build') {
                    // Clona o repositório do plugin
                    git url: 'https://github.com/ot4i/ace-maven-plugin.git', branch: 'feature/java17'
                    
                    // Entra no diretório correto do módulo do plugin para o build
                    dir('ace-maven-plugin') {
                        // Executa o build e instala o plugin no repositório local do workspace
                        // Este é um processo Maven separado e independente
                        bat "\"%MAVEN_HOME%\\bin\\mvn\" clean install -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                echo 'Executando o build do projeto Backup...'
                // Entra no diretório do seu projeto que foi baixado no estágio 'Checkout'
                dir('Backup') {
                    // Executa o build da aplicação. 
                    // Este novo processo Maven encontrará o plugin no repositório local.
                    // O -U força a verificação de atualizações de snapshots.
                    bat "\"%MAVEN_HOME%\\bin\\mvn\" clean install -U -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build concluído com sucesso!'
            // Arquiva o artefato .bar gerado
            archiveArtifacts artifacts: 'Backup/target/*.bar', fingerprint: true
        }
        failure {
            echo 'O build falhou. Verifique os logs para detalhes.'
        }
        always {
            echo 'Limpando o workspace ao final do pipeline...'
            cleanWs()
        }
    }
}

