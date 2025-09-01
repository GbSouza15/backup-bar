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
            rem ==== ACE bin (8.3 p/ evitar espaços) ====
            for %%A in ("C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin") do set "ACE_BIN_S=%%~sA"
            set "PATH=%ACE_BIN_S%;%PATH%"
    
            rem ==== Carrega variáveis do ACE ====
            call "%ACE_BIN_S%\\mqsiprofile.cmd"
    
            rem ==== Caminhos ====
            set "APP=%WORKSPACE%\\Backup"
            set "TARGET=%APP%\\target"
            set "BAR=%TARGET%\\Backup.bar"
            set "TRACE=%TARGET%\\ibmint_package.trace"
    
            if not exist "%TARGET%" mkdir "%TARGET%"
    
            rem ==== Confere application.descriptor ====
            if not exist "%APP%\\application.descriptor" (
              echo ERRO: application.descriptor nao encontrado em "%APP%"
              exit /b 2
            )
    
            rem ==== PACKAGE (sem --log-level / --log-file) ====
            echo --- ibmint package ---
            ibmint package ^
              --input-path "%APP%" ^
              --output-bar-file "%BAR%" ^
              --java-version 17 ^
              --compile-maps-and-schemas ^
              --trace "%TRACE%"  2>&1
    
            set EC=%ERRORLEVEL%
            echo PACKAGE_EXITCODE=%EC%
            if %EC% NEQ 0 exit /b %EC%
    
            rem ==== Lista artefato gerado ====
            dir /-c "%TARGET%"
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
