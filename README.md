# Projeto de Pipeline CI/CD com Jenkins

Este repositório contém um exemplo de pipeline declarativo (`Jenkinsfile`) que demonstra diversas funcionalidades de CI/CD, como stages, paralelismo, matriz de builds, parâmetros e aprovação manual.

O pipeline orquestra scripts simples de build e deploy (`app.sh`, `test.sh`).

## Como Executar

### Pré-requisitos

- Docker
- Docker Compose

### 1. Iniciar o Jenkins

O `docker-compose.yml` na raiz do projeto sobe uma instância do Jenkins com a versão LTS recomendada.

```bash
docker-compose up -d
```

- Acesse a interface do Jenkins em `http://localhost:8080`.
- A senha inicial pode ser obtida com o comando: `docker-compose logs jenkins`.

### 2. Configurar a Credencial

O pipeline requer uma credencial do tipo "Secret Text" para simular o uso de chaves de API.

1.  No painel do Jenkins, navegue até **Manage Jenkins > Credentials > System > Global credentials**.
2.  Clique em **Add Credentials**.
3.  **Kind**: `Secret text`
4.  **Secret**: `12345` (ou qualquer outro valor)
5.  **ID**: `minha-chave-secreta-exemplo`
6.  Salve a credencial.

### 3. Criar o Job no Jenkins

1.  Na página inicial, clique em **New Item**.
2.  Dê um nome ao job (ex: `projeto-cicd`) e selecione o tipo **Pipeline**.
3.  Na aba de configuração, vá até a seção **Pipeline**.
4.  Altere a **Definition** para `Pipeline script from SCM`.
5.  Em **SCM**, selecione `Git`.
6.  **Repository URL**: Informe a URL deste repositório Git.
7.  **Script Path**: Mantenha o padrão `Jenkinsfile`.
8.  Salve o job.

### 4. Executar o Pipeline

1.  Na página do job, clique em **Build with Parameters**.
2.  Escolha os parâmetros desejados (ex: `AMBIENTE_ALVO`).
3.  Clique em **Build**.

O pipeline será executado. Se o ambiente `producao` for selecionado, o pipeline pausará para aprovação manual antes de continuar.
