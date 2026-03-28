---
document_type: tsd
domain: plataforma
derives_from: sdd/01_platform_sdd.md
file: tsd/01_platform_tsd.md
---

# TSD-01 — Deploy e Implantação da Plataforma

## 1. Stages de Deploy

### 1.1 Ambientes

| Ambiente | Infra | Dados | Acesso |
|----------|-------|-------|--------|
| `development` | Docker Compose local | Seed fictício (1 tenant demo) | Desenvolvedor local |
| `staging` | Kubernetes (cluster compartilhado) | Anonimizados de produção | Time QA + PO |
| `production` | Kubernetes (cluster dedicado) | Dados reais dos municípios | Usuários finais |

### 1.2 Critérios de Promoção

| Transição | Critérios obrigatórios |
|-----------|----------------------|
| dev → staging | Build passa; testes unit ≥ 80% coverage; lint sem erros; migrations aplicam sem erro |
| staging → production | QA aprova; testes E2E passam; nenhum bug crítico/blocker aberto; rollback testado |

### 1.3 Proteções

- **Production:** deploy apenas via pipeline CI/CD (nunca manual).
- **Staging:** pode receber deploys automáticos de `main`.
- **Rollback:** revert automático se health check falhar nos primeiros 5min pós-deploy.

---

## 2. Estratégias e Diretrizes de Deploy

### 2.1 Cloud e Hospedagem

| Aspecto | Decisão | Justificativa |
|---------|---------|---------------|
| Cloud | Cloud-agnostic (AWS, GCP, Azure ou on-premise) | Municípios podem exigir hospedagem nacional ou on-premise |
| Compute | Kubernetes (EKS/GKE/AKS ou bare-metal K8s) | Padronizado, portável |
| Database | PostgreSQL managed (RDS/CloudSQL) ou self-hosted | Managed preferível em cloud; self-hosted para on-premise |
| Cache | Redis managed (ElastiCache/Memorystore) ou self-hosted | Idem |
| Storage | S3 (AWS) / GCS / MinIO (on-premise) | MinIO para cenários self-hosted |
| DNS | Wildcard DNS `*.geo.app` | Subdomínios por tenant |

### 2.2 Estratégia de Deploy

- **Tipo:** Rolling update (Kubernetes default).
- **Replicas mínimas:** 2 (zero-downtime durante deploy).
- **Health checks:** `/api/health` (liveness) e `/api/ready` (readiness).
- **Readiness gate:** app responde 200 em `/api/ready` somente após conectar ao PostgreSQL de controle e Redis.

### 2.3 Serviços SaaS/PaaS Permitidos

| Serviço | Uso | Alternativa self-hosted |
|---------|-----|------------------------|
| Cloudflare Turnstile | Captcha | Nenhuma (SaaS obrigatório) |
| Gov.Br (OIDC) | Autenticação cidadão | Nenhuma (Gov Federal) |
| Google Street View API | Imagens de rua | Nenhuma (Google) |
| Provedor de imagens satélite | Tiles de satélite | Importação de arquivos (GeoTIFF) |

---

## 3. Componentes de Arquitetura

### 3.1 Permitidos

| Componente | Justificativa |
|-----------|---------------|
| PostgreSQL + PostGIS | Banco principal — geoespacial nativo |
| Redis | Sessões, cache de permissões, filas BullMQ |
| S3-compatible storage | Arquivos binários (anexos, PDFs, imagens) |
| BullMQ (sobre Redis) | Processamento assíncrono de jobs |
| Nginx | Reverse proxy, serving de SPA estática, TLS termination |
| Puppeteer (Chromium headless) | Geração de PDF — roda como sidecar ou no mesmo container |
| GDAL | Conversão de datum e formatos geoespaciais — via `child_process` |

### 3.2 Negados

| Componente | Justificativa |
|-----------|---------------|
| MongoDB / DynamoDB | Dados são relacionais e geoespaciais; PostgreSQL cobre tudo |
| Elasticsearch | Overkill para v1; Full-Text Search do PostgreSQL suficiente |
| Kafka / RabbitMQ | BullMQ sobre Redis suficiente para volume atual |
| GraphQL | API REST com OpenAPI é mais simples e auditável para este contexto |
| Lambda / Cloud Functions | Monólito — sem serverless |
| Terraform / Pulumi | Fora do escopo v1; infra manual ou scripts Docker Compose/K8s |

---

## 4. Especificações de Implementação da Plataforma

### 4.1 Autenticação e Acesso

---

#### IMP-01-001 — Deploy do Sistema RBAC
**Derives:** DES-01-001

**Migrations:** `001_create_roles`, `002_create_permissions`, `003_create_role_permissions`, `004_create_user_roles`.
**Seed:** Roles padrão: `super_admin`, `admin`, `servidor_cadastro`, `servidor_urbanismo`, `servidor_ambiental`, `fiscal`, `cidadao`.
**Índices:** `user_roles(user_id)`, `role_permissions(role_id)`.
**Testes:** Unit: `requirePermission` middleware bloqueia sem role. Integration: endpoint protegido retorna 403 sem permissão.

---

#### IMP-01-002 — Deploy da Gestão de Termo de Uso
**Derives:** DES-01-002

**Migrations:** `005_create_terms_of_use`, `006_create_term_acceptances`.
**Seed:** Termo de Uso v1 (texto padrão) para tenant demo.
**Testes:** Integration: usuário sem aceite recebe 451; após aceite, acesso liberado.

---

#### IMP-01-003 — Deploy do Banner de Privacidade
**Derives:** DES-01-003

**Deploy:** 100% frontend — sem migração, sem env var. Componente React puro.
**Testes:** E2E: banner aparece no primeiro acesso; desaparece após fechar; não reaparece.

---

#### IMP-01-004 — Deploy da Política de Privacidade
**Derives:** DES-01-004

**Migrations:** `007_create_tenant_content`.
**Seed:** Política de Privacidade template padrão.
**Testes:** Integration: `GET /api/public/privacy-policy` retorna 200 sem auth.

---

#### IMP-01-005 — Deploy do Portal LGPD
**Derives:** DES-01-005

**Migrations:** `008_create_lgpd_requests`.
**Env vars:** — (usa `ENCRYPTION_KEY` global para dados exportados).
**Testes:** Integration: cidadão cria request; admin visualiza fila; job de portabilidade gera ZIP.

---

#### IMP-01-006 — Deploy do Registro de Tratamento
**Derives:** DES-01-006

**Migrations:** `009_create_data_processing_records`.
**Testes:** Unit: helper `trackDataProcessing` insere registro. Integration: endpoint de export CSV funcional.

---

#### IMP-01-007 — Deploy da Interface de Auditoria
**Derives:** DES-01-007

**Testes:** Integration: `GET /api/admin/audit-logs` retorna logs paginados; filtros por data/user/action funcionam.

---

#### IMP-01-008 — Deploy da Exportação Integral
**Derives:** DES-01-008

**Env vars:** `S3_BUCKET_EXPORTS`, `EXPORT_LINK_TTL_HOURS=24`.
**Dependências:** `pg_dump` e `ogr2ogr` disponíveis no container.
**Testes:** Integration: job `tenant-full-export` gera ZIP no S3; link funcional.

---

### 4.2 Segurança e Privacidade

---

#### IMP-01-009 — Minimização de Coleta
**Derives:** DES-01-009

**Deploy:** Arquivo `data-dictionary.yml` no repositório. CI valida que novos campos PII possuem entry.
**Testes:** Unit: repositories usam `select` explícito (não `select *`).

---

#### IMP-01-010 — Dados Sensíveis
**Derives:** DES-01-010

**Env vars:** `ENCRYPTION_KEY` (32 bytes, base64).
**Migrations:** Extensão `pgcrypto` habilitada no database template.
**Testes:** Unit: CPF criptografado no INSERT; descriptografado no SELECT com permissão.

---

#### IMP-01-011 — Anonimização em Indicadores
**Derives:** DES-01-011

**Migrations:** Views materializadas para indicadores criadas em migration dedicada.
**Testes:** Unit: queries de indicadores não retornam PII.

---

#### IMP-01-012 — Mascaramento em Exportações
**Derives:** DES-01-012

**Migrations:** `010_create_masking_rules`. Seed com regras padrão.
**Testes:** Integration: export CSV como `servidor` mascara CPF; como `admin` exibe completo.

---

#### IMP-01-013 — Pseudonimização em Logs
**Derives:** DES-01-013

**Testes:** Unit: audit logs contêm `user_id` numérico, nunca CPF/nome em texto.

---

### 4.3 Propriedade Intelectual

---

#### IMP-01-014 — Licenças Cartográficas
**Derives:** DES-01-014
**Deploy:** Componente frontend `MapAttribution` — sem infra. **Testes:** E2E: atribuição visível no mapa.

---

#### IMP-01-015 — Titularidade de Documentos
**Derives:** DES-01-015
**Deploy:** Templates Handlebars incluem variáveis de município. **Testes:** Unit: PDF gerado contém rodapé com nome do município.

---

#### IMP-01-016 — Aviso Legal
**Derives:** DES-01-016
**Deploy:** Texto estático em Termos e Política. **Testes:** E2E: aviso visível nos documentos.

---

#### IMP-01-017 — Proteção Download em Massa
**Derives:** DES-01-017
**Env vars:** `RATE_LIMIT_TILES_PER_MIN=100`.
**Testes:** Integration: requisição 101 em 1min retorna 429.

---

### 4.4 Infraestrutura

---

#### IMP-01-018 — Deploy Multi-tenant (Database-per-tenant)
**Derives:** DES-01-018

**Banco de controle (`geo_control`):**

```sql
CREATE TABLE tenants (
  id SERIAL PRIMARY KEY,
  slug VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(200) NOT NULL,
  db_name VARCHAR(100) UNIQUE NOT NULL,
  db_host VARCHAR(200) NOT NULL DEFAULT 'localhost',
  status VARCHAR(20) NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE tenant_config (
  tenant_id INT REFERENCES tenants(id),
  config JSONB NOT NULL DEFAULT '{}',
  PRIMARY KEY (tenant_id)
);
```

**Database template (`geo_template`):** Schema Prisma completo + PostGIS + extensão pgcrypto + migrations aplicadas + sem dados.

**Provisioning de novo tenant:**
```bash
# 1. Criar database a partir do template
CREATE DATABASE geo_${slug} TEMPLATE geo_template;
# 2. Registrar em geo_control
INSERT INTO tenants (slug, name, db_name) VALUES ('${slug}', '${name}', 'geo_${slug}');
# 3. Aplicar migrations pendentes
npx prisma migrate deploy --schema=./prisma/schema.prisma
# 4. Seed de dados padrão (roles, permissions)
npx prisma db seed
```

**Env vars:** `CONTROL_DATABASE_URL=postgres://...geo_control`.
**Testes:** Integration: request para `tenant-a.geo.app` acessa `geo_tenant_a`; dados isolados entre tenants.

---

#### IMP-01-019 — Deploy da Trilha de Auditoria
**Derives:** DES-01-019

**Migration:** `011_create_audit_logs` com particionamento por mês.
**DB role:** `geo_app` tem apenas `INSERT` em `audit_logs`.
**Testes:** Integration: tentativa de UPDATE/DELETE em `audit_logs` retorna erro de permissão.

---

#### IMP-01-020 — Deploy de Backup e Recuperação
**Derives:** DES-01-020

**Infra:**
- CronJob K8s: `pg_dump` diário de cada tenant DB → S3 (`/backups/{tenant_slug}/{date}.sql.gz`).
- WAL archiving: `archive_command` envia para S3.
- Runbook de restore em `docs/runbooks/restore-tenant.md`.
**Env vars:** `BACKUP_S3_BUCKET`, `BACKUP_RETENTION_DAYS=90`.
**Testes:** Trimestral: restore de backup em ambiente isolado, verificar integridade.
