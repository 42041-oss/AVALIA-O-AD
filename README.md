# API de Upload e Download de Arquivos

API REST em **Python + Flask** para enviar (upload) e baixar (download) arquivos. Segue uma organização em camadas (rotas → controllers → models → banco) e registra os uploads em um arquivo de log.

🔗 **API em produção:** `https://SEU-SERVICO.onrender.com` *(substitua pelo link do Render)*

> Observação: no plano gratuito do Render o serviço "dorme" após alguns minutos sem uso; a primeira requisição pode levar cerca de 1 minuto para responder.

## Tecnologias

- Python 3
- Flask 3
- SQLite (módulo `sqlite3` da biblioteca padrão)
- Gunicorn (servidor de produção)

## Estrutura do projeto

```
.
├── app.py                      # Cria o app Flask e registra as rotas
├── controllers/
│   ├── upload_controller.py    # Validações e gravação do arquivo enviado
│   └── download_controller.py  # Envio do arquivo para download
├── models/                     # Acesso ao banco (SQLite)
├── routes/
│   ├── upload_route.py         # Rotas de upload
│   └── download_route.py       # Rotas de download
├── database/db.py              # Conexão e criação da tabela
├── requirements.txt            # Dependências
└── render.yaml                 # Configuração de deploy no Render
```

## Endpoints

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/` | Verifica se a API está no ar |
| POST | `/uploads/upload` | Envia um arquivo (`multipart/form-data`, campo `file`) |
| GET | `/downloads/download/<filename>` | Baixa um arquivo já enviado |

### Regras do upload

- Extensões permitidas: `png`, `jpg`, `jpeg`, `gif`, `pdf`, `txt`
- Tamanho máximo: **30 KB**
- O nome do arquivo é sanitizado com `secure_filename`
- Cada tentativa (sucesso ou falha) é registrada em `uploads.log`

### Exemplos (cURL)

**Upload**

```bash
curl -X POST -F "file=@exemplo.txt" https://SEU-SERVICO.onrender.com/uploads/upload
```

Resposta (200):

```json
{ "mensagem": "Arquivo salvo com sucesso!", "arquivo": "exemplo.txt" }
```

**Download**

```bash
curl -OJ https://SEU-SERVICO.onrender.com/downloads/download/exemplo.txt
```

### Respostas de erro

| Situação | Código | Corpo |
|----------|--------|-------|
| Nenhum arquivo enviado | 400 | `{"erro": "Nenhum arquivo enviado"}` |
| Nome de arquivo vazio | 400 | `{"erro": "Nome do arquivo vazio"}` |
| Arquivo maior que 30 KB | 400 | `{"erro": "Arquivo excede o tamanho máximo permitido (30 KB)"}` |
| Extensão não permitida | 400 | `{"erro": "Tipo de arquivo não permitido!", "tipos_permitidos": [...]}` |
| Arquivo não encontrado no download | 404 | página/JSON de "não encontrado" |

### Testando no Postman / Insomnia

1. Crie uma requisição **POST** para `/uploads/upload`.
2. Em *Body*, escolha **form-data**, adicione o campo `file` com tipo **File** e selecione o arquivo.
3. Envie. Depois faça um **GET** em `/downloads/download/<nome-do-arquivo>`.

## Rodando localmente

```bash
# 1. Clonar o repositório
git clone https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git
cd SEU-REPOSITORIO

# 2. Criar e ativar o ambiente virtual
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Instalar as dependências
pip install -r requirements.txt

# 4. Executar
python app.py
```

A API fica disponível em `http://localhost:5000`. Para ativar o modo debug localmente: `FLASK_DEBUG=1 python app.py`.

## Deploy no Render

O projeto já inclui `requirements.txt` e `render.yaml`. Passo a passo:

1. Envie o projeto para um repositório no GitHub.
2. No [Render](https://render.com), clique em **New +** → **Web Service** e conecte o repositório.
3. Configure:
   - **Runtime:** Python 3
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `gunicorn app:app`
   - **Instance type:** Free
4. Clique em **Create Web Service** e aguarde o deploy terminar.
5. Acesse o link `https://<nome>.onrender.com/` — deve retornar `{"status": "ok", ...}`.

Configurações específicas de produção: o servidor é o **Gunicorn** (não o servidor de desenvolvimento do Flask), o modo debug fica desligado por padrão e a porta vem da variável de ambiente `PORT` fornecida pelo Render.

## Limitações conhecidas

- No plano gratuito do Render o **disco é efêmero**: arquivos enviados (`uploads/`), o banco `database.db` e o `uploads.log` são apagados a cada novo deploy ou reinício do serviço. Para persistência real seria necessário um disco persistente (plano pago) ou armazenamento externo (ex.: S3).
- A rota de download não exige autenticação.
