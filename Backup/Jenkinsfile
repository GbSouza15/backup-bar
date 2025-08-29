pipeline {
	// Define onde o pipeline será executado
	agent any
	
	// Define as ferramentas que serão usadas no pipeline
	tools {
	    // Assume que o Maven e o JDK já estão configurados no Jenkins
	    maven 'Maven'
	    jdk 'JDK 17'
	}

	// Define os estágios do pipeline
	stages {
	    // Estágio para fazer o build do projeto
	    stage('Build') {
	        steps {
	            // Executa o comando do Maven para fazer o build
	            echo 'Executando o build do projeto...'
	            sh 'mvn clean install'
	        }
	    }
	}
	
	// Define ações pós-build, como sucesso ou falha
	post {
	    // Executa se o build for bem-sucedido
	    success {
	        echo 'Build concluído com sucesso!'
	    }
	    // Executa se o build falhar
	    failure {
	        echo 'O build falhou. Verifique os logs para detalhes.'
	    }
	}
}