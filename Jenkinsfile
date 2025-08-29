pipeline {
    agent any

    // Define as ferramentas que serão usadas em todo o pipeline
    tools {
        jdk 'JDK 17'
        maven 'Maven'
    }

    stages {
        // Estágio 1: Baixa o código da sua aplicação (Backup)
        stage('Checkout Application SCM') {
            steps {
                echo 'Baixando o código-fonte do projeto...'
                cleanWs() 
                checkout scm
            }
        }

        // Estágio 2: Constrói o plugin a partir da branch para Java 17
        stage('Build and Install ACE Maven Plugin') {
            steps {
                echo 'Clonando a branch feature/java17 e construindo o ace-maven-plugin...'
                
                dir('ace-maven-plugin-build') {
                    git url: 'https://github.com/ot4i/ace-maven-plugin.git', branch: 'feature/java17'
                    
                    dir('ace-maven-plugin') {
                        // Constrói e instala o plugin no repositório local .m2 do Jenkins
                        bat '"%MAVEN_HOME%\\bin\\mvn" clean install'
                    }
                }
            }
        }

        // Estágio 3: Constrói a sua aplicação
        stage('Build Application') {
            steps {
                dir('Backup') {
                    echo 'Executando o build do projeto Backup...'
                    // CORREÇÃO FINAL: Usando a variável MAVEN_HOME para garantir que o Maven correto seja executado.
                    bat '"%MAVEN_HOME%\\bin\\mvn" clean install -U'
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
