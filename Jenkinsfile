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
        dir("${env.APP_DIR}") {
          bat """
            @echo on
            rem ==== Converte ACE_BIN p/ 8.3 e coloca no PATH ====
            for %%A in ("${env.ACE_BIN}") do set "ACE_BIN_S=%%~sA"
            set "PATH=%ACE_BIN_S%;%PATH%"

            rem ==== Variaveis de caminho ====
            set "APP=%WORKSPACE%\\${env.APP_DIR}"
            set "OUT=%WORKSPACE%\\_out"
            set "TARGET=%APP%\\${env.OUT_DIR}"
            set "BAR=%TARGET%\\${env.BAR_NAME}"

            echo --- whoami ---
            whoami
            echo --- where ibmint ---
            where ibmint

            rem ==== Aceita licenca (idempotente) ====
            if exist "%ACE_BIN_S%\\ace.exe" "%ACE_BIN_S%\\ace.exe" license --accept  2>&1

            rem ==== Valida Application ACE ====
            if not exist "%APP%\\application.descriptor" (
              echo ERRO: application.descriptor nao encontrado em "%APP%"
              exit /b 2
            )

            rem ==== Garante pastas ====
            if not exist "%OUT%"    mkdir "%OUT%"
            if not exist "%TARGET%" mkdir "%TARGET%"

            rem ==== Compile ====
            echo --- ibmint compile ---
            ibmint compile ^
              --input-path "%APP%" ^
              --output-work-dir "%OUT%" ^
              --log-level debug  2>&1
            if errorlevel 1 exit /b %ERRORLEVEL%

            rem ==== Package ====
            echo --- ibmint package ---
            ibmint package ^
              --input-path "%APP%" ^
              --output-bar-file "%BAR%" ^
              --log-level debug  2>&1
            if errorlevel 1 exit /b %ERRORLEVEL%

            rem ==== Lista artefato ====
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
