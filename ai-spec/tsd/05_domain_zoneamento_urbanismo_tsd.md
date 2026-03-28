---
document_type: tsd
domain: zoneamento_urbanismo
derives_from: sdd/05_domain_zoneamento_urbanismo_sdd.md
file: tsd/05_domain_zoneamento_urbanismo_tsd.md
---

# TSD-05 — Implementação: Zoneamento e Urbanismo

## 1. Schema e Índices

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| `zones` | `geom` | GiST | ST_Contains para zona↔imóvel |
| `zones` | `codigo` | UNIQUE | Identificador da zona |
| `zone_rules` | `zone_id` | B-tree (FK) | Lookup de regras por zona |
| `zone_permissibilities` | `zone_id` | B-tree (FK) | Lookup de permissibilidade |
| `subdivisions` | `performed_at` | B-tree | Histórico cronológico |
| `pgv` | `referencia_tipo, referencia_id` | B-tree | Lookup valor por logradouro/bairro |
| `pgv` | `vigencia` | B-tree | Filtro temporal |

**Migrations:** `050_create_zones`, `051_create_zone_rules`, `052_create_zone_permissibilities`, `053_create_subdivisions`, `054_create_pgv`.

---

## 2. Implementações

---

#### IMP-05-001 — Gestão de Zonas no Mapa
**Derives:** DES-05-001

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| POST | `/api/v1/zoneamento/zones` | Criar zona | `zoneamento.zone.admin` |
| GET | `/api/v1/zoneamento/zones` | Listar todas (GeoJSON) | Sim |
| PUT | `/api/v1/zoneamento/zones/:id` | Editar zona | `zoneamento.zone.admin` |
| PUT | `/api/v1/zoneamento/zones/:id/rules` | Atualizar parâmetros | `zoneamento.zone.admin` |
| DELETE | `/api/v1/zoneamento/zones/:id` | Excluir zona | `zoneamento.zone.admin` |

**Request `POST /zones`:**
```json
{
  "codigo": "ZR-2",
  "nome": "Zona Residencial 2",
  "geom": { "type": "MultiPolygon", "coordinates": [[[[...]]]] },
  "cor_hex": "#4CAF50",
  "opacidade": 0.3
}
```

**Response `GET /zones`:** GeoJSON FeatureCollection com propriedades: `id`, `codigo`, `nome`, `cor_hex`, `opacidade`.

**Detecção de sobreposição (warning, não bloqueia):**
```json
{ "data": { "id": 15 }, "warnings": [{ "type": "ZONE_OVERLAP", "overlaps": [{ "id": 3, "nome": "ZC-1" }] }] }
```

**Testes:**
- Integration: criar zona → GET retorna GeoJSON válido; zona com overlap retorna warning.

---

#### IMP-05-002 — Regras de Permissibilidade
**Derives:** DES-05-002

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/zoneamento/zones/:id/permissibilities` | Sim |
| POST | `/api/v1/zoneamento/zones/:id/permissibilities` | `zoneamento.zone.admin` |
| PUT | `/api/v1/zoneamento/permissibilities/:pid` | `zoneamento.zone.admin` |
| DELETE | `/api/v1/zoneamento/permissibilities/:pid` | `zoneamento.zone.admin` |

**Request `POST`:**
```json
{ "uso": "comercial", "classificacao": "permissivel", "condicoes": "Máximo 200m² de área construída" }
```

**Testes:** Integration: criar permissibilidade → visível na consulta da zona.

---

#### IMP-05-003 — Consulta de Zoneamento por Imóvel
**Derives:** DES-05-003

**API:**
- `GET /api/v1/zoneamento/by-property/:propertyId`

**Response:**
```json
{
  "data": {
    "zones": [{
      "codigo": "ZR-2", "nome": "Zona Residencial 2",
      "rules": { "pavimentos_max": 2, "afastamento_frontal": 4.0, "taxa_ocupacao": 0.6 },
      "permissibilities": [{ "uso": "residencial_uni", "classificacao": "permitido" }]
    }]
  }
}
```

**Erros:** 404 — imóvel não encontrado. `zones: []` se imóvel sem zona (não é erro).
**Testes:** Integration: imóvel dentro de zona → retorna zona + regras; imóvel sem geometria → `zones: []`.

---

#### IMP-05-004 — Desmembramento de Lotes
**Derives:** DES-05-004

**API:**
- `POST /api/v1/zoneamento/subdivisions/split`

**Request:**
```json
{
  "property_id": 42,
  "split_line": { "type": "LineString", "coordinates": [[-49.07, -26.91], [-49.06, -26.91]] }
}
```

**Response (sucesso):**
```json
{
  "data": {
    "type": "desmembramento",
    "origin": { "id": 42, "inscricao": "01.02.003.0001" },
    "results": [
      { "id": 100, "inscricao": "01.02.003.0002", "area": 180.5 },
      { "id": 101, "inscricao": "01.02.003.0003", "area": 180.0 }
    ]
  }
}
```

**Erros:**
- 422 `AREA_BELOW_MINIMUM` — `{ min_required: 125, actual: 98.5, zone: "ZR-2" }`
- 422 `INVALID_SPLIT_LINE` — linha não cruza o polígono

**Testes:**
- Unit: `turf.lineSplit` divide polígono em 2 partes válidas.
- Unit: rejeitar se área resultante < mínimo.
- Integration: desmembrar → 2 novos imóveis criados; original inativado; histórico em `subdivisions`.

---

#### IMP-05-005 — Remembramento de Lotes
**Derives:** DES-05-005

**API:**
- `POST /api/v1/zoneamento/subdivisions/merge`

**Request:**
```json
{ "property_ids": [42, 43] }
```

**Erros:** 422 `NOT_CONTIGUOUS` — lotes não compartilham borda. 422 `MIN_PROPERTIES` — menos de 2 lotes.

**Testes:**
- Unit: `ST_Union` de 2 polígonos contíguos gera polígono válido.
- Integration: remembrar → novo imóvel criado; originais inativados; histórico registrado.

---

#### IMP-05-006 — Planta Genérica de Valores
**Derives:** DES-05-006

**API:**

| Método | Path | Auth |
|--------|------|------|
| GET | `/api/v1/zoneamento/pgv?tipo=logradouro&vigencia=` | Sim |
| POST | `/api/v1/zoneamento/pgv` | `zoneamento.pgv.write` |
| POST | `/api/v1/zoneamento/pgv/bulk` | `zoneamento.pgv.write` |
| GET | `/api/v1/zoneamento/pgv/compare?vigencia_a=&vigencia_b=` | Sim |
| GET | `/api/v1/zoneamento/pgv/ranking?tipo=&order=desc&limit=20` | Sim |

**Request `POST /pgv/bulk` (multipart CSV):**
```csv
tipo;referencia_id;valor_m2;vigencia
logradouro;10;150.00;2025-01-01
logradouro;11;200.00;2025-01-01
```

**Testes:**
- Integration: bulk import CSV → valores atualizados; compare retorna diff entre 2 vigências.
- Unit: ranking ordena corretamente por valor_m2.
