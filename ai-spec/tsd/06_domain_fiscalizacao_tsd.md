---
document_type: tsd
domain: fiscalizacao
derives_from: sdd/06_domain_fiscalizacao_sdd.md
file: tsd/06_domain_fiscalizacao_tsd.md
---

# TSD-06 — Implementação: Fiscalização

## 1. Schema e Índices

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| `inspections` | `property_id` | B-tree | Histórico por imóvel |
| `inspections` | `status` | B-tree | Filtro por status (legendas) |
| `inspections` | `responsible_user_id` | B-tree | Fiscalizações por servidor |
| `surveys` | `inspection_id` | B-tree (FK) | Vistorias por fiscalização |
| `survey_attachments` | `survey_id` | B-tree (FK) | Anexos por vistoria |
| `inspection_documents` | `inspection_id` | B-tree (FK) | Documentos por fiscalização |
| `inspection_documents` | `verification_code` | UNIQUE | Verificação pública |
| `legend_config` | `status` | UNIQUE | Config de legenda por status |

**Migrations:** `060_create_inspections`, `061_create_surveys`, `062_create_survey_attachments`, `063_create_inspection_documents`, `064_create_legend_config`, `065_create_inspection_motives`, `066_create_inspection_responses`.

---

## 2. Implementações

---

#### IMP-06-001 — Abertura de Fiscalização
**Derives:** DES-06-001

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| POST | `/api/v1/fiscalizacao/inspections` | Abrir fiscalização | `fiscalizacao.inspection.write` |
| GET | `/api/v1/fiscalizacao/inspections?property_id=&status=&page=&limit=` | Listar | Sim |
| GET | `/api/v1/fiscalizacao/inspections/:id` | Detalhe com surveys e docs | Sim |

**Request `POST /inspections`:**
```json
{ "property_id": 42, "motivo": "construcao_sem_alvara", "descricao": "Obra identificada via satélite" }
```

**Response 201:**
```json
{ "data": { "id": 1, "status": "aberta", "opened_at": "2026-03-28T10:00:00Z" } }
```

**Testes:**
- Integration: abrir fiscalização → status "aberta"; `responsible_user_id` preenchido do `req.user`.
- Integration: listar por `property_id` retorna múltiplas fiscalizações.

---

#### IMP-06-002 — Registro de Vistorias e Visitas
**Derives:** DES-06-002

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/fiscalizacao/inspections/:id/surveys` | `fiscalizacao.survey.write` |
| GET | `/api/v1/fiscalizacao/inspections/:id/surveys` | Sim |
| POST | `/api/v1/fiscalizacao/surveys/:sid/attachments` | `fiscalizacao.survey.write` |

**Request `POST /surveys`:**
```json
{ "tipo": "vistoria", "data": "2026-03-28", "observacoes": "Construção em alvenaria sem alvará", "parecer": "Irregular — notificar proprietário" }
```

**Validação:** Se `tipo = 'vistoria'`, `parecer` é obrigatório. Se `tipo = 'visita'`, `parecer` é opcional.
**Attachments:** Multipart upload → S3 `/{tenant}/inspections/{inspection_id}/surveys/{survey_id}/`.
**Erros:** 422 `PARECER_REQUIRED` (vistoria sem parecer). 413 `FILE_TOO_LARGE` (> 20MB).

**Testes:**
- Integration: criar vistoria com parecer → 201; criar vistoria sem parecer → 422.
- Integration: anexar foto → listada em survey attachments.

---

#### IMP-06-003 — Geração de Documentos de Fiscalização
**Derives:** DES-06-003

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/fiscalizacao/inspections/:id/documents` | `fiscalizacao.document.write` |
| GET | `/api/v1/fiscalizacao/documents/:did/download` | Sim |
| GET | `/api/public/verify/:code` | Público |

**Request `POST /documents`:**
```json
{ "tipo": "notificacao", "campos_extras": { "prazo_dias": 15 } }
```

**Response 202:** `{ "data": { "jobId": "xyz", "status": "processing" } }`

**Fluxo:** BullMQ `generate-document` → Puppeteer renderiza template → PDF com QR code → S3.
**Deploy:** Puppeteer + Chromium no container (ou sidecar).
**Testes:**
- Integration: gerar notificação → PDF no S3; verificação por código retorna metadados.
- Unit: template Handlebars compila com dados do imóvel e vistoria.

---

#### IMP-06-004 — Legendas e Status no Mapa
**Derives:** DES-06-004

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/fiscalizacao/map-features?bbox=&status=` | `fiscalizacao.inspection.read` |
| GET | `/api/v1/fiscalizacao/legends` | Sim |
| PUT | `/api/v1/admin/fiscalizacao/legends` | `fiscalizacao.legend.admin` |

**Response `GET /map-features`:** GeoJSON FeatureCollection com `properties: { inspection_id, status, cor_hex, icone }`.

**Seed `legend_config`:**

| Status | Cor | Ícone |
|--------|-----|-------|
| aberta | #FFC107 | warning |
| embargado | #F44336 | block |
| notificado | #FF9800 | mail |
| regular | #4CAF50 | check |
| pendente_vistoria | #9C27B0 | search |

**Testes:** Integration: fiscalização embargada → feature GeoJSON com cor vermelha; filtro por status funciona.

---

#### IMP-06-005 — Indicadores por Bairro
**Derives:** DES-06-005

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/fiscalizacao/indicators?type=construcoes\|area\|baldios\|itbi&period=` | Sim |
| GET | `/api/v1/fiscalizacao/indicators/geojson?type=` | Sim |

**Response `GET /indicators?type=baldios`:**
```json
{
  "data": [
    { "bairro": "Centro", "value": 45, "rank": 1 },
    { "bairro": "Jardim América", "value": 32, "rank": 2 }
  ]
}
```

**Response `GET /indicators/geojson?type=baldios`:** GeoJSON FeatureCollection com polígonos de bairros e `value` para choropleth.

**Views materializadas:** `mv_indicators_by_bairro` refreshed via `node-cron` a cada 1h.
**Testes:**
- Integration: com seed de 1000 imóveis → indicadores calculados; ranking ordenado.
- Integration: GeoJSON retorna polígonos de bairro com valores.
