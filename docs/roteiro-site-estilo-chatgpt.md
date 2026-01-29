## Objetivo didático (o “site estilo ChatGPT”)
Construir um **chat web** onde o aluno:
- faz login;
- cria **conversas**;
- envia **mensagens**;
- o servidor chama um **LLM** (API externa ou modelo local) e retorna a resposta;
- opcionalmente com **streaming** (resposta “digitando”).

## Próximos passos (ordem recomendada para ensino)
- **1) Fechar o “mínimo rodável” com Docker**
  - Garantir `docker-compose.yml` com serviços: **web (Django)** + **db (MySQL)**.
  - Padronizar variáveis no `.env` (sem sobrescrever; apenas documentar).
  - Confirmar migrações e criação de superusuário via `entrypoint.sh` (se aplicável).

- **2) Modelagem no MySQL (dados do chat)**
  - Criar app Django (ex: `chat/`) e models:
    - `Conversation` (título, usuário, timestamps)
    - `Message` (conversation, role: user/assistant/system, content, timestamps)
    - opcional: `LLMCall`/`Usage` (tokens, custo, latência) para ensinar observabilidade.

- **3) UI simples (sem reinventar front no começo)**
  - Começar com **templates Django** + Bootstrap via CDN (rápido para curso técnico).
  - Páginas:
    - lista de conversas
    - tela de conversa (mensagens + input)
    - criar nova conversa

- **4) Endpoint de chat (primeiro sem streaming)**
  - POST “enviar mensagem”: salva `Message(user)`, chama serviço do LLM, salva `Message(assistant)`, retorna JSON.
  - Separar em camadas: `views` → `services/llm.py` → (opcional) `repositories` para manter legível.

- **5) Streaming (para ficar “igual ChatGPT”)**
  - Implementar **SSE** (Server-Sent Events) como caminho mais simples no Django.
  - Front recebe “chunks” e vai atualizando a última mensagem.

- **6) Autenticação e permissões**
  - Usar auth padrão do Django (login/logout) e garantir que cada usuário só vê suas conversas.

- **7) Segurança e ambientes (dev/test/prod)**
  - Ajustar `settings.py` para separar configs (ex: `DEBUG`, `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS`, credenciais do MySQL).
  - Segredos só no `.env`, sem commitar.

- **8) Qualidade e evolução**
  - Testes de models/serviços (pelo menos 2–3 testes simples).
  - Logs e métricas básicas (tempo de resposta do LLM, erros).
  - Paginação de mensagens, limite de tamanho, rate limit simples.

## Sugestão de trilha em aulas (para manter ritmo)
- **Aula 1–2**: Docker + Django rodando + MySQL + admin.
- **Aula 3**: Models `Conversation/Message` + telas HTML.
- **Aula 4**: Endpoint de chat + integração LLM (sem streaming).
- **Aula 5**: Streaming SSE + UX “digitando”.
- **Aula 6**: auth/permissões + hardening + testes.

## Reflexão (escalabilidade e manutenibilidade)
Para turma técnica, a maior fonte de dívida costuma ser misturar tudo em `views.py` (regra de negócio + chamada ao LLM + persistência). Separar cedo em **camadas pequenas** (view fina, `services/llm.py`, `services/chat.py`, models claros) mantém o projeto explicável e facilita trocar o provedor de LLM no futuro sem reescrever a aplicação.

Em termos de escala, o ponto crítico é **streaming e concorrência** (muitas conexões abertas) e **latência do LLM**. Começar com SSE e depois discutir alternativas (cache, filas assíncronas, workers) vira um excelente gancho pedagógico sem complicar o MVP.

