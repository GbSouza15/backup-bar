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
        
                rem Garante a pasta de saída
                if not exist "%WORKSPACE%\\Backup\\target" mkdir "%WORKSPACE%\\Backup\\target"
        
                rem Verifica versão (mostra se o PATH está certo)
                ibmint version 2>&1
        
                rem Build com log detalhado e captura de stderr
                ibmint package ^
                  --input-path "%WORKSPACE%\\Backup" ^
                  --output-bar-file "%WORKSPACE%\\Backup\\target\\Backup.bar" ^
                  --log-level debug ^
                  --log-file "%WORKSPACE%\\Backup\\target\\ibmint_build.log"  2>&1
        
                echo EXITCODE=%ERRORLEVEL%
        
                rem Mostra o log detalhado do ibmint
                if exist "%WORKSPACE%\\Backup\\target\\ibmint_build.log" type "%WORKSPACE%\\Backup\\target\\ibmint_build.log"
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
