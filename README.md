# 🚀 Portfolio de Automação de Processos

Bem-vindo ao meu portfolio técnico! Aqui você encontra documentação dos meus projetos de automação com **n8n**, **Docker**, **PostgreSQL** e **API integrations**.

## 👨‍💻 Sobre Mim

Sou Automation Specialist com foco em:
- Automação de processos com n8n
- Integrações via APIs REST e Webhooks
- Infraestrutura self-hosted com Docker
- Persistência e idempotência com PostgreSQL

## 🛠️ Stack Técnico

**Automação & Integrações:**
- n8n (workflows complexos)
- APIs REST (GET, POST, PUT, DELETE)
- Webhooks
- OAuth & API Authentication
- JSON/XML parsing

**Infraestrutura:**
- Docker & Docker Compose
- PostgreSQL
- Redis
- Cloudflare Tunnel
- Linux (Ubuntu)

**Práticas:**
- Idempotência
- Error handling & retry logic
- Logging estruturado
- Backup/restore
- Hardening de segurança

## 📁 Projetos

### 1️⃣ Automação de Cobrança (Asaas + Omie)
**Status:** Em produção

Sistema de automação para processos financeiros com:
- Recepção de webhooks do Asaas
- Validação de payload
- Idempotência via PostgreSQL
- Sincronização com Omie
- Tratamento de erros com retry
- Respostas HTTP padronizadas

**Stack:** n8n, PostgreSQL, APIs REST, Webhooks

📂 [Ver documentação](./workflows/cobranca-asaas-omie/) *(em breve)*

---

### 2️⃣ Infraestrutura n8n Self-hosted
**Status:** Em produção

Stack completa Docker para n8n em ambiente de produção:
- n8n + PostgreSQL + Redis
- HTTPS via Cloudflare Tunnel
- Hardening de segurança (UFW, Fail2Ban, SSH)
- Backup automatizado
- Monitoramento e logs

**Stack:** Docker, PostgreSQL, Redis, Cloudflare, Ubuntu

📂 [Ver documentação](./infrastructure/) *(em breve)*

---

### 3️⃣ Workflow Financeiro Avançado
**Status:** Em estudo

Análise técnica de workflow complexo para processos financeiros.
Estudo nó a nó para compreensão de arquitetura de automação.

**Stack:** n8n, API integrations

📂 [Ver documentação](./workflows/financeiro-avancado/) *(em breve)*

---

## 📊 Destaques Técnicos

### Idempotência
Implementação de controle de duplicidade usando PostgreSQL como fonte de verdade:
- Busca prévia por ID único antes de processar
- Registro de estado com timestamp
- Validação de processamento anterior

### Tratamento de Erro
Estratégia de retry com backoff exponencial:
- 3 tentativas automáticas
- Logging estruturado de erros
- Notificação apenas em falha crítica

### Respostas HTTP Padronizadas
- `200` - Processado com sucesso
- `401` - Autenticação inválida
- `422` - Payload inválido ou duplicado

## 🌐 Links

* 💼 LinkedIn: [linkedin.com/in/alisson-araujo-aliflow22](https://www.linkedin.com/in/alisson-araujo-aliflow22)
* 🌍 Portfólio: [aliflow.com.br](https://aliflow.com.br)
* 📧 Email: [APAS22@proton.me](mailto:APAS22@proton.me)[APAS22@proton.me](mailto:APAS22@proton.me)

## 📝 Status

Este portfolio está em **construção ativa**. Novos projetos e documentações serão adicionados regularmente.

---

💡 **Interessado em automação de processos?** Sinta-se à vontade para explorar os projetos e entrar em contato!
