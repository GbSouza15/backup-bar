pipeline {
    agent any

    tools {
        jdk 'JDK 17'
        maven 'Maven'
    }

    environment {
        // O repositório local ainda é uma boa prática para isolamento
        MAVEN_LOCAL_REPO = "${workspace}/.m2/repository"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Baixando o código-fonte...'
                cleanWs() 
                checkout scm
                
                // Precisamos clonar o plugin aqui também, pois ele faz parte do build
                dir('ace-maven-plugin-build') {
                    git url: 'https://github.com/ot4i/ace-maven-plugin.git', branch: 'feature/java17'
                }
            }
        }

        stage('Build All') {
            steps {
                echo 'Executando o build multi-módulo...'
                // Executa um único comando a partir da raiz
                bat "\"%MAVEN_HOME%\\bin\\mvn\" clean install -U -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
            }
        }
    }
    
    post {
        success {
            echo 'Build concluído com sucesso!'
            // O artefato .bar ainda estará no mesmo lugar
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
