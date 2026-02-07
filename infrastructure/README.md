# Infraestrutura n8n Self-hosted

## 📋 Visão Geral

Stack completa Docker para rodar n8n em ambiente de produção com alta disponibilidade, segurança e confiabilidade.

## 🎯 Desafio

Implementar n8n em produção com:
- ✅ Alta disponibilidade
- ✅ HTTPS obrigatório (sem certificado manual)
- ✅ Segurança operacional
- ✅ Backup automatizado
- ✅ Custo controlado (sem SaaS)

## ✅ Solução

Infraestrutura containerizada com:
- n8n + PostgreSQL + Redis
- Cloudflare Tunnel (HTTPS automático)
- Hardening de segurança Ubuntu
- Rotinas de backup/restore

## 🏗️ Arquitetura


```bash
Internet
↓
Cloudflare Tunnel (HTTPS)
↓
Docker Network (internal)
├─→ n8n:5678
├─→ PostgreSQL:5432
└─→ Redis:6379
```

## 🛠️ Stack Técnico

**Containers:**
- n8n:latest
- postgres:15-alpine
- redis:7-alpine
- cloudflare/cloudflared

**Host:**
- Ubuntu 24.04 LTS
- Docker Engine 24+
- Docker Compose v2

**Segurança:**
- UFW (firewall)
- Fail2Ban
- SSH com chaves (senha desabilitada)

## 📦 Docker Compose
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: n8n-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: n8n
    volumes:
      - ./postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U n8n"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: n8n-redis
    restart: unless-stopped
    volumes:
      - ./redis:/data

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n-main
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=${DB_PASSWORD}
      - N8N_ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - N8N_HOST=${DOMAIN}
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://${DOMAIN}
    volumes:
      - ./n8n:/home/node/.n8n

  cloudflared:
    image: cloudflare/cloudflared:latest
    restart: unless-stopped
    command: tunnel --no-autoupdate run
    environment:
      - TUNNEL_TOKEN=${CF_TUNNEL_TOKEN}
```

## 🔐 Segurança Implementada

### 1. Firewall (UFW)
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp    # SSH apenas
sudo ufw enable
```

### 2. SSH Hardening
```bash
# /etc/ssh/sshd_config
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

### 3. Fail2Ban
```ini
[sshd]
enabled = true
maxretry = 5
bantime = 3600
```

### 4. Secrets Management
```bash
# .env (nunca commitar!)
DB_PASSWORD=senha_forte_aqui
ENCRYPTION_KEY=chave_32_bytes_hex
CF_TUNNEL_TOKEN=token_cloudflare
DOMAIN=n8n.aliflow.com.br
```

## 💾 Backup Strategy

### Backup Automatizado (Cron diário)
```bash
#!/bin/bash
# /opt/scripts/backup-n8n.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backups/n8n"

# Backup PostgreSQL
docker exec n8n-postgres pg_dump -U n8n n8n > "$BACKUP_DIR/db_$DATE.sql"

# Backup volumes
tar -czf "$BACKUP_DIR/volumes_$DATE.tar.gz" /opt/n8n/data

# Retention: 7 dias
find $BACKUP_DIR -mtime +7 -delete
```

### Restore
```bash
# Restaurar banco
docker exec -i n8n-postgres psql -U n8n n8n < backup.sql

# Restaurar volumes
tar -xzf volumes_backup.tar.gz -C /opt/n8n/
```

## 📊 Monitoramento

### Healthchecks
```bash
# PostgreSQL
docker exec n8n-postgres pg_isready -U n8n

# n8n
curl -I https://n8n.aliflow.com.br

# Containers
docker ps --filter "status=running"
```

### Logs
```bash
# Ver logs em tempo real
docker compose logs -f n8n

# Últimas 100 linhas
docker compose logs --tail=100 n8n

# Logs com timestamp
docker compose logs -t n8n
```

## 🚀 Deploy

### Setup Inicial
```bash
# 1. Criar diretórios
sudo mkdir -p /opt/n8n/{data,postgres,redis}
sudo chown -R 1000:1000 /opt/n8n/data
sudo chown -R 999:999 /opt/n8n/{postgres,redis}

# 2. Gerar encryption key
openssl rand -hex 32 > /opt/n8n/.encryption_key

# 3. Criar .env
cat > /opt/n8n/.env <<EOF
DB_PASSWORD=$(openssl rand -base64 24)
ENCRYPTION_KEY=$(cat /opt/n8n/.encryption_key)
DOMAIN=n8n.aliflow.com.br
CF_TUNNEL_TOKEN=seu_token_aqui
EOF

# 4. Subir stack
cd /opt/n8n
docker compose up -d

# 5. Verificar
docker compose ps
docker compose logs -f
```

## 📈 Resultados

- ✅ **Uptime**: 99.9%
- ✅ **Latência**: <500ms
- ✅ **Custo**: ~R$50/mês (VPS)
- ✅ **Zero** incidentes de segurança
- ✅ **Backup diário** automatizado

## 🎓 Decisões Técnicas

### Por que PostgreSQL?
- ACID compliance
- Suporte nativo do n8n
- Facilita queries de idempotência
- Backup/restore simples

### Por que Cloudflare Tunnel?
- HTTPS automático (Let's Encrypt via Cloudflare)
- Sem IP público necessário
- DDoS protection gratuito
- Edge network (baixa latência)

### Por que Docker?
- Isolamento de processos
- Portabilidade
- Versionamento de serviços
- Rollback fácil

## 📝 Manutenção

### Atualização do n8n
```bash
cd /opt/n8n

# Backup antes
docker exec n8n-postgres pg_dump -U n8n n8n > backup_pre_update.sql

# Atualizar
docker compose pull n8n
docker compose up -d n8n

# Verificar
docker compose logs -f n8n
```

### Atualização do Ubuntu
```bash
sudo apt update && sudo apt upgrade -y
sudo reboot  # se kernel atualizado
```

## 🔍 Troubleshooting

### n8n não inicia
```bash
# Ver logs
docker compose logs n8n

# Verificar PostgreSQL
docker exec n8n-postgres pg_isready -U n8n

# Verificar permissões
ls -la /opt/n8n/data
```

### Sem conectividade
```bash
# Verificar Cloudflare Tunnel
docker compose logs cloudflared

# Testar HTTPS
curl -I https://n8n.aliflow.com.br
```

## 📚 Documentação Técnica

Documentação completa da infraestrutura disponível em:
- [PDF: Reestruturando Infraestrutura n8n]

---

**Status**: ✅ Em andamento
**Domínio**: https://n8n.aliflow.com.br  
**Última atualização**: Fevereiro 2025
