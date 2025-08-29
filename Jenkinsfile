pipeline {
    agent any

    // A definição global de ferramentas foi removida para que cada estágio possa especificar a sua.
    // Isso permite usar JDK 8 para o plugin e outra versão para a aplicação, se necessário.

    stages {
        // Estágio 1: Baixa o código da sua aplicação (Backup)
        stage('Checkout Application SCM') {
            steps {
                echo 'Baixando o código-fonte do projeto...'
                // Limpa o workspace para garantir que não haja arquivos de clones anteriores
                cleanWs() 
                checkout scm
            }
        }

        // Estágio 2: Constrói o plugin que está faltando
        stage('Build and Install ACE Maven Plugin') {
            // Este estágio PRECISA usar JDK 8, conforme a documentação do plugin.
            tools {
                // **IMPORTANTE**: Certifique-se de que 'JDK 8' é o nome exato da sua
                // configuração do JDK 8 em "Manage Jenkins" -> "Global Tool Configuration"
                jdk 'JDK 8' 
                maven 'Maven'
            }
            steps {
                echo 'Clonando e construindo o ace-maven-plugin a partir do fonte...'
                
                // Cria um diretório temporário para não misturar com o código da sua aplicação
                dir('ace-maven-plugin-build') {
                    // Clona o repositório do plugin
                    git url: 'https://github.com/ot4i/ace-maven-plugin.git', branch: 'master'
                    
                    // Constrói e instala o plugin no repositório local .m2 do Jenkins
                    bat 'mvn clean install'
                }
            }
        }

        // Estágio 3: Constrói a sua aplicação
        stage('Build Application') {
            // Este estágio também usará o JDK 8 para manter a consistência,
            // pois o ambiente IBM ACE é historicamente sensível à versão do Java.
            tools {
                jdk 'JDK 8'
                maven 'Maven'
            }
            steps {
                // Navega para o diretório do seu projeto que foi baixado no primeiro stage
                dir('Backup') {
                    echo 'Executando o build do projeto Backup...'
                    // Agora o Maven encontrará o plugin no repositório local e o build funcionará
                    bat 'mvn clean install -U'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build concluído com sucesso!'
            // Arquiva o .bar gerado
            archiveArtifacts artifacts: 'Backup/target/*.bar', fingerprint: true
        }
        failure {
            echo 'O build falhou. Verifique os logs para detalhes.'
        }
        always {
            // Limpa o workspace no final de cada execução para garantir um ambiente limpo
            echo 'Limpando o workspace...'
            cleanWs()
        }
    }
}
