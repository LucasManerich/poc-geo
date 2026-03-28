---
document_type: tsd
domain: cadastro_territorial
derives_from: sdd/04_domain_cadastro_territorial_sdd.md
file: tsd/04_domain_cadastro_territorial_tsd.md
---

# TSD-04 — Implementação: Cadastro Territorial

## 1. Schema e Índices

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| `properties` | `inscricao_imobiliaria` | UNIQUE | Identificador principal |
| `properties` | `geom` | GiST | Queries espaciais |
| `properties` | `bairro` | B-tree | Indicadores por bairro |
| `properties` | `logradouro_id` | B-tree | JOIN com logradouros |
| `businesses` | `cnpj` | UNIQUE | Identificador empresa |
| `businesses` | `property_id` | B-tree | Lookup por imóvel |
| `businesses` | `cnae` | B-tree | Filtro por atividade |
| `streets` | Full-Text on `nome` | GIN | Busca por logradouro |
| `pois` | `geom` | GiST | Camada no mapa |
| `pois` | `categoria` | B-tree | Filtro por categoria |
| `environmental_restrictions` | `property_id` | B-tree | Lookup por imóvel |
| `attachments` | `property_id` | B-tree | Lookup por imóvel |

**Migrations:** `040_create_properties`, `041_create_env_restrictions`, `042_create_attachments`, `043_create_businesses`, `044_create_streets`, `045_create_pois`, `046_create_poi_categories`, `047_create_viability_reports`.

---

## 2. Implementações

---

#### IMP-04-001 — Manutenção do Cadastro Imobiliário
**Derives:** DES-04-001

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| GET | `/api/v1/cadastro/properties/:id` | Consultar imóvel | `cadastro.property.read` |
| POST | `/api/v1/cadastro/properties` | Criar imóvel | `cadastro.property.write` |
| PUT | `/api/v1/cadastro/properties/:id` | Atualizar imóvel | `cadastro.property.write` |
| GET | `/api/v1/cadastro/properties/by-polygon/:polygonId` | Via clique no mapa | `cadastro.property.read` |

**Request `POST /properties`:**
```json
{
  "inscricao_imobiliaria": "01.02.003.0001",
  "matricula": "12345",
  "tipo": "lote",
  "area_terreno": 360.5,
  "proprietario_cpf_cnpj": "123.456.789-00",
  "logradouro_id": 10,
  "numero": "150",
  "bairro": "Centro"
}
```

**Erros:** 409 `DUPLICATE_INSCRICAO` — inscrição já existe. 422 `INVALID_CPF` — CPF/CNPJ inválido.

**Testes:**
- Unit: validar CPF com dígitos verificadores (válidos e inválidos).
- Integration: criar imóvel → GET retorna dados; CPF armazenado criptografado no DB.
- Integration: inscrição duplicada retorna 409.

---

#### IMP-04-002 — Restrições Ambientais
**Derives:** DES-04-002

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/cadastro/properties/:id/restrictions` | `cadastro.restriction.write` |
| GET | `/api/v1/cadastro/properties/:id/restrictions` | `cadastro.property.read` |
| DELETE | `/api/v1/cadastro/properties/:id/restrictions/:rid` | `cadastro.restriction.write` |

**Request `DELETE` body:** `{ "justificativa": "Laudo atualizado revoga APP" }` — registrado em audit_logs.

**Testes:** Integration: adicionar APP → visível na consulta; remover → audit log com justificativa.

---

#### IMP-04-003 — Viabilidade de Construção
**Derives:** DES-04-003

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/cadastro/properties/:id/viability` | `cadastro.property.read` |
| POST | `/api/v1/cadastro/properties/:id/viability` | `zoneamento.viability.write` |

**Response `GET /viability`:**
```json
{
  "data": {
    "zone": { "codigo": "ZR-2", "nome": "Zona Residencial 2" },
    "rules": { "pavimentos_max": 2, "afastamento_frontal": 4.0 },
    "permissibilities": [{ "uso": "residencial_uni", "classificacao": "permitido" }],
    "restrictions": [{ "tipo": "APP", "descricao": "Margem do rio" }],
    "parecer": null
  }
}
```

**Testes:** Integration: imóvel com zona definida → retorna regras; sem zona → `zone: null`.

---

#### IMP-04-004 — Gestão de Anexos
**Derives:** DES-04-004

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/cadastro/properties/:id/attachments` | `cadastro.attachment.write` |
| GET | `/api/v1/cadastro/properties/:id/attachments` | `cadastro.property.read` |
| GET | `/api/v1/cadastro/attachments/:aid/download` | `cadastro.property.read` |

**Limits:** Max 20MB. Formatos: PDF, JPG, PNG, DWG, DXF.
**Erros:** 413 `FILE_TOO_LARGE`. 415 `UNSUPPORTED_FORMAT`.
**Testes:** Integration: upload JPG → listado em attachments; download retorna signed URL funcional.

---

#### IMP-04-005 — Coordenadas e Importação em Massa
**Derives:** DES-04-005

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/cadastro/properties/:id/coordinates?format=json\|csv\|txt` | `cadastro.property.read` |
| POST | `/api/v1/cadastro/properties/import-coordinates` | `cadastro.import.write` |

**Response `GET /coordinates`:** Array de vértices em `{ lat, lng, utm_e, utm_n, utm_zone }`.
**Testes:** Integration: imóvel com polígono → coordenadas retornadas em ambos formatos; import TXT → polígonos vinculados por inscrição.

---

#### IMP-04-006 — Consulta Multicritério
**Derives:** DES-04-006

**API:**
- `GET /api/v1/cadastro/properties/search?q=Rua+das+Flores&field=endereco&page=1&limit=20`

**Campos de busca:** `cpf`, `endereco`, `cadastro`, `matricula`, `inscricao`.
**Busca por CPF:** Hash do input comparado com `cpf_hash` armazenado (evita decrypt em massa).
**Testes:** Integration: busca por endereço parcial retorna resultados; busca por CPF encontra proprietário.

---

#### IMP-04-007 — Extração por Raio e Exportação
**Derives:** DES-04-007

**API:**
- `GET /api/v1/cadastro/properties/by-radius?lat=-26.91&lng=-49.07&radius=500&format=json`
- `POST /api/v1/cadastro/properties/export` — body: `{ property_ids: [...], format: "xlsx" }` → BullMQ job.

**Erros:** 422 `RADIUS_TOO_LARGE` (max 5000m).
**Testes:** Integration: extração com raio 500m retorna imóveis dentro; export XLSX gerado com dados mascarados.

---

#### IMP-04-008 — Cadastro Mobiliário
**Derives:** DES-04-008

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/cadastro/businesses` | `cadastro.business.write` |
| GET | `/api/v1/cadastro/businesses?cnae=&q=&page=&limit=` | `cadastro.business.read` |
| PUT | `/api/v1/cadastro/businesses/:id` | `cadastro.business.write` |
| POST | `/api/v1/cadastro/businesses/export?cnae=&format=csv\|xlsx` | `cadastro.export` |

**Erros:** 409 `DUPLICATE_CNPJ`.
**Testes:** Integration: criar empresa vinculada a imóvel; filtrar por CNAE retorna resultados.

---

#### IMP-04-009 — Manutenção de Logradouros
**Derives:** DES-04-009

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/cadastro/streets/search?q=` | Sim |
| POST | `/api/v1/cadastro/streets` | `cadastro.street.write` |
| PUT | `/api/v1/cadastro/streets/:id` | `cadastro.street.write` |

**Busca:** Full-Text Search com `to_tsvector('portuguese', nome)`. Autocomplete com LIMIT 10.
**Testes:** Integration: busca "Flores" retorna "Rua das Flores"; campos de infraestrutura salvos.

---

#### IMP-04-010 — Pontos de Interesse
**Derives:** DES-04-010

**API:**

| Método | Path | Auth |
|--------|------|------|
| POST | `/api/v1/cadastro/pois` | `cadastro.poi.write` |
| GET | `/api/v1/cadastro/pois?categoria=&bbox=` | Sim |
| PUT | `/api/v1/cadastro/pois/:id` | `cadastro.poi.write` |

**Request `POST /pois`:**
```json
{ "nome": "UBS Centro", "categoria": "saude", "lat": -26.91, "lng": -49.07, "descricao": "Unidade Básica" }
```

**Migration:** `046_create_poi_categories` com seed: saude, educacao, infraestrutura, lazer, seguranca.
**Testes:** Integration: criar POI → aparece em busca por categoria e bbox.
