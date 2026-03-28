---
document_type: prd
domain: fiscalizacao
derives_from: brd/06_domain_fiscalizacao_brd.md
file: prd/06_domain_fiscalizacao_prd.md
requirements:
  - id: REQ-06-001
    title: "Abertura de Fiscalização"
    depends_on: [REQ-04-001, REQ-03-001]
  - id: REQ-06-002
    title: "Registro de Vistorias e Visitas"
    depends_on: [REQ-06-001]
  - id: REQ-06-003
    title: "Geração de Documentos de Fiscalização"
    depends_on: [REQ-06-002, REQ-02-005]
  - id: REQ-06-004
    title: "Legendas e Status no Mapa"
    depends_on: [REQ-06-001, REQ-03-005]
  - id: REQ-06-005
    title: "Indicadores por Bairro"
    depends_on: [REQ-04-001, REQ-03-001]
---

# PRD-06 — Domínio: Fiscalização

## 1. Processo de Fiscalização

---

#### REQ-06-001 — Abertura de Fiscalização
**Derives:** BIZ-06-001, BIZ-06-002, BIZ-06-003

**Descrição:** Seleção de imóvel no mapa para abertura de processo de fiscalização, com registro de motivo, servidor responsável e data. Suporte a múltiplas fiscalizações por imóvel com histórico completo.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Imóvel existente no cadastro; servidor autenticado com perfil de fiscalização.
- **Fluxo principal:**
  1. Servidor seleciona imóvel no mapa.
  2. Clica em "Abrir Fiscalização".
  3. Preenche: motivo (construção sem alvará / uso irregular / outros), descrição adicional.
  4. Sistema registra automaticamente: servidor responsável (do login), data de abertura, imóvel.
  5. Fiscalização criada com status "Aberta".
- **Fluxos alternativos:**
  - 2a. Imóvel já possui fiscalizações anteriores → sistema lista histórico; servidor pode prosseguir com nova.
- **Pós-condições:** Fiscalização registrada e vinculada ao imóvel.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Fiscalização aberta a partir de seleção de imóvel no mapa
- [ ] Registro: imóvel, motivo, servidor responsável, data
- [ ] Múltiplas fiscalizações por imóvel com histórico
- [ ] Status inicial: "Aberta"
- [ ] Motivos configuráveis pelo município

**Dependências:** `depends_on: [REQ-04-001, REQ-03-001]`

---

#### REQ-06-002 — Registro de Vistorias e Visitas
**Derives:** BIZ-06-004, BIZ-06-005, BIZ-06-006

**Descrição:** Cadastro de vistorias e visitas vinculadas a uma fiscalização, com registro de data, servidor fiscal, observações, parecer e anexação de fotos/documentos.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Fiscalização aberta.
- **Fluxo principal:**
  1. Servidor acessa fiscalização existente.
  2. Clica em "Registrar Vistoria" ou "Registrar Visita".
  3. Preenche: data, observações, parecer (para vistoria).
  4. Anexa fotografias e documentos comprobatórios.
  5. Salva o registro.
- **Fluxos alternativos:**
  - 2a. Visita (tentativa de contato) → campos simplificados: data, observação.
  - 4a. Sem anexos → permitido salvar sem fotos.
- **Pós-condições:** Vistoria/visita registrada no histórico da fiscalização.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Vistoria: data, servidor fiscal, observações, parecer
- [ ] Visita: data, observação (simplificado)
- [ ] Anexação de fotos (JPG, PNG) e documentos (PDF)
- [ ] Múltiplas vistorias/visitas por fiscalização
- [ ] Ordenação cronológica no histórico

**Dependências:** `depends_on: [REQ-06-001]`

---

#### REQ-06-003 — Geração de Documentos de Fiscalização
**Derives:** BIZ-06-007, BIZ-06-008

**Descrição:** Geração de documentos (notificações, autos de infração, termos de embargo) a partir de templates configuráveis, contendo dados do imóvel, proprietário, observações da vistoria e fundamentação legal.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Vistoria registrada; template configurado pelo município.
- **Fluxo principal:**
  1. Servidor acessa fiscalização com vistoria registrada.
  2. Clica em "Gerar Documento".
  3. Seleciona tipo: Notificação / Auto de Infração / Termo de Embargo.
  4. Sistema preenche template com dados: imóvel (endereço, inscrição, proprietário), observações da vistoria, fundamentação legal.
  5. Servidor revisa, edita campos livres e confirma.
  6. Documento gerado em PDF com código de verificação.
- **Fluxos alternativos:**
  - 4a. Template não configurado → erro "Template de [tipo] não configurado. Contate o administrador."
- **Pós-condições:** Documento gerado, vinculado à fiscalização e disponível para impressão/envio.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Tipos: notificação, auto de infração, termo de embargo (configuráveis)
- [ ] Template preenchido automaticamente com dados do cadastro e da vistoria
- [ ] Campos editáveis antes da geração final
- [ ] PDF com código de verificação de autenticidade
- [ ] Documento vinculado à fiscalização no histórico

**Dependências:** `depends_on: [REQ-06-002, REQ-02-005]`

---

## 2. Visualização e Indicadores

---

#### REQ-06-004 — Legendas e Status no Mapa
**Derives:** BIZ-06-009, BIZ-06-010, BIZ-06-011

**Descrição:** Legendas visuais nos imóveis conforme status de fiscalização (fiscalizado, embargado, notificado, regular, pendente), configuráveis por município, com filtro no mapa.

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Fiscalizações existentes com status definidos; camada de fiscalização habilitada.
- **Fluxo principal:**
  1. Servidor ativa camada de fiscalização no mapa.
  2. Imóveis com fiscalização exibem ícone/cor conforme status.
  3. Servidor clica em "Filtrar" e seleciona um status (ex.: "Embargado").
  4. Mapa exibe apenas imóveis com o status selecionado.
- **Fluxos alternativos:**
  - 3a. Filtro múltiplo: selecionar mais de um status.
  - 1a. Administrador configura cores/ícones/nomenclaturas dos status.
- **Pós-condições:** Mapa filtrado conforme legendas de fiscalização.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Status visuais: fiscalizado, embargado, notificado, regular, pendente de vistoria
- [ ] Cores, ícones e nomenclaturas configuráveis pelo município
- [ ] Filtro por status no mapa (um ou múltiplos)
- [ ] Legenda visível no painel de camadas

**Dependências:** `depends_on: [REQ-06-001, REQ-03-005]`

---

#### REQ-06-005 — Indicadores por Bairro
**Derives:** BIZ-06-012, BIZ-06-013, BIZ-06-014

**Descrição:** Indicadores por bairro: nº de construções, área construída, lotes baldios e ITBIs (crescimento). Visualizáveis no mapa como mapa de calor ou escala de cores.

**Use Case:**
- **Ator:** Servidor Público (Urbanismo/Planejamento)
- **Pré-condições:** Base cadastral populada; dados de ITBI via integração ERP.
- **Fluxo principal:**
  1. Servidor acessa módulo de Indicadores.
  2. Seleciona indicador: construções / área construída / lotes baldios / ITBIs.
  3. Sistema calcula por bairro e exibe ranking.
  4. Servidor ativa visualização no mapa (escala de cores por bairro).
- **Fluxos alternativos:**
  - 4a. Mapa de calor como alternativa à escala de cores.
  - 2a. Filtro por período (para ITBIs).
- **Pós-condições:** Indicadores exibidos no ranking e no mapa.
- **Exceções:** Dados de ITBI indisponíveis (ERP offline) → indicador de ITBI com aviso "Dados podem estar desatualizados."

**Critérios de aceite:**
- [ ] Indicadores: nº construções, área construída, lotes baldios, nº ITBIs
- [ ] Cálculo por bairro com ranking
- [ ] Visualização no mapa: escala de cores ou mapa de calor
- [ ] ITBIs como indicador de crescimento/aquecimento imobiliário
- [ ] Filtro por período para ITBIs

**Dependências:** `depends_on: [REQ-04-001, REQ-03-001]`
