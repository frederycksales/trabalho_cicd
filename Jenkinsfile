
pipeline {
    // AGENT: Define onde o pipeline será executado. 'any' significa qualquer agente disponível.
    agent any

    // ENVIRONMENT: Define variáveis de ambiente para todo o pipeline.
    environment {
        // Variável de ambiente simples
        NOME_PROJETO = 'ProjetoExemploCI'
        // Usando credenciais como variável de ambiente
        // A 'SECRET_KEY' deve ser uma credencial do tipo "Secret Text" no Jenkins
        SECRET_KEY = credentials('minha-chave-secreta-exemplo')
    }

    // OPTIONS: Configurações específicas do pipeline.
    options {
        // Guarda os 5 últimos builds
        buildDiscarder(logRotator(numToKeepStr: '5'))
        // Adiciona timestamps nos logs do console
        timestamps()
    }

    // PARAMETERS: Permite que o usuário insira valores ao iniciar o pipeline.
    parameters {
        string(name: 'NOME', defaultValue: 'Mundo', description: 'O nome para saudar.')
        choice(name: 'AMBIENTE_ALVO', choices: ['desenvolvimento', 'homologacao', 'producao'], description: 'Ambiente de deploy.')
    }

    // TRIGGERS: Define o que dispara o pipeline automaticamente.
    triggers {
        // Executa a cada 15 minutos (sintaxe cron)
        cron('H/15 * * * *')
    }

    stages {
        stage('Preparação') {
            steps {
                script {
                    echo "Iniciando o pipeline para o projeto: ${env.NOME_PROJETO}"
                    echo "Parâmetros recebidos: NOME=${params.NOME}, AMBIENTE_ALVO=${params.AMBIENTE_ALVO}"
                    // Torna os scripts executáveis
                    sh 'chmod +x app.sh test.sh'
                }
            }
        }

        stage('Build & Teste') {
            // PARALLEL: Executa estágios em paralelo para otimizar o tempo.
            parallel {
                stage('Build') {
                    steps {
                        echo "Iniciando o build..."
                        sleep 5 // Simula um processo de build
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
            // WHEN: Condição para executar o estágio.
            // Só executa se o ambiente alvo for 'producao'.
            when {
                expression { params.AMBIENTE_ALVO == 'producao' }
            }
            // INPUT: Pausa o pipeline e aguarda a aprovação do usuário.
            input {
                message "O deploy para produção requer aprovação manual. Continuar?"
                ok "Sim, fazer deploy para produção"
            }
            steps {
                echo "Aprovação recebida. Continuando com o deploy..."
            }
        }

        stage('Deploy por Matriz') {
            // MATRIX: Executa o mesmo estágio com diferentes combinações de parâmetros.
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
                            echo "Iniciando deploy para a região: ${REGIAO}..."
                            // Passa os parâmetros para o script de "deploy"
                            sh "export NOME='${params.NOME}'; export AMBIENTE_ALVO='${params.AMBIENTE_ALVO}'; ./app.sh"
                        }
                        // NOT: Condição para executar um passo apenas se a condição NÃO for verdadeira.
                        post {
                            success {
                                script {
                                    if (params.AMBIENTE_ALVO != 'producao') {
                                        echo "Deploy em ambiente de não-produção concluído."
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }

    // POST: Ações a serem executadas no final do pipeline.
    post {
        always {
            echo "Pipeline finalizado."
        }
        success {
            echo "O pipeline foi executado com sucesso!"
        }
        failure {
            echo "O pipeline falhou."
        }
        unstable {
            echo "O pipeline está instável (testes podem ter falhado)."
        }
        changed {
            echo "O estado do pipeline mudou desde a última execução."
        }
    }
}
