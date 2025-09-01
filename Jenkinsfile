pipeline {
    agent any

    tools {
        jdk 'JDK 17'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '🧹 Limpando e clonando...'
                cleanWs()
                checkout scm
            }
        }

        stage('Build BAR (ibmint)') {
            steps {
                dir('Backup') {
                    echo "📦 Gerando BAR em: ${pwd()}\\target"
                    bat """
                        set "ACE_BIN=C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin"
                        set "PATH=%ACE_BIN%;%PATH%"
                        ibmint version

                        ibmint package ^
                          --input-path ${WORKSPACE}\\Backup ^
                          --output-bar-file ${WORKSPACE}\\Backup\\target\\Backup.bar
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build OK!'
            archiveArtifacts artifacts: 'Backup/target/*.bar', fingerprint: true
        }
        failure {
            echo '❌ Falhou. Veja os logs do stage "Build BAR (ibmint)".'
        }
        always {
            echo '🧹 Limpando workspace...'
            cleanWs()
        }
    }
}
