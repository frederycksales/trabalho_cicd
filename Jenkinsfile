pipeline {
    agent any

    environment {
        NOME_PROJETO = 'ProjetoExemploCI'
        SECRET_KEY   = credentials('minha-chave-secreta-exemplo')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
    }

    parameters {
        string(name: 'NOME', defaultValue: 'Mundo', description: 'O nome para saudar.')
        choice(name: 'AMBIENTE_ALVO', choices: ['desenvolvimento', 'homologacao', 'producao'], description: 'Ambiente de deploy.')
    }

    triggers {
        cron('H/15 * * * *')
    }

    stages {
        stage('Preparação') {
            steps {
                echo "Iniciando pipeline para o projeto: ${env.NOME_PROJETO}"
                sh 'chmod +x app.sh test.sh'
            }
        }

        stage('Build & Teste') {
            parallel {
                stage('Build') {
                    steps {
                        echo "Iniciando build..."
                        sleep 5
                        echo "Build concluído."
                    }
                }
                stage('Teste') {
                    steps {
                        echo "Executando testes..."
                        sh './test.sh'
                    }
                }
            }
        }

        stage('Aprovação para Deploy') {
            when {
                expression { params.AMBIENTE_ALVO == 'producao' }
            }
            input {
                message "Deploy para produção requer aprovação. Continuar?"
                ok "Sim, fazer deploy"
            }
            steps {
                echo "Aprovação recebida. Continuando..."
            }
        }

        stage('Deploy por Matriz') {
            matrix {
                axes {
                    axis {
                        name 'REGIAO'
                        values 'us-east-1', 'sa-east-1', 'eu-west-1'
                    }
                }
                stages {
                    stage('Deploy Regional') {
                        steps {
                            echo "Iniciando deploy para a região: ${REGIAO}"
                            sh "export NOME='${params.NOME}'; export AMBIENTE_ALVO='${params.AMBIENTE_ALVO}'; ./app.sh"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizado."
        }
        success {
            echo "Pipeline executado com sucesso."
        }
        failure {
            echo "Pipeline falhou."
        }
        changed {
            echo "O estado do pipeline mudou."
        }
    }
}