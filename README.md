# Desafio MBA Engenharia de Software com IA - Full Cycle

Sistema de **Retrieval-Augmented Generation (RAG)** que ingere um documento PDF, armazena os embeddings em PostgreSQL com pgVector, e permite fazer perguntas ao conteúdo via CLI utilizando um LLM (Google Gemini ou OpenAI).

## Arquitetura

```
document.pdf → ingest.py (chunks + embeddings) → PostgreSQL/pgVector
                                                        ↓
                        chat.py (CLI) → search.py (busca semântica + LLM) → resposta
```

## Pré-requisitos

- **Python 3.10+**
- **Docker** e **Docker Compose**
- Chave de API de um dos provedores:
  - [Google AI Studio](https://aistudio.google.com/) (Gemini) — ou
  - [OpenAI](https://platform.openai.com/)

## Como executar

### 1. Clonar o repositório

```bash
git clone <url-do-repositorio>
cd mba-ia-desafio-ingestao-busca
```

### 2. Subir o banco de dados PostgreSQL com pgVector

```bash
docker compose up -d
```

Isso inicia o PostgreSQL com a extensão `pgVector` já habilitada automaticamente.

### 3. Criar e ativar o ambiente virtual Python

```bash
python -m venv venv
source venv/bin/activate
```

### 4. Instalar as dependências

```bash
pip install -r requirements.txt
```

### 5. Configurar as variáveis de ambiente

Copie o arquivo de exemplo e preencha com suas credenciais:

```bash
cp .env.example .env
```

Edite o `.env` com os valores adequados:

| Variável | Descrição | Exemplo |
|---|---|---|
| `AI_PROVIDER` | Provedor de IA: `google` ou `openai` | `google` |
| `GOOGLE_API_KEY` | Chave de API do Google AI Studio | `AIza...` |
| `GOOGLE_EMBEDDING_MODEL` | Modelo de embedding do Google | `models/gemini-embedding-001` |
| `OPENAI_API_KEY` | Chave de API da OpenAI | `sk-...` |
| `OPENAI_EMBEDDING_MODEL` | Modelo de embedding da OpenAI | `text-embedding-3-small` |
| `LLM_MODEL` | Modelo do LLM para gerar respostas | `gemini-2.5-flash-lite` ou `gpt-4o-mini` |
| `DATABASE_URL` | String de conexão do PostgreSQL | `postgresql+psycopg://postgres:postgres@localhost:5432/rag` |
| `PG_VECTOR_COLLECTION_NAME` | Nome da coleção de vetores | `rag_collection` |
| `PDF_PATH` | Caminho para o arquivo PDF | `document.pdf` |

> **Nota:** Você só precisa configurar as variáveis do provedor escolhido (`GOOGLE_*` ou `OPENAI_*`).

### 6. Executar a ingestão do PDF

Este passo lê o PDF, divide em chunks, gera os embeddings e armazena no banco de dados:

```bash
python src/ingest.py
```

Saída esperada:

```
Carregando PDF: document.pdf
Total de páginas carregadas: X
Total de chunks gerados: Y
Ingestão concluída com sucesso!
```

### 7. Iniciar o chat

Após a ingestão, inicie o chat interativo:

```bash
python src/chat.py
```

Faça suas perguntas no terminal. O sistema busca os trechos mais relevantes do documento e gera uma resposta contextualizada. Digite `sair` para encerrar.

```
Faça sua pergunta (digite 'sair' para encerrar):

PERGUNTA: Qual o objetivo do documento?
RESPOSTA: ...
```

> **Importante:** O sistema só responde com base no conteúdo do PDF. Perguntas fora do contexto recebem: *"Não tenho informações necessárias para responder sua pergunta."*

## Estrutura do projeto

```
├── docker-compose.yml      # PostgreSQL + pgVector
├── document.pdf             # PDF para ingestão
├── requirements.txt         # Dependências Python
├── .env.example             # Template das variáveis de ambiente
└── src/
    ├── ingest.py            # Ingestão: PDF → chunks → embeddings → pgVector
    ├── search.py            # Busca semântica + cadeia RAG com LLM
    └── chat.py              # Interface CLI interativa
```