# TaskFlowCards: versão para servidor

Esta pasta contém o aplicativo Flask, o frontend, o esquema PostgreSQL e o `render.yaml` para implantação no Render. A entrada de produção é `backend.render_app:app`, que exige `DATABASE_URL`. Não há inicializadores `.bat` ou `.ps1`, credenciais, configuração MySQL, banco local ou backup nesta pasta.

## Publicar

1. Envie **esta pasta** para um repositório na conta GitHub pessoal escolhida por você. Confira os arquivos antes de publicar. Nenhum envio ao GitHub é feito por este pacote.
2. No Render, crie um Blueprint a partir desse repositório e do arquivo `render.yaml`. Ele define um serviço web Python e um PostgreSQL no plano `free`.
3. Preencha no painel do Render as variáveis marcadas `sync: false`: `TASKFLOW_DATA_KEY`, `TASKFLOW_AUDIT_SEED`, `GEMINI_API_KEY`, `TASKFLOW_GMAIL_CLIENT_ID`, `TASKFLOW_GMAIL_CLIENT_SECRET`, `TASKFLOW_GMAIL_REFRESH_TOKEN` e `TASKFLOW_GMAIL_SENDER`. O Render gera `FLASK_SECRET_KEY` e liga `DATABASE_URL` ao banco. Não há código privado para cadastro de professor: professor e aluno confirmam o próprio Gmail por código numérico.
4. Aguarde o deploy e abra a URL `https://...onrender.com`. Teste cadastro de professor e aluno, confirmação por e-mail, recuperação de senha, bot, salas, dashboard e Gemini.

`TASKFLOW_DATA_KEY` é uma chave Fernet. Gere uma chave nova para o banco novo: `python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"`. Guarde-a fora do repositório. O banco PostgreSQL começa vazio; esta cópia não migra automaticamente os dados locais.

`TASKFLOW_AUDIT_SEED` precisa ser o Base64 de exatamente 32 bytes. Gere com `python -c "import base64,secrets; print(base64.b64encode(secrets.token_bytes(32)).decode())"`. Mantenha a mesma semente nos próximos deploys para verificar o histórico de auditoria.

O remetente Gmail usa as quatro variáveis `TASKFLOW_GMAIL_*`. A URL de retorno do fluxo de autorização é diferente para cada domínio; se precisar renovar o token, autorize a URL do servidor no Google Cloud. O token de atualização não deve ir para o código.

O aluno acessa a partida pela sala vinculada ao seu ano, turma e período, sem digitar código. O identificador interno da sala permanece no servidor para endereçar a partida e as operações de edição. A dashboard do professor oferece bimestres de calendário (janeiro–fevereiro, março–abril, e assim por diante); os acertos, erros, gráficos e a tabela respeitam o mesmo período. A narração usa o áudio retornado pela API REST de Interactions do Gemini.

Use as credenciais já existentes do Gemini e do Gmail; esta atualização não troca nenhuma delas. Professor e aluno se cadastram com um código de seis dígitos enviado ao próprio Gmail. O professor vê apenas alunos e resultados vinculados às suas salas, seus eventos de log e suas questões. Em **Minha conta**, o aluno pode excluir a própria conta mediante senha e confirmação escrita; a ação retira cadastro e vínculos pessoais do banco e encerra a sessão.

Se o serviço Render já foi criado com `TASKFLOW_SETUP_CODE`, remova essa variável antiga em **Environment** após atualizar o código. Ela não é mais lida. `TASKFLOW_DATA_KEY` e `TASKFLOW_AUDIT_SEED` continuam necessárias; mantenha os valores estáveis após criar dados.

## Arquivos de implantação

- `render.yaml`: serviço web, PostgreSQL e variáveis.
- `requirements.txt`: bibliotecas de produção.
- `backend/render_app.py`: entrada que exige PostgreSQL.
- `backend/`, `frontend/`, `banco_de_dados/`: aplicativo e esquema.

Esta cópia foi preparada para hospedagem e ainda precisa ser testada após o deploy real. Use somente dados de demonstração até validar o serviço publicado.
