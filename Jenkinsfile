pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        echo '🧹 Limpando workspace...'
        cleanWs()
        echo '📥 Fazendo checkout do repositório...'
        checkout scm
        echo "📂 WORKSPACE: ${env.WORKSPACE}"
        bat 'dir /a /-c'
        bat 'if exist "Backup" (pushd "Backup" & cd & dir /a /-c & popd) else (echo Pasta "Backup" nao encontrada )'
      }
    }

    stage('Build BAR (ibmint)') {
      steps {
        dir('Backup') {
          bat """
            rem ==== ACE bin (8.3 p/ evitar espacos) ====
            for %%A in ("C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin") do set "ACE_BIN_S=%%~sA"
            set "PATH=%ACE_BIN_S%;%PATH%"

            rem ==== Carrega variaveis do ACE ====
            call "%ACE_BIN_S%\\mqsiprofile.cmd"

            rem ==== Caminhos ====
            set "APP=%WORKSPACE%\\Backup"
            set "TARGET=%APP%\\target"
            set "BAR=%TARGET%\\Backup.bar"
            set "TRACE=%TARGET%\\ibmint_package.trace"
            if not exist "%TARGET%" mkdir "%TARGET%"

            rem ==== Package ====
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

            dir /-c "%TARGET%"
          """
        }
      }
    }
  }

  post {
    success {
      echo '✅ Build concluído com sucesso!'
      // ARQUIVA ANTES DE LIMPAR
      archiveArtifacts artifacts: 'Backup/target/*.bar', fingerprint: true, onlyIfSuccessful: true
    }
    always {
      echo '🧹 Limpando workspace...'
      cleanWs()
    }
  }
}
