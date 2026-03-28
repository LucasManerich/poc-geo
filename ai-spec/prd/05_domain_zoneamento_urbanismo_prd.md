---
document_type: prd
domain: zoneamento_urbanismo
derives_from: brd/05_domain_zoneamento_urbanismo_brd.md
file: prd/05_domain_zoneamento_urbanismo_prd.md
requirements:
  - id: REQ-05-001
    title: "Gestão de Zonas no Mapa"
    depends_on: [REQ-03-001, REQ-03-003, REQ-03-005, REQ-02-007]
  - id: REQ-05-002
    title: "Regras de Permissibilidade"
    depends_on: [REQ-05-001]
  - id: REQ-05-003
    title: "Consulta de Zoneamento por Imóvel"
    depends_on: [REQ-05-001, REQ-04-001]
  - id: REQ-05-004
    title: "Desmembramento de Lotes"
    depends_on: [REQ-04-001, REQ-05-001, REQ-02-006]
  - id: REQ-05-005
    title: "Remembramento de Lotes"
    depends_on: [REQ-04-001, REQ-05-001, REQ-02-006]
  - id: REQ-05-006
    title: "Planta Genérica de Valores"
    depends_on: [REQ-04-001, REQ-04-009]
---

# PRD-05 — Domínio: Zoneamento e Urbanismo

## 1. Zoneamento

---

#### REQ-05-001 — Gestão de Zonas no Mapa
**Derives:** BIZ-05-001, BIZ-05-002, BIZ-05-005

**Descrição:** Criação e edição de zonas no mapa, associando polígonos a regras de uso e ocupação. Cada zona possui parâmetros configuráveis (pavimentos, afastamentos, taxas) e sobreposição visual com cores e legendas distintas.

**Use Case:**
- **Ator:** Servidor Público (Urbanismo/Planejamento)
- **Pré-condições:** Servidor autenticado com perfil de gestão de zoneamento.
- **Fluxo principal:**
  1. Servidor acessa módulo de Zoneamento.
  2. Cria nova zona: código, nome, descrição.
  3. Desenha polígono(s) da zona no mapa.
  4. Define parâmetros: pavimentos máx., afastamentos (frontal, lateral, fundos), taxa de ocupação, coeficiente de aproveitamento.
  5. Define cor e estilo de renderização para o mapa.
  6. Salva a zona.
- **Fluxos alternativos:**
  - 3a. Edição de zona existente: seleciona zona → edita polígono ou parâmetros.
  - 3b. Sobreposição de zonas → alerta "Zona sobrepõe área de zona [X]. Deseja prosseguir?"
- **Pós-condições:** Zona criada/editada e visível no mapa com cor e legenda.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Parâmetros configuráveis por zona: pavimentos, afastamentos, taxa de ocupação, coeficiente
- [ ] Zonas renderizadas com cores distintas e legenda no mapa
- [ ] Alerta ao criar zona que sobrepõe outra
- [ ] Regras totalmente configuráveis por município (REQ-02-007)

**Dependências:** `depends_on: [REQ-03-001, REQ-03-003, REQ-03-005, REQ-02-007]`

---

#### REQ-05-002 — Regras de Permissibilidade
**Derives:** BIZ-05-003

**Descrição:** Configuração de regras de permissibilidade por zona: usos permitidos, permissíveis (sob condições) e proibidos, para construção e abertura de empresa.

**Use Case:**
- **Ator:** Servidor Público (Urbanismo/Planejamento)
- **Pré-condições:** Zona existente.
- **Fluxo principal:**
  1. Servidor seleciona zona.
  2. Acessa aba "Permissibilidade".
  3. Para cada tipo de uso (residencial, comercial, industrial, misto, serviços), define a classificação:

| Uso | Classificação | Condições |
|-----|--------------|-----------|
| Residencial unifamiliar | Permitido | — |
| Residencial multifamiliar | Permissível | Máx. 4 unidades |
| Comercial varejista | Permitido | — |
| Industrial pesada | Proibido | — |

  4. Salva as regras.
- **Pós-condições:** Regras de permissibilidade configuradas para a zona.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Três classificações: permitido, permissível (com condições), proibido
- [ ] Aplicável a construção e abertura de empresa
- [ ] Tipos de uso configuráveis pelo município
- [ ] Condições de permissibilidade em texto livre

**Dependências:** `depends_on: [REQ-05-001]`

---

#### REQ-05-003 — Consulta de Zoneamento por Imóvel
**Derives:** BIZ-05-004

**Descrição:** Ao consultar um imóvel, o sistema exibe automaticamente a zona e as regras aplicáveis.

**Use Case:**
- **Ator:** Servidor Público / Cidadão
- **Pré-condições:** Imóvel existente; zoneamento configurado.
- **Fluxo principal:**
  1. Usuário seleciona imóvel no mapa.
  2. Sistema identifica a zona que contém o polígono do imóvel.
  3. Exibe no painel cadastral: código/nome da zona, parâmetros urbanísticos e regras de permissibilidade.
- **Fluxos alternativos:**
  - 2a. Imóvel em interseção de duas zonas → exibir ambas com indicação de sobreposição.
  - 2b. Imóvel sem zona → mensagem "Zoneamento não definido para este imóvel."
- **Pós-condições:** Informação de zoneamento exibida.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Zona identificada automaticamente pela posição do polígono
- [ ] Parâmetros e permissibilidade exibidos na consulta do imóvel
- [ ] Funciona tanto no módulo servidor quanto no Portal do Cidadão (dados públicos)

**Dependências:** `depends_on: [REQ-05-001, REQ-04-001]`

---

## 2. Parcelamento de Solo

---

#### REQ-05-004 — Desmembramento de Lotes
**Derives:** BIZ-05-006, BIZ-05-008, BIZ-05-009, BIZ-05-010

**Descrição:** Subdivisão de um lote em dois ou mais novos lotes, com validação de dimensões mínimas, geração de novas inscrições, atualização de polígonos e registro de histórico.

**Use Case:**
- **Ator:** Servidor Público (Urbanismo/Planejamento)
- **Pré-condições:** Lote existente com polígono no mapa; zona com dimensões mínimas definidas.
- **Fluxo principal:**
  1. Servidor seleciona lote no mapa.
  2. Ativa ferramenta "Desmembrar".
  3. Desenha linha(s) de divisão sobre o polígono.
  4. Sistema calcula áreas resultantes.
  5. Sistema valida: cada lote resultante ≥ área mínima da zona (padrão Lei 6.766: 125m², testada 5m).
  6. Servidor confirma. Sistema gera novas inscrições imobiliárias.
  7. Polígonos atualizados no mapa.
  8. Histórico registrado: lote de origem, lotes resultantes, data, servidor.
- **Fluxos alternativos:**
  - 5a. Área resultante < mínimo legal → rejeita operação: "Lote resultante ([X] m²) inferior ao mínimo da zona ([Y] m²). Operação não permitida."
  - 6a. Servidor cancela → nenhuma alteração persistida.
- **Pós-condições:** Lotes novos criados; lote original inativado; histórico registrado.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Divisão visual sobre o polígono com cálculo automático de áreas
- [ ] Validação de área mínima conforme zona (configurável por município)
- [ ] Padrão da Lei 6.766: mínimo 125m², testada 5m (sobrescrevível)
- [ ] Geração automática de novas inscrições imobiliárias
- [ ] Atualização automática dos polígonos no mapa
- [ ] Histórico rastreável: origem → resultantes

**Dependências:** `depends_on: [REQ-04-001, REQ-05-001, REQ-02-006]`

---

#### REQ-05-005 — Remembramento de Lotes
**Derives:** BIZ-05-007, BIZ-05-008, BIZ-05-009, BIZ-05-010

**Descrição:** Unificação de dois ou mais lotes contíguos em um único lote com nova inscrição, atualização de polígono e histórico.

**Use Case:**
- **Ator:** Servidor Público (Urbanismo/Planejamento)
- **Pré-condições:** Dois ou mais lotes contíguos com polígonos no mapa.
- **Fluxo principal:**
  1. Servidor seleciona dois ou mais lotes contíguos.
  2. Ativa ferramenta "Remembrar".
  3. Sistema valida contiguidade (polígonos compartilham pelo menos um lado).
  4. Sistema calcula área do lote unificado.
  5. Servidor confirma. Sistema gera nova inscrição imobiliária.
  6. Polígono unificado substitui os anteriores no mapa.
  7. Histórico registrado: lotes de origem, lote resultante, data, servidor.
- **Fluxos alternativos:**
  - 3a. Lotes não contíguos → erro "Os lotes selecionados não são contíguos."
  - 1a. Apenas um lote selecionado → erro "Selecione ao menos dois lotes para remembramento."
- **Pós-condições:** Lote unificado criado; lotes originais inativados; histórico registrado.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Validação de contiguidade entre lotes
- [ ] Cálculo automático de área unificada
- [ ] Geração de nova inscrição imobiliária
- [ ] Polígono unificado no mapa
- [ ] Histórico rastreável: origens → resultante

**Dependências:** `depends_on: [REQ-04-001, REQ-05-001, REQ-02-006]`

---

## 3. Planta Genérica de Valores

---

#### REQ-05-006 — Planta Genérica de Valores
**Derives:** BIZ-05-011, BIZ-05-012, BIZ-05-013, BIZ-05-014

**Descrição:** Atribuição de valor do m² por logradouro, imóvel individual e bairro, com comparativos para análise.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Logradouros e imóveis cadastrados.
- **Fluxo principal:**
  1. Servidor acessa módulo PGV.
  2. Define valores do m² por logradouro (tabela editável).
  3. Opcionalmente, define valor individualizado por imóvel.
  4. Sistema calcula valor por bairro (média dos logradouros do bairro).
  5. Servidor acessa painel de comparativos: ranking de logradouros, bairros e imóveis por valor de m².
- **Fluxos alternativos:**
  - 2a. Importação em massa de valores via planilha (Excel/CSV).
  - 5a. Comparativo entre PGV vigente e anterior (se houver).
- **Pós-condições:** PGV atualizada; comparativos disponíveis.
- **Exceções:** Valor de m² ≤ 0 → erro "Valor do m² deve ser positivo."

**Critérios de aceite:**
- [ ] Valor do m² por logradouro
- [ ] Valor do m² por imóvel individual (quando aplicável)
- [ ] Valor do m² por bairro (agregado)
- [ ] Comparativos entre logradouros, bairros e imóveis
- [ ] Importação em massa via planilha
- [ ] Comparativo com PGV anterior

**Dependências:** `depends_on: [REQ-04-001, REQ-04-009]`
