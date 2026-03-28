---
document_type: tsd
domain: portal_cidadao
derives_from: sdd/07_domain_portal_cidadao_sdd.md
file: tsd/07_domain_portal_cidadao_tsd.md
---

# TSD-07 — Implementação: Portal do Cidadão

## 1. Schema e Índices

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| `citizens` | `cpf_hash` | UNIQUE | Busca por CPF sem decrypt |
| `citizens` | `govbr_sub` | UNIQUE | Lookup por Gov.Br subject |
| `certificate_templates` | `tipo` | B-tree | Catálogo por tipo |
| `issued_certificates` | `verification_code` | UNIQUE | Verificação pública |
| `issued_certificates` | `citizen_id` | B-tree | Histórico do cidadão |
| `itbi_requests` | `citizen_id` | B-tree | Histórico do cidadão |
| `itbi_requests` | `property_id` | B-tree | ITBIs por imóvel |
| `protocols` | `citizen_id` | B-tree | Histórico do cidadão |
| `protocol_attachments` | `protocol_id` | B-tree (FK) | Anexos por protocolo |

**Migrations:** `070_create_citizens`, `071_create_certificate_templates`, `072_create_issued_certificates`, `073_create_itbi_requests`, `074_create_protocols`, `075_create_protocol_attachments`, `076_create_protocol_types`, `077_create_inspection_responses`.

---

## 2. Implementações

---

#### IMP-07-001 — Acesso Web ao Portal
**Derives:** DES-07-001

**Deploy:** React SPA (Vite build) servida via Nginx. Rota catch-all `/*` → `index.html` para SPA routing.
**Nginx config:** `location /api/ { proxy_pass http://api:3000; }` — reverse proxy para Express.
**Testes:** E2E: Portal carrega em < 5s em 4G simulado; responsivo em mobile 375px.

---

#### IMP-07-002 — Acesso Anônimo
**Derives:** DES-07-002

**API (pública, sem auth):**

| Método | Path | Descrição |
|--------|------|-----------|
| GET | `/api/v1/portal/properties/search?q=&field=` | Busca pública |
| GET | `/api/v1/portal/properties/:id/public` | Dados públicos do imóvel |
| GET | `/api/v1/portal/layers` | Camadas públicas |

**Response `GET /properties/:id/public`:**
```json
{
  "data": {
    "endereco": "Rua das Flores, 150",
    "bairro": "Centro",
    "area_terreno": 360.5,
    "tipo": "lote",
    "zoneamento": { "codigo": "ZR-2", "nome": "Zona Residencial 2" }
  }
}
```

> CPF, nome do proprietário e débitos **nunca** retornados neste endpoint.

**Testes:** Integration: GET público retorna dados sem PII; endpoint autenticado retorna 401 sem sessão.

---

#### IMP-07-003 — Acesso Autenticado
**Derives:** DES-07-003

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| GET | `/api/v1/portal/me` | Dados do cidadão logado | Cidadão |
| GET | `/api/v1/portal/me/properties` | Imóveis vinculados ao CPF | Cidadão |

**Fluxo de sessão:**
1. Após login OIDC (IMP-07-004), sessão Redis contém: `{ citizen_id, cpf_hash, nome }`.
2. `GET /me/properties` busca por `cpf_hash` na tabela `properties` (decrypt CPF → hash → compare).
3. Retorna dados completos (incluindo débitos) dos imóveis do cidadão.

**Testes:** Integration: cidadão logado vê seus imóveis; não vê imóveis de outros.

---

#### IMP-07-004 — Login via Gov.Br
**Derives:** DES-07-004

**API:**

| Método | Path | Descrição |
|--------|------|-----------|
| GET | `/api/v1/portal/auth/govbr/login` | Redirect para Gov.Br |
| GET | `/api/v1/portal/auth/govbr/callback` | Callback OIDC |
| GET | `/api/v1/portal/auth/logout` | Destruir sessão + logout OIDC |

**Fluxo OIDC (Authorization Code + PKCE):**
1. `/login` → gera `code_verifier`, `code_challenge`, `state`, `nonce` → redirect para Gov.Br.
2. Gov.Br autentica → callback com `code`.
3. `/callback` → troca code por tokens → valida `id_token` → extrai `sub`, `cpf`, `name`, nível.
4. Upsert `citizens` → cria sessão Redis → Set-Cookie.

**Config (em `tenant_config` no `geo_control`):**
```json
{
  "govbr_client_id": "...",
  "govbr_client_secret": "...",
  "govbr_redirect_uri": "https://{slug}.geo.app/api/v1/portal/auth/govbr/callback"
}
```

**Fallback:** Se Gov.Br indisponível (OIDC discovery falha), login local disponível.
**Testes:**
- Integration: mock OIDC provider → callback processa tokens → sessão criada.
- Integration: logout destrói sessão Redis.

---

#### IMP-07-005 — Captcha Cloudflare Turnstile
**Derives:** DES-07-005

**API:**
- Backend middleware `turnstileMiddleware` aplicado em: login local, protocolo, ITBI.
- Valida token via `POST https://challenges.cloudflare.com/turnstile/v0/siteverify`.
- Config: `tenant_config.turnstile_site_key` (público) + `turnstile_secret_key` (backend).

**Frontend:** Widget renderizado via `@cloudflare/turnstile`. `site_key` obtida de `GET /api/v1/portal/config`.
**Testes:** Integration: request sem token Turnstile → 403; com token válido (mock) → passa.

---

#### IMP-07-006 — Consulta de Imóveis pelo Cidadão
**Derives:** DES-07-006

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/portal/properties/search?q=&field=` | Público |
| GET | `/api/v1/portal/properties/:id/public` | Público |
| GET | `/api/v1/portal/properties/:id/full` | Cidadão (proprietário) |

**Controle de acesso `/full`:** Verifica `cpf_hash` do cidadão logado == hash do CPF do proprietário do imóvel. Se não → 403.

**Testes:**
- Integration: cidadão proprietário acessa `/full` → dados completos.
- Integration: cidadão não proprietário acessa `/full` → 403.
- Integration: anônimo acessa `/public` → dados sem PII.

---

#### IMP-07-007 — Emissão de Certidões Online
**Derives:** DES-07-007

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/portal/certificates/catalog` | Cidadão |
| POST | `/api/v1/portal/certificates/issue` | Cidadão |
| GET | `/api/v1/portal/certificates/:id/download` | Cidadão |
| GET | `/api/v1/portal/certificates/history` | Cidadão |
| GET | `/api/public/verify/:code` | Público |

**Request `POST /issue`:**
```json
{ "property_id": 42, "tipo": "debitos" }
```

**Erros:** 422 `NOT_ELIGIBLE` — `{ reason: "Débitos pendentes" }`. 404 `TEMPLATE_NOT_FOUND`.

**Seed `certificate_templates`:** confrontacao, debitos, avaliacao, uso_solo, negativa, localizacao.
**Testes:**
- Integration: emitir certidão → PDF no S3 com QR code → verificação pública funcional.
- Integration: cidadão não proprietário → 403.
- Integration: elegibilidade falha (débitos) → 422.

---

#### IMP-07-008 — Solicitação de ITBI
**Derives:** DES-07-008

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/portal/itbi` | Cidadão |
| GET | `/api/v1/portal/itbi/:id` | Cidadão |
| GET | `/api/v1/portal/itbi/:id/guia` | Cidadão |

**Request `POST /itbi`:**
```json
{
  "property_id": 42,
  "valor_declarado": 300000,
  "comprador": { "nome": "João", "cpf": "123.456.789-00" },
  "vendedor": { "nome": "Maria", "cpf": "987.654.321-00" }
}
```

**Response 201:**
```json
{
  "data": {
    "id": 1,
    "valor_declarado": 300000,
    "valor_venal": 250000,
    "base_calculo": 300000,
    "aliquota": 0.02,
    "valor_itbi": 6000,
    "isento": false,
    "memoria_calculo": { "regra": "max(declarado, venal)", "declarado": 300000, "venal": 250000 },
    "status": "pendente"
  }
}
```

**Testes:**
- Unit: cálculo com valor declarado > venal; cálculo com isenção.
- Integration: solicitar ITBI → guia PDF gerada → download funcional.

---

#### IMP-07-009 — Interação com Fiscalizações
**Derives:** DES-07-009

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/portal/inspections` | Cidadão |
| GET | `/api/v1/portal/inspections/:id` | Cidadão |
| POST | `/api/v1/portal/inspections/:id/responses` | Cidadão |

**Filtro:** Apenas fiscalizações de imóveis vinculados ao CPF do cidadão.
**Testes:** Integration: cidadão vê fiscalizações dos seus imóveis; resposta registrada; servidor notificado.

---

#### IMP-07-010 — Abertura de Protocolos
**Derives:** DES-07-010

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/portal/protocols` | Cidadão |
| GET | `/api/v1/portal/protocols` | Cidadão |
| GET | `/api/v1/portal/protocols/:id` | Cidadão |

**Request `POST /protocols`:** Multipart — `tipo_solicitacao`, `descricao`, `attachments[]`.
**Número de protocolo:** `YYYYMMDD-NNNN` (sequencial diário).
**Integração terceiro:** Se `tenant_config.protocol_integration` configurado → forward via API (ver TSD-08 IMP-08-007).
**Captcha:** Turnstile obrigatório neste endpoint.
**Testes:** Integration: abrir protocolo → número gerado; com integração terceira mock → `external_protocol_number` preenchido.

---

#### IMP-07-011 — Camadas Públicas e Impressão
**Derives:** DES-07-011

**API:**
- `GET /api/v1/portal/layers` — retorna apenas `visible_portal = true`.

**Deploy:** Impressão 100% frontend (leaflet-image + jsPDF).
**Testes:** E2E: toggle de camada pública funciona; exportação gera PDF com imóvel destacado.
