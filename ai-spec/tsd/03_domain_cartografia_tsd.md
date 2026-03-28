---
document_type: tsd
domain: cartografia
derives_from: sdd/03_domain_cartografia_sdd.md
file: tsd/03_domain_cartografia_tsd.md
---

# TSD-03 — Implementação: Cartografia

## 1. Schema e Índices

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| `polygons` | `geom` | GiST | Queries espaciais (ST_Intersects, ST_Contains) |
| `polygons` | `layer_id` | B-tree | Filtro por camada |
| `polygons` | `entity_type, entity_id` | B-tree | Lookup de polígono por entidade |
| `layers` | `name` | UNIQUE | Unicidade por tenant DB |
| `layers` | `visibility, visible_portal` | B-tree | Filtro de camadas por visibilidade |
| `satellite_images` | `bbox` | GiST | Busca por cobertura espacial |
| `satellite_images` | `capture_date` | B-tree | Busca temporal |
| `import_jobs` | `status` | B-tree | Listagem de jobs por status |

**Migrations:** `030_create_layers`, `031_create_layer_permissions`, `032_create_polygons`, `033_create_satellite_images`, `034_create_import_jobs`.

---

## 2. Implementações

---

#### IMP-03-001 — Mapa Interativo Web
**Derives:** DES-03-001

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| GET | `/api/v1/cartografia/layers` | Listar camadas visíveis ao usuário | Sim |
| GET | `/api/v1/cartografia/layers/:id/features?bbox=` | GeoJSON por viewport | Sim |

**Response `GET /layers/:id/features?bbox=-49.1,-27.0,-49.0,-26.9`:**
```json
{
  "data": {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "geometry": { "type": "Polygon", "coordinates": [[...]] },
        "properties": { "id": 1, "entity_type": "property", "entity_id": 42 }
      }
    ]
  }
}
```

**Erros:** 404 — camada não existe. 403 — camada restrita, sem permissão.

**Testes:**
- Unit: `layerService.getFeaturesByBBox()` filtra por ST_Intersects.
- Integration: GET retorna GeoJSON válido; camada restrita retorna 403.
- E2E: mapa carrega em < 3s com 1000 polígonos no viewport.

---

#### IMP-03-002 — Provedores Plug-and-Play
**Derives:** DES-03-002

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| GET | `/api/v1/admin/map-provider` | Config atual do provedor | Admin |
| PUT | `/api/v1/admin/map-provider` | Atualizar provedor + API key | Admin |
| POST | `/api/v1/admin/map-provider/validate` | Testar API key | Admin |

**Request `PUT /admin/map-provider`:**
```json
{ "provider": "google_maps", "api_key": "AIza..." }
```

**Response `POST /admin/map-provider/validate`:**
```json
{ "data": { "valid": true, "provider": "google_maps" } }
// ou
{ "error": { "code": "INVALID_API_KEY", "message": "Chave de acesso inválida" } }
```

**Testes:**
- Unit: factory resolve provider correto por nome.
- Integration: PUT atualiza `tenant_config`; validate com key inválida retorna erro.

---

#### IMP-03-003 — Desenho e Edição de Polígonos
**Derives:** DES-03-003

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| POST | `/api/v1/cartografia/polygons` | Criar polígono | `cartografia.polygon.write` |
| PUT | `/api/v1/cartografia/polygons/:id` | Editar geometria | `cartografia.polygon.write` |
| DELETE | `/api/v1/cartografia/polygons/:id` | Excluir polígono | `cartografia.polygon.write` |

**Request `POST /polygons`:**
```json
{
  "geometry": { "type": "Polygon", "coordinates": [[[-49.07, -26.91], ...]] },
  "layer_id": 5,
  "entity_type": "property",
  "entity_id": 42
}
```

**Validações:** `ST_IsValid(geom)` = true; `ST_GeometryType(geom)` = 'ST_Polygon'; `entity_id` existe na tabela correspondente.

**Erros:** 422 `INVALID_GEOMETRY` — autointerseção. 404 — entity_id não encontrado.

**Testes:**
- Unit: rejeitar polígono com autointerseção.
- Integration: criar polígono → verificar no banco com ST_IsValid.

---

#### IMP-03-004 — Associação de Polígonos a Entidades
**Derives:** DES-03-004

**Deploy:** Campos `entity_type` e `entity_id` na tabela `polygons`. Tabela N:N `polygon_layers` para múltiplas camadas.
**Migration:** `035_create_polygon_layers`.
**Testes:** Unit: polígono associado a 2 camadas; remoção de associação não exclui polígono.

---

#### IMP-03-005 — Gestão de Camadas
**Derives:** DES-03-005

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| POST | `/api/v1/cartografia/layers` | Criar camada | `cartografia.layer.admin` |
| GET | `/api/v1/cartografia/layers` | Listar camadas (filtradas por role) | Sim |
| PUT | `/api/v1/cartografia/layers/:id` | Editar camada | `cartografia.layer.admin` |
| DELETE | `/api/v1/cartografia/layers/:id` | Excluir camada | `cartografia.layer.admin` |
| PATCH | `/api/v1/cartografia/layers/reorder` | Reordenar z-index | `cartografia.layer.admin` |

**Request `PATCH /layers/reorder`:**
```json
{ "order": [{ "id": 3, "z_index": 1 }, { "id": 1, "z_index": 2 }] }
```

**Testes:**
- Integration: criar camada restrita → usuário sem role não vê; admin vê.
- Integration: reordenar → z_index atualizado.

---

#### IMP-03-006 — Ferramentas de Medição
**Derives:** DES-03-006

**Deploy:** 100% frontend (turf.js). Sem endpoint backend.
**Testes:** E2E: ativar ferramenta de distância → clicar 2 pontos → valor exibido em metros.

---

#### IMP-03-007 — Sistema de Coordenadas
**Derives:** DES-03-007

**Deploy:** 100% frontend (proj4js). Sem endpoint backend.
**Env vars frontend:** Nenhuma (SIRGAS 2000 EPSG:4674 hardcoded como padrão).
**Testes:** Unit: converter lat/long para UTM e vice-versa. E2E: "Ir para coordenada" centraliza mapa.

---

#### IMP-03-008 — Importação de Arquivos Geoespaciais
**Derives:** DES-03-008

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| POST | `/api/v1/cartografia/import` | Upload arquivo (multipart) | `cartografia.import.write` |
| GET | `/api/v1/cartografia/import/:jobId` | Status do job | Sim |

**Response `POST /import`:**
```json
{ "data": { "jobId": "abc-123", "status": "pending" } }
```

**Response `GET /import/:jobId` (após processamento):**
```json
{
  "data": {
    "status": "done",
    "total": 500,
    "imported": 485,
    "converted": 12,
    "rejected": 15,
    "errors": [{ "record": 23, "reason": "Geometria inválida" }]
  }
}
```

**Deploy:** GDAL (`ogr2ogr`) instalado no container. BullMQ fila `import-geospatial`.
**Testes:** Integration: upload SHP válido → job completa → polígonos criados. Upload com datum SAD69 → conversão para 4674.

---

#### IMP-03-009 — Visualização de Imagens de Satélite
**Derives:** DES-03-009

**API:**

| Método | Path | Descrição | Auth |
|--------|------|-----------|------|
| GET | `/api/v1/cartografia/satellite/dates?bbox=` | Listar datas disponíveis | Sim |
| GET | `/api/v1/cartografia/satellite/:id/tile` | Servir tile de imagem | Sim |

**Testes:** Integration: upload GeoTIFF → data aparece em `/dates`; tile servido corretamente.
**Feature flag:** `satellite_module` — rotas retornam 404 se desabilitado.

---

#### IMP-03-010 — Integração Street View
**Derives:** DES-03-010

**Deploy:** 100% frontend (Google Maps Embed API iframe).
**Config:** `tenant_config.google_streetview_key`.
**Testes:** E2E: com key configurada, clicar "Street View" abre painel. Sem key, botão oculto.

---

#### IMP-03-011 — Exportação e Impressão
**Derives:** DES-03-011

**Deploy:** 100% frontend (`leaflet-image` + `jsPDF`).
**Testes:** E2E: clicar "Exportar" → arquivo PNG/PDF baixado com mapa visível.
