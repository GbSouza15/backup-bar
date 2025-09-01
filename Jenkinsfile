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
        dir("${env.APP_DIR}") {
          bat """
            rem ===== PATH do ACE (usa nome 8.3 para evitar espacos) =====
            for %%A in ("${env.ACE_BIN}") do set "ACE_BIN_S=%%~sA"
            set "PATH=%ACE_BIN_S%;%PATH%"

            rem ===== Conferencias rapidas =====
            where ibmint
            ibmint version  2>&1

            rem ===== Confere se e uma Application ACE valida =====
            if not exist "%WORKSPACE%\\${env.APP_DIR}\\application.descriptor" (
              echo ERRO: application.descriptor nao encontrado em %WORKSPACE%\\${env.APP_DIR}
              exit /b 2
            )

            rem ===== Garante pasta de saida =====
            if not exist "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}" mkdir "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}"

            rem ===== Compile (mensagens mais claras) =====
            ibmint compile ^
              --input-path "%WORKSPACE%\\${env.APP_DIR}" ^
              --output-work-dir "%WORKSPACE%\\_out" ^
              --log-level debug ^
              --log-file "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\ibmint_compile.log"  2>&1
            if errorlevel 1 (
              type "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\ibmint_compile.log"
              exit /b 1
            )

            rem ===== Package =====
            ibmint package ^
              --input-path "%WORKSPACE%\\${env.APP_DIR}" ^
              --output-bar-file "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\${env.BAR_NAME}" ^
              --log-level debug ^
              --log-file "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\ibmint_package.log"  2>&1
            if errorlevel 1 (
              type "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\ibmint_package.log"
              exit /b 1
            )

            rem ===== Mostra logs no console (util para diagnostico) =====
            type "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\ibmint_compile.log"
            type "%WORKSPACE%\\${env.APP_DIR}\\${env.OUT_DIR}\\ibmint_package.log"
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
