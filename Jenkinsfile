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
                checkout scm 
            }
        }

        stage('Build and Install ACE Plugin') {
            steps {
                echo 'Clonando e instalando o ace-maven-plugin...'
                dir('ace-maven-plugin-build') {
                    git url: 'https://github.com/ot4i/ace-maven-plugin.git', branch: 'feature/java17'
                    
                    dir('ace-maven-plugin') {
                        bat "\"%MAVEN_HOME%\\bin\\mvn\" clean install -Dmaven.repo.local=\"%MAVEN_LOCAL_REPO%\""
                        
                        // NOVO: 'Empacota' o plugin construído para uso no próximo estágio
                        stash name: 'ace-plugin', includes: 'target/*.jar'
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                echo 'Executando o build do projeto Backup...'
                
                // NOVO: 'Desempacota' o plugin no diretório da aplicação
                // Embora já esteja no repo local, isso garante que ele está acessível
                unstash 'ace-plugin'

                dir('Backup') {
                    echo "Iniciando o build dentro de: ${pwd()}"
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
            echo 'Limpando o workspace ao final do pipeline...'
            cleanWs()
        }
    }
}

