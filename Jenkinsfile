pipeline {
  agent any
  tools { jdk 'JDK 17' }

  stages {
    stage('Checkout') {
      steps {
        cleanWs()
        checkout scm
      }
    }

    stage('Build BAR (ibmint)') {
      steps {
        dir('Backup') {
          bat """
            rem ===== Quem sou eu? (confirma usuário do serviço Jenkins) =====
            whoami

            rem ===== PATH do ACE (8.3 para evitar espaços) =====
            for %%A in ("C:\\Program Files\\IBM\\ACE\\13.0.4.0\\server\\bin") do set "ACE_BIN=%%~sA"
            set "PATH=%ACE_BIN%;%PATH%"

            rem ===== Onde está o ibmint/ace? =====
            where ibmint
            where ace

            rem ===== Status/aceite de licença (tem que ser no MESMO usuário) =====
            ace license --status  2>&1
            if errorlevel 1 (
              echo Aceitando licenca para o usuario atual do servico Jenkins...
              ace license --accept  2>&1
              if errorlevel 1 (
                echo ERRO: falha ao aceitar a licenca para este usuario.
                exit /b 10
              )
            )

            rem ===== Diagnostico da pasta do projeto =====
            echo Conteudo de %WORKSPACE%\\Backup:
            dir /a /b "%WORKSPACE%\\Backup"
            echo Procurando application.descriptor (recursivo):
            dir /a /b /s "%WORKSPACE%\\Backup\\application.descriptor"

            if not exist "%WORKSPACE%\\Backup\\application.descriptor" (
              echo ERRO: Nao encontrei "%WORKSPACE%\\Backup\\application.descriptor".
              echo Verifique se a raiz da Application ACE esta mesmo em %WORKSPACE%\\Backup
              echo ou ajuste --input-path para a pasta correta.
              exit /b 2
            )

            rem ===== Garante pasta de saida =====
            if not exist "%WORKSPACE%\\Backup\\target" mkdir "%WORKSPACE%\\Backup\\target"

            rem ===== Versao do ibmint (confirma PATH) =====
            ibmint version  2>&1

            rem ===== Compile (mensagens mais claras) =====
            ibmint compile ^
              --input-path "%WORKSPACE%\\Backup" ^
              --output-work-dir "%WORKSPACE%\\_out" ^
              --log-level debug ^
              --log-file "%WORKSPACE%\\Backup\\target\\ibmint_compile.log"  2>&1

            echo COMPILE_EXITCODE=%ERRORLEVEL%
            if exist "%WORKSPACE%\\Backup\\target\\ibmint_compile.log" type "%WORKSPACE%\\Backup\\target\\ibmint_compile.log"
            if errorlevel 1 exit /b %ERRORLEVEL%

            rem ===== Package =====
            ibmint package ^
              --input-path "%WORKSPACE%\\Backup" ^
              --output-bar-file "%WORKSPACE%\\Backup\\target\\Backup.bar" ^
              --log-level debug ^
              --log-file "%WORKSPACE%\\Backup\\target\\ibmint_package.log"  2>&1

            echo PACKAGE_EXITCODE=%ERRORLEVEL%
            if exist "%WORKSPACE%\\Backup\\target\\ibmint_package.log" type "%WORKSPACE%\\Backup\\target\\ibmint_package.log"
            if errorlevel 1 exit /b %ERRORLEVEL%
          """
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'Backup/target/*.bar', fingerprint: true
    }
    always {
      cleanWs()
    }
  }
}
