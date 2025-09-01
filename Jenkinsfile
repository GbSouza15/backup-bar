pipeline {
  agent any

  environment {
    // Ajuste se seu ACE estiver em outro caminho/versão
    ACE_BIN = 'C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin'
    APP_DIR = 'Backup'                  // pasta da Application no repo
    OUT_DIR = 'target'                  // pasta de saída do BAR
    BAR_NAME = 'Backup.bar'             // nome do arquivo .bar
  }

  stages {
    stage('Checkout') {
      steps {
        echo '🧹 Limpando workspace...'
        cleanWs()

        echo '📥 Fazendo checkout do repositório...'
        checkout scm

        echo "📂 WORKSPACE atual: ${env.WORKSPACE}"
        echo '📄 Listando conteúdo da raiz do workspace:'
        bat 'dir /a /-c'

        echo "📄 Listando conteúdo da pasta ${env.APP_DIR} (se existir):"
        bat "if exist \"${env.APP_DIR}\" (pushd ${env.APP_DIR} & cd & dir /a /-c & popd) else (echo Pasta \"${env.APP_DIR}\" nao encontrada no repo)"
      }
    }

    stage('Build BAR (ibmint)') {
      steps {
        dir('Backup') {
          bat """
            rem ===== PATH do ACE (usa nome 8.3 p/ evitar espacos) =====
            for %%A in ("C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin") do set "ACE_BIN_S=%%~sA"
            set "PATH=%ACE_BIN_S%;%PATH%"
    
            rem ===== Conferencias rapidas =====
            echo --- whoami ---
            whoami
            echo --- where ibmint / ace ---
            where ibmint
            where ace
    
            rem ===== Checa/aceita licenca no usuario de servico =====
            if exist "%ACE_BIN_S%\\ace.exe" (
              echo --- ace license --status ---
              "%ACE_BIN_S%\\ace.exe" license --status  2>&1 || (
                echo --- ace license --accept ---
                "%ACE_BIN_S%\\ace.exe" license --accept  2>&1
              )
            )
    
            rem ===== Verifica Application ACE =====
            if not exist "%WORKSPACE%\\Backup\\application.descriptor" (
              echo ERRO: application.descriptor nao encontrado em %WORKSPACE%\\Backup
              exit /b 2
            )
    
            rem ===== Pre-cria pastas de trabalho/saida (evita erro de permissao) =====
            if not exist "%WORKSPACE%\\_out" mkdir "%WORKSPACE%\\_out"
            if not exist "%WORKSPACE%\\Backup\\target" mkdir "%WORKSPACE%\\Backup\\target"
    
            rem ===== Mostra versao (confirma PATH) =====
            echo --- ibmint version ---
            ibmint version 2>&1
    
            rem ===== COMPILE (sem --log-file; imprime direto no console) =====
            echo --- ibmint compile ---
            ibmint compile ^
              --input-path "%WORKSPACE%\\Backup" ^
              --output-work-dir "%WORKSPACE%\\_out" ^
              --log-level debug  2>&1
            echo COMPILE_EXITCODE=%ERRORLEVEL%
            if errorlevel 1 exit /b %ERRORLEVEL%
    
            rem ===== PACKAGE (idem, log no console) =====
            echo --- ibmint package ---
            ibmint package ^
              --input-path "%WORKSPACE%\\Backup" ^
              --output-bar-file "%WORKSPACE%\\Backup\\target\\Backup.bar" ^
              --log-level debug  2>&1
            echo PACKAGE_EXITCODE=%ERRORLEVEL%
            if errorlevel 1 exit /b %ERRORLEVEL%
    
            rem ===== Lista artefato gerado =====
            dir /b "%WORKSPACE%\\Backup\\target"
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
