pipeline {
  agent any

  environment {
    // ↙ ajuste a versão/caminho do ACE se necessário
    ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\13.0.4.0'
    ACE_BIN  = "${ACE_HOME}\\server\\bin"

    // ↙ sua aplicação ACE no repositório
    APP_DIR  = 'Backup'
    OUT_DIR  = 'target'
    BAR_NAME = 'Backup.bar'
  }

  stages {

    stage('Checkout') {
      steps {
        echo '🧹 Limpando workspace...'
        cleanWs()

        echo '📥 Fazendo checkout do repositório...'
        checkout scm

        echo "📂 WORKSPACE: ${env.WORKSPACE}"
        bat 'dir /a /-c'
        bat "if exist \"${env.APP_DIR}\" (pushd \"${env.APP_DIR}\" & cd & dir /a /-c & popd) else (echo Pasta \"${env.APP_DIR}\" nao encontrada)"
      }
    }

    stage('Build BAR (ibmint)') {
      steps {
        dir('Backup') {
          bat """
            @echo on
            rem ==== ACE bin (8.3 para evitar espacos) ====
            for %%A in ("C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin") do set "ACE_BIN_S=%%~sA"
            set "PATH=%ACE_BIN_S%;%PATH%"
    
            rem ==== Carrega variaveis de ambiente do ACE ====
            call "%ACE_BIN_S%\\mqsiprofile.cmd"
    
            rem ==== Diag rapido ====
            echo --- whoami ---
            whoami
            echo --- where ibmint ---
            where ibmint
    
            rem (idempotente) aceita licenca se existir ace.exe
            if exist "%ACE_BIN_S%\\ace.exe" "%ACE_BIN_S%\\ace.exe" license --accept  2>&1
    
            rem ==== Caminhos ====
            set "APP=%WORKSPACE%\\Backup"
            set "OUT=%WORKSPACE%\\_out"
            set "TARGET=%APP%\\target"
            set "BAR=%TARGET%\\Backup.bar"
            set "LOGC=%TARGET%\\ibmint_compile.log"
            set "LOGP=%TARGET%\\ibmint_package.log"
    
            if not exist "%OUT%"    mkdir "%OUT%"
            if not exist "%TARGET%" mkdir "%TARGET%"
    
            rem ==== Confere application.descriptor ====
            if not exist "%APP%\\application.descriptor" (
              echo ERRO: application.descriptor nao encontrado em "%APP%"
              exit /b 2
            )
    
            rem ==== Versao/help (mostra se o binario responde) ====
            ibmint help      2>&1
            ibmint --version 2>&1
    
            rem ==== COMPILE (com log em arquivo) ====
            echo --- ibmint compile ---
            ibmint compile ^
              --input-path "%APP%" ^
              --output-work-dir "%OUT%" ^
              --log-level debug ^
              --log-file "%LOGC%"  2>&1
            set EC=%ERRORLEVEL%
            echo COMPILE_EXITCODE=%EC%
            if exist "%LOGC%" type "%LOGC%"
            if %EC% NEQ 0 exit /b %EC%
    
            rem ==== PACKAGE (com log em arquivo) ====
            echo --- ibmint package ---
            ibmint package ^
              --input-path "%APP%" ^
              --output-bar-file "%BAR%" ^
              --log-level debug ^
              --log-file "%LOGP%"  2>&1
            set EC=%ERRORLEVEL%
            echo PACKAGE_EXITCODE=%EC%
            if exist "%LOGP%" type "%LOGP%"
            if %EC% NEQ 0 exit /b %EC%
    
            dir /b "%TARGET%"
          """
        }
      }
    }

  }

  post {
    success {
      echo '✅ Build concluído com sucesso!'
      archiveArtifacts artifacts: "${env.APP_DIR}/${env.OUT_DIR}/*.bar", fingerprint: true
    }
    always {
      echo '🧹 Limpando workspace...'
      cleanWs()
    }
  }
}
