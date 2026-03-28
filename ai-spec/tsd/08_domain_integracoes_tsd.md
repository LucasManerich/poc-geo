---
document_type: tsd
domain: integracoes
derives_from: sdd/08_domain_integracoes_sdd.md
file: tsd/08_domain_integracoes_tsd.md
---

# TSD-08 — Implementação: Integrações

## 1. Schema e Índices

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| `integration_config` | `provider` | UNIQUE | Uma config por provedor |
| `integration_status` | `provider` | UNIQUE | Um status por provedor |
| `integration_logs` | `provider, created_at` | B-tree | Consulta por provedor + período |
| `integration_logs` | `status` | B-tree | Filtro por sucesso/erro |
| `sync_queue` | `status, created_at` | B-tree | Processamento FIFO |
| `sync_queue` | `provider, entity_type, entity_id` | B-tree | Dedup de jobs |
| `data_conflicts` | `resolution, created_at` | B-tree | Fila de pendentes |

**Migrations:** `080_create_integration_config`, `081_create_integration_status`, `082_create_integration_logs`, `083_create_sync_queue`, `084_create_data_conflicts`, `085_create_conflict_rules`.

---

## 2. Implementações

---

#### IMP-08-001 — Integração Bidirecional com ERP
**Derives:** DES-08-001

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| GET | `/api/v1/admin/integracoes/erp/config` | Config do adaptador | Admin |
| PUT | `/api/v1/admin/integracoes/erp/config` | Atualizar config | Admin |
| POST | `/api/v1/admin/integracoes/erp/test` | Testar conexão | Admin |
| POST | `/api/v1/integracoes/erp/webhook` | Receber dados do ERP | Webhook auth |
| GET | `/api/v1/admin/integracoes/erp/logs?page=&limit=` | Logs de integração | Admin |

**Request `PUT /erp/config`:**
```json
{
  "adapter_type": "erp_vendor_a",
  "base_url": "https://erp.municipio.gov.br/api",
  "credentials": { "api_key": "xxx", "secret": "yyy" },
  "field_mapping": {
    "property.inscricao": "imovel.codigo",
    "property.area_terreno": "imovel.area",
    "person.cpf": "contribuinte.documento"
  }
}
```

**Webhook auth:** Header `X-Webhook-Secret` validado contra `integration_config.credentials.webhook_secret`.

**Outbound (GEO → ERP):** EventEmitter `propertyCreated` / `propertyUpdated` → BullMQ `sync-erp` → adapter traduz + envia.
**Inbound (ERP → GEO):** Webhook POST → adapter traduz → service atualiza.

**Testes:**
- Unit: adapter mapeia campos conforme `field_mapping`.
- Integration: mock ERP → outbound sync envia payload correto; webhook inbound atualiza dados.
- Integration: `POST /test` verifica conectividade.

---

#### IMP-08-002 — Resolução de Conflitos GEO-ERP
**Derives:** DES-08-002

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/admin/integracoes/conflicts?status=pending&page=&limit=` | Admin |
| GET | `/api/v1/admin/integracoes/conflicts/:id` | Admin |
| PUT | `/api/v1/admin/integracoes/conflicts/:id/resolve` | Admin |

**Request `PUT /resolve`:**
```json
{ "resolution": "keep_geo" }
// ou
{ "resolution": "manual", "manual_value": { "proprietario_nome": "João Corrigido" } }
```

**Response `GET /conflicts/:id`:**
```json
{
  "data": {
    "id": 5,
    "provider": "erp",
    "entity_type": "property",
    "entity_id": 42,
    "geo_value": { "proprietario_nome": "João Silva" },
    "external_value": { "proprietario_nome": "João S. Silva" },
    "resolution": "pending",
    "created_at": "2026-03-28T10:00:00Z"
  }
}
```

**Auto-resolução:** Tabela `conflict_rules` com regras ex.: `{ entity_type: 'property', field: 'proprietario', rule: 'keep_external' }`.
**Testes:**
- Integration: sync com dados divergentes → conflito criado; resolução `keep_geo` aplica valor local.
- Unit: regra automática resolve sem intervenção.

---

#### IMP-08-003 — Modo Degradado de Integração
**Derives:** DES-08-003

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/admin/integracoes/status` | Admin |
| GET | `/api/v1/admin/integracoes/sync-queue?status=pending&page=&limit=` | Admin |

**Response `GET /status`:**
```json
{
  "data": {
    "erp": { "status": "offline", "last_check": "2026-03-28T10:00:00Z", "pending_jobs": 15 },
    "sinter": { "status": "online", "last_check": "2026-03-28T10:05:00Z", "pending_jobs": 0 }
  }
}
```

**Health check:** `node-cron` a cada 5min executa `adapter.healthCheck()` → atualiza `integration_status`.
**Fila:** Quando ERP offline, jobs enfileirados em `sync_queue`. Quando ERP retorna → worker processa fila FIFO.
**Alerta:** WebSocket push para admins logados quando status muda.

**Testes:**
- Integration: mock ERP offline → alterações enfileiradas; mock ERP retorna → fila processada.
- Integration: status endpoint reflete estado correto.

---

#### IMP-08-004 — Integração com SINTER/CIB
**Derives:** DES-08-004

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/admin/integracoes/sinter/config` | Admin |
| PUT | `/api/v1/admin/integracoes/sinter/config` | Admin |
| GET | `/api/v1/admin/integracoes/sinter/pending` | Admin |
| POST | `/api/v1/admin/integracoes/sinter/retry/:propertyId` | Admin |

**Config:**
```json
{
  "base_url": "https://api.sinter.gov.br/v1",
  "credentials": { "certificate": "...", "api_key": "..." },
  "api_version": "v1"
}
```

**Fluxo automático:**
1. `propertyCreated` event → BullMQ `sync-sinter` → POST para SINTER → CIB retornado → salvo em `properties.cib`.
2. `propertyUpdated` (campos relevantes) → BullMQ `sync-sinter` → PUT para SINTER.

**Retry:** 3x backoff exponencial (1min, 5min, 15min). Após 3 falhas → `cib_pending = true`.
**`GET /pending`:** Lista imóveis com `cib_pending = true`.
**`POST /retry/:propertyId`:** Re-enfileira job manualmente.

**Schemas:** `src/modules/integracoes/sinter-schemas/v1.json`, `v2.json`.

**Testes:**
- Integration: mock SINTER → novo imóvel recebe CIB; SINTER offline → `cib_pending = true`.
- Unit: schema v1 e v2 mapeiam campos corretamente.

---

#### IMP-08-005 — Integração com Google Street View
**Derives:** DES-08-005

**Deploy:** 100% frontend via iframe (Google Maps Embed API).
**Config:** `tenant_config.google_streetview_key`.
**API:** `GET /api/v1/portal/config` retorna `streetview_key` para o frontend (campo público).
**Testes:** E2E: com key configurada → Street View abre; sem key → botão oculto.

---

#### IMP-08-006 — Importação de Imagens de Satélite
**Derives:** DES-08-006

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/admin/integracoes/satellite/upload` | Admin |
| GET | `/api/v1/admin/integracoes/satellite/providers` | Admin |
| PUT | `/api/v1/admin/integracoes/satellite/providers` | Admin |

**Request `POST /upload`:** Multipart GeoTIFF.
**Validação:** Verificar metadata geoespacial (GDAL `gdalinfo`). Se datum ≠ 4674 → converter via `gdalwarp`.
**Storage:** S3 `/{tenant}/satellite/{date}/{uuid}.tif`.
**Feature flag:** `satellite_module` controla disponibilidade.

**Testes:**
- Integration: upload GeoTIFF válido → registrado em `satellite_images`; GeoTIFF sem metadados → erro.

---

#### IMP-08-007 — Integração com Sistemas de Protocolo
**Derives:** DES-08-007

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/admin/integracoes/protocol/config` | Admin |
| PUT | `/api/v1/admin/integracoes/protocol/config` | Admin |
| POST | `/api/v1/admin/integracoes/protocol/test` | Admin |

**Config:**
```json
{
  "mode": "api",
  "base_url": "https://protocolo.municipio.gov.br/api",
  "credentials": { "api_key": "xxx" },
  "field_mapping": { "descricao": "texto", "tipo": "categoria" }
}
```

**Fluxo (acionado por IMP-07-010):**
1. Cidadão abre protocolo → salvo localmente.
2. Se `protocol_integration` configurado e `mode = 'api'`:
   - `protocolIntegrationService.forward(protocol)` → POST para sistema terceiro.
   - Sucesso → `external_protocol_number` salvo.
   - Falha → `status = 'pendente_encaminhamento'`; retentativa via `node-cron` a cada 15min.
3. Se `mode = 'link'`: frontend redireciona cidadão para URL externa.

**Testes:**
- Integration: mock API terceira → protocolo encaminhado → número externo salvo.
- Integration: API falha → protocolo salvo localmente com status pendente; retentativa funciona.
