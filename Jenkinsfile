pipeline {
  agent any

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

        echo '📄 Listando conteúdo da pasta Backup (se existir):'
        bat 'if exist "Backup" (pushd Backup & cd & dir /a /-c & popd) else (echo Pasta "Backup" nao encontrada no repo)'
      }
    }
  }

  post {
    always {
      echo '✅ Fim do teste de checkout.'
    }
  }
}
