pipeline {
    agent any

    tools {
        jdk 'JDK 17'
        maven 'Maven'
    }

    environment {
        // Repositório Maven local para este build
        MAVEN_LOCAL_REPO = "${WORKSPACE}/.m2/repository"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Limpando o workspace e baixando o código-fonte da aplicação...'
                cleanWs()
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                echo 'Executando o build do projeto Backup...'
                dir('Backup') {
                    echo "Iniciando o build dentro de: ${pwd()}"
                    bat "\"%MAVEN_HOME%\\bin\\mvn\" clean package -U -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build concluído com sucesso!'
            archiveArtifacts artifacts: 'Backup/builds/*.bar', fingerprint: true
        }
        failure {
            echo '❌ O build falhou. Verifique os logs para detalhes.'
        }
        always {
            echo '🧹 Limpando o workspace ao final do pipeline...'
            cleanWs()
        }
    }
}
