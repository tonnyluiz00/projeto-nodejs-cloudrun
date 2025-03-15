
# Implementação Contínua com Docker, Cloud Build e Cloud Run

Este repositório demonstra como configurar uma pipeline de implementação contínua (CI/CD) usando Docker, Google Cloud Build e Google Cloud Run. Ele utiliza um projeto Node.js como exemplo, mas os conceitos podem ser aplicados a outras linguagens e aplicações.

## GitHub

* Repositório: [https://github.com/camilla-m/descomplicando-devops](https://github.com/camilla-m/descomplicando-devops)

## Tecnologias Utilizadas

* Docker
* Google Cloud Build
* Google Cloud Run

## Pré-requisitos

* Conta do Google Cloud Platform (GCP)
* Docker instalado
* Repositório no GitHub
* Repositório de exemplo: [https://github.com/camilla-m/descomplicando-devops](https://github.com/camilla-m/descomplicando-devops)

## Passos

### Passo 1: Criar um Projeto Node.js

1.  Siga este tutorial para criar um projeto Node.js e containerizá-lo com Docker: [https://docs.docker.com/guides/nodejs/containerize/](https://docs.docker.com/guides/nodejs/containerize/)

### Passo 2: Configurar o Google Cloud Platform (GCP)

1.  Acesse o GCP: [https://console.cloud.google.com/?inv=1&invt=AbsG8Q](https://console.cloud.google.com/?inv=1&invt=AbsG8Q)
2.  Crie um novo projeto ou selecione um projeto existente.
3.  Ative as APIs necessárias:
    * Cloud Build API: [https://cloud.google.com/build/docs/api/reference/rest](https://cloud.google.com/build/docs/api/reference/rest)
    * Cloud Run API

### Passo 3: Configurar a Pipeline no GCP

1.  Crie um arquivo `cloudbuild.yaml` na pasta raiz do seu projeto com o seguinte conteúdo:

    ```yaml
    steps:
      # Build the Docker image
      - name: 'gcr.io/cloud-builders/docker'
        args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/hello-repo/cicd-nodejs:1.0', '.']
      # Push the Docker image to Artifact Registry
      - name: 'gcr.io/cloud-builders/docker'
        args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/hello-repo/cicd-nodejs:1.0']
      # Deploy to Cloud Run
      - name: 'gcr.io/cloud-builders/gcloud'
        args:
          - 'run'
          - 'deploy'
          - 'minha-aplicacao-nodejs'
          - '--image=us-central1-docker.pkg.dev/$PROJECT_ID/hello-repo/nodejs-teste:1.0'
          - '--platform=managed'
          - '--region=us-central1'
          - '--allow-unauthenticated'
    options:
      logging: CLOUD_LOGGING_ONLY
    ```

2.  Substitua `cicd-nodejs` pelo nome da imagem Docker que você definiu e `minha-aplicacao-nodejs` pelo nome desejado para o serviço Cloud Run.
3.  Substitua `$PROJECT_ID` pelo ID do seu projeto no GCP.
4.  Salve o arquivo `cloudbuild.yaml`.

### Passo 4: Configurar a Integração com o GitHub

1.  No Google Cloud Console, vá para o serviço Cloud Build e selecione "Gatilhos".
2.  Clique em "Configurar conexão" e siga as instruções para conectar sua conta do GitHub.
3.  Selecione o repositório do GitHub que contém sua aplicação Node.js.

### Passo 5: Ativar a Pipeline de Implementação Automática

1.  No Google Cloud Console, vá para o serviço Cloud Build e selecione "Gatilhos".
2.  Clique em "Configurar disparador" para criar um novo disparador.
3.  Defina as condições de acionamento do disparador, por exemplo, quando houver alterações na branch principal do repositório do GitHub.
4.  Escolha a opção "Detecção automática" para usar o arquivo de configuração `cloudbuild.yaml` criado anteriormente.
5.  Clique em "Criar" para criar o disparador.

Agora, sempre que você fizer push para a branch principal do seu repositório do GitHub, a pipeline do Google Cloud Build será acionada. Ela construirá a imagem Docker, fará o push para o Artifact Registry e implantará no Cloud Run.

### Passo 6: Ativar Acesso Sem Autorização

1.  Após a implantação, você pode encontrar um erro 403 - Forbidden ao tentar acessar a aplicação. Para permitir acesso não autenticado, execute o seguinte comando no Cloud Shell:

    ```bash
    gcloud run services add-iam-policy-binding minha-aplicacao-nodejs \
    --member="allUsers" \
    --role="roles/run.invoker"
    ```

2.  Substitua `minha-aplicacao-nodejs` pelo nome do seu serviço Cloud Run.
3.  Selecione a região `us-central1` (opção 34).

## CI/CD - Finalizado

Com esses passos, você configurou uma pipeline de CI/CD completa para sua aplicação Node.js no Google Cloud Platform.