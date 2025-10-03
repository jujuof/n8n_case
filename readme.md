# Documentação do Workflow n8n – Processamento de E-mails Financeiros

**Autor:** Júlia Lana

## 1. Objetivo

Este workflow automatiza o processo de recebimento de e-mails com anexos financeiros em formato CSV. O objetivo é extrair o anexo, salvá-lo em um repositório na nuvem (Google Drive), consultar a cotação do dólar do dia e enviar um e-mail de confirmação para o remetente original.

## 2. Visão Geral do Fluxo

O workflow foi estruturado na seguinte ordem lógica para garantir eficiência e simplicidade no fluxo de dados:

1.  **Gatilho de E-mail (IMAP):** O fluxo é iniciado sempre que um novo e-mail nomeado como "Planilha financeira" com anexo chega na caixa de entrada monitorada. **Importante:** O e-mail deve conter como assunto "Planilha financeira" (com ou sem anexo).

2.  **Se (IF):** O fluxo verifica se o anexo recebido possui a extensão `.csv`.
    * **Se for verdadeiro:** A automação prossegue para o salvamento do arquivo.
    * **Se for falso:** O fluxo é desviado para enviar um e-mail de erro ao remetente.

3.  **Carregar Arquivo (Google Drive):** O arquivo CSV é enviado para uma pasta pré-definida no Google Drive (https://drive.google.com/drive/folders/1Ockom5h1kf_Y_ikQFH64eeJSFHZYHqqT?usp=drive_link). 

4.  **Solicitação HTTP:** O workflow consulta a API pública `Open ER-API` para obter a cotação do dólar (USD) mais recente.

5.  **Editar Campos (Set):** Este nó prepara os dados para o e-mail final. Filtra a resposta da API, extraindo apenas as cotações de interesse (`BRL` e `USD`).

6.  **Enviar E-mail (Send Email):** Por fim, um e-mail de confirmação é enviado ao remetente. A mensagem inclui o nome do arquivo processado juntamente com o link de armazenamento e a cotação do dólar para real (USD/BRL).

## 3. Configuração do Ambiente

Para que o workflow funcione corretamente, é necessário configurar as seguintes credenciais e parâmetros.

### 3.1. Credenciais Necessárias

As seguintes credenciais devem ser criadas no painel `Credentials` do n8n:

* **Credencial IMAP (para ler e-mails):**
    * **Serviço:** Gmail ou outro provedor de e-mail.
    * **Autenticação:** Requer a criação de uma **Senha de App (App Password)** na conta Google, não a senha principal do usuário.
    * **Configuração:** Host (ex: `imap.gmail.com`), Porta (`993`), Usuário (e-mail completo) e a Senha de App.

* **Credencial SMTP (para enviar e-mails):**
    * **Serviço:** O mesmo provedor da credencial IMAP.
    * **Autenticação:** Utiliza a **mesma Senha de App** criada para a IMAP.
    * **Configuração:** Host (ex: `smtp.gmail.com`), Porta (`587`), Usuário (e-mail completo), Senha de App e a opção `SSL/TLS` **desativada** para a porta 587.

* **Credencial Google Drive OAuth2 (para salvar arquivos):**
    * **Autenticação:** Requer a configuração de um cliente OAuth2 no Google Cloud Console, seguindo o passo a passo para gerar um `Client ID` e um `Client Secret`. O app deve ser configurado com o usuário de e-mail sendo um "Usuário de teste".

### 3.2. Configuração dos Nós

* **Nó `Carregar Arquivo` (Google Drive):** O principal parâmetro a ser configurado manualmente é o ID da pasta de destino.
    * No campo `Parent`, o usuário deve inserir o **ID da pasta do Google Drive**. Este ID pode ser encontrado na URL ao abrir a pasta no navegador.

## 4. Observações Técnicas

* **Nome do Remetente:** O nó `Send Email` foi configurado no campo `From Email` para exibir o nome "Financeiro Onfly" em vez do endereço de e-mail, tornando a comunicação mais profissional.

* **Título de e-mail:** Para que o fluxo não envie e-mails de retorno para o restante da caixa, apenas os e-mails com o título de `Planilha financeira`  entrarão no fluxo, seja um e-mail com ou sem anexo.

## 5. Como Usar

Para acionar o workflow, basta enviar um e-mail com o assunto contendo `Planilha financeira` e um único anexo com a extensão `.csv` para a caixa de entrada configurada na credencial IMAP.

## 6. Entregáveis

* fluxo_financeiro_julialana.json (export do workflow)

* readme.md
