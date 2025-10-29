# Projeto Website Estático com CI/CD na AWS

# [Aws Cloud Foundations]

Este repositório contém um website estático em HTML e demonstra como configurar uma pipeline de CI/CD utilizando GitHub Actions para fazer o deploy automático para um bucket S3 na AWS.

## Estrutura do Repositório

- **.github/workflows**: Contém os arquivos de workflow do GitHub Actions (ex.: `main.yml`) que orquestram o processo de deploy.
- **website**: Pasta que contém os arquivos do site (HTML, CSS, JS e imagens).
- **.gitignore**: Define os arquivos e pastas que não serão versionados.
- **README.md**: Este arquivo, com informações importantes sobre o projeto e o fluxo de deploy.

## Pontos Importantes

1. **Deploy Automático com GitHub Actions**

   - Sempre que houver um push para o branch `main`, o workflow é acionado.
   - O workflow faz o checkout do repositório, remove pastas desnecessárias (como `.git` e `.github`) e sincroniza apenas a pasta `website` com o bucket S3.
   - O deploy é feito utilizando a ação `jakejarvis/s3-sync-action` sem aplicar ACLs, garantindo compatibilidade com buckets com Object Ownership configurado como "Bucket owner enforced".

2. **Configuração do Bucket S3 para Hospedagem de Site Estático**

   - O bucket S3 foi configurado para hospedar um site estático.
   - Foi habilitada a opção de "Static website hosting", definindo o documento de índice (ex.: `index.html`) e, se necessário, o documento de erro.
   - Uma política de bucket foi aplicada para permitir acesso público de leitura aos arquivos.

3. **Variáveis e Segredos do AWS**

   - Os segredos (como `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` e `S3_BUCKET`) estão configurados em um Environment no GitHub, e são injetados no workflow durante a execução.

4. **.gitignore**
   - O arquivo `.gitignore` garante que arquivos desnecessários (como logs, pastas de dependências, arquivos de sistema e configurações de IDE) não sejam versionados.

## Instruções para Alunos

- **Entender o Pipeline:**  
   Estudem como o GitHub Actions é configurado no arquivo de workflow para automatizar o processo de deploy. Observem os passos de checkout, limpeza do repositório e sincronização com o S3.
  O arquivo que define tudo isto esta em: .github
  /workflows/main.yml

        name: Deploy Website to S3

        on:
        push:
        branches: - main

        jobs:
        deploy:
        environment: AWS
        runs-on: ubuntu-latest
        steps: - name: Checkout do repositório
        uses: actions/checkout@v3
        with:
        fetch-depth: 1

            - name: Remover pastas indesejadas
                run: |
                rm -rf .git
                rm -rf .github

            - name: Sincronizar arquivos com o S3
                uses: jakejarvis/s3-sync-action@v0.5.1
                with:
                args: --delete --exclude ".git/*" --exclude ".github/*"
                env:
                AWS_S3_BUCKET: ${{ secrets.S3_BUCKET }}
                AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
                AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
                AWS_REGION: ${{ secrets.AWS_REGION }}
                SOURCE_DIR: "website/"

![alt text](image.png)

- **Configurar o Bucket S3:**  
  Verifiquem as configurações do bucket S3 para hospedagem de site estático e a política de acesso público.
  Aplicar a politica no seu bucket, para não permitir somente que leiam os arquivos e não apaguem:

         {
            "Version": "2012-10-17",
            "Statement": [
               {
                  "Sid": "PublicReadGetObject",
                  "Effect": "Allow",
                  "Principal": "*",
                  "Action": "s3:GetObject",
                  "Resource": "arn:aws:s3:::meu-website-static/*"
               }
            ]
         }

  Habilitar em propriedades para ser um repositorio de site estático:
  ![alt text](image-1.png)

- **Ajustar o Repositório:**  
  Caso necessário, modifiquem a estrutura ou os arquivos do website na pasta `website` e observem como o deploy é refletido automaticamente após cada push para o branch `main`.

Este repositório é um exemplo prático de como integrar GitHub e AWS para automatizar o deploy de um site estático. Aproveitem para explorar e ajustar conforme as necessidades dos seus projetos!
