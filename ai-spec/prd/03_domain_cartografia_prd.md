---
document_type: prd
domain: cartografia
derives_from: brd/03_domain_cartografia_brd.md
file: prd/03_domain_cartografia_prd.md
requirements:
  - id: REQ-03-001
    title: "Mapa Interativo Web"
    depends_on: [REQ-02-004]
  - id: REQ-03-002
    title: "Provedores de Mapa Plug-and-Play"
    depends_on: [REQ-03-001, REQ-01-018]
  - id: REQ-03-003
    title: "Desenho e Edição de Polígonos"
    depends_on: [REQ-03-001]
  - id: REQ-03-004
    title: "Associação de Polígonos a Entidades"
    depends_on: [REQ-03-003, REQ-03-005]
  - id: REQ-03-005
    title: "Gestão de Camadas"
    depends_on: [REQ-03-001, REQ-01-001]
  - id: REQ-03-006
    title: "Ferramentas de Medição"
    depends_on: [REQ-03-001]
  - id: REQ-03-007
    title: "Sistema de Coordenadas"
    depends_on: [REQ-03-001, REQ-02-004]
  - id: REQ-03-008
    title: "Importação de Arquivos Geoespaciais"
    depends_on: [REQ-03-001, REQ-03-005, REQ-03-007]
  - id: REQ-03-009
    title: "Visualização de Imagens de Satélite"
    depends_on: [REQ-03-001, REQ-02-013]
  - id: REQ-03-010
    title: "Integração Street View"
    depends_on: [REQ-03-001]
  - id: REQ-03-011
    title: "Exportação e Impressão de Mapa"
    depends_on: [REQ-03-001]
---

# PRD-03 — Domínio: Cartografia

## 1. Motor de Mapa

---

#### REQ-03-001 — Mapa Interativo Web
**Derives:** BIZ-03-001, BIZ-03-002, BIZ-03-003

**Descrição:** Mapa interativo web com zoom, pan, navegação fluida, exibição de múltiplas camadas sobrepostas com controle de visibilidade individual e seleção de elementos com exibição de informações.

**Use Case:**
- **Ator:** Servidor Público / Cidadão
- **Pré-condições:** Usuário acessa o sistema via navegador.
- **Fluxo principal:**
  1. Sistema carrega o mapa com a base cartográfica do município e camadas visíveis conforme perfil.
  2. Usuário navega pelo mapa (zoom, pan).
  3. Usuário ativa/desativa camadas no painel de controle de camadas.
  4. Usuário clica em um polígono ou ponto no mapa.
  5. Sistema exibe popup ou painel lateral com informações da entidade associada.
- **Fluxos alternativos:**
  - 3a. Nenhuma camada habilitada → mapa exibe apenas a base cartográfica do provedor.
  - 4a. Clique em área sem elementos → nenhuma informação exibida.
- **Pós-condições:** Mapa renderizado com camadas e elemento selecionado destacado.
- **Exceções:** Falha de carregamento de tiles → exibir mensagem "Erro ao carregar mapa. Tente novamente."

**Critérios de aceite:**
- [ ] Mapa renderiza em menos de 3 segundos na carga inicial
- [ ] Zoom e pan respondem de forma fluida (sem lag perceptível)
- [ ] Múltiplas camadas exibidas simultaneamente com controle individual
- [ ] Clique em polígono exibe informações associadas
- [ ] Funciona em Chrome, Firefox, Edge e Safari (últimas 2 versões)

**Dependências:** `depends_on: [REQ-02-004]`

---

#### REQ-03-002 — Provedores de Mapa Plug-and-Play
**Derives:** BIZ-03-004

**Descrição:** Arquitetura plug-and-play de provedores de mapa base, com suporte mínimo a OpenStreetMap/Leaflet (padrão, gratuito, self-hosted), QGIS, ArcGIS, Google Maps e Mapbox. Ativação por tenant via API Keys.

**Use Case:**
- **Ator:** Administrador do Sistema
- **Pré-condições:** Administrador autenticado; API Key do provedor disponível.
- **Fluxo principal:**
  1. Administrador acessa configurações de mapa do tenant.
  2. Visualiza o provedor ativo (padrão: OpenStreetMap/Leaflet).
  3. Seleciona provedor alternativo (ex.: Google Maps).
  4. Informa API Key do provedor.
  5. Sistema valida a chave (requisição de teste).
  6. Provedor ativado; mapa recarrega com novo provedor.
- **Fluxos alternativos:**
  - 5a. API Key inválida → mensagem "Chave de acesso inválida. Verifique com o provedor."
  - 2a. Administrador mantém o padrão → nenhuma configuração necessária.
- **Pós-condições:** Mapa do tenant utiliza o provedor selecionado.
- **Exceções:** Provedor indisponível → fallback automático para OpenStreetMap/Leaflet.

**Critérios de aceite:**
- [ ] OpenStreetMap/Leaflet funciona sem API Key (padrão out-of-the-box)
- [ ] Provedores suportados: OpenStreetMap/Leaflet, QGIS Server, ArcGIS, Google Maps, Mapbox
- [ ] API Key configurável por tenant
- [ ] Validação de API Key com feedback imediato
- [ ] Fallback para OpenStreetMap/Leaflet se provedor configurado ficar indisponível
- [ ] Troca de provedor não requer redeployment

**Dependências:** `depends_on: [REQ-03-001, REQ-01-018]`

---

## 2. Desenho e Edição

---

#### REQ-03-003 — Desenho e Edição de Polígonos
**Derives:** BIZ-03-005, BIZ-03-006

**Descrição:** Ferramentas de desenho preciso de polígonos com funcionalidades CAD (snap, alinhamento, vértices editáveis), incluindo edição de polígonos existentes (mover vértices, adicionar/remover pontos, redimensionar, rotacionar).

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Usuário autenticado com perfil de edição cartográfica; mapa carregado.
- **Fluxo principal:**
  1. Servidor ativa ferramenta de desenho na toolbar.
  2. Clica nos vértices sobre o mapa para definir os pontos do polígono.
  3. Sistema exibe snap guides e alinhamento com elementos vizinhos.
  4. Servidor finaliza o polígono (clique duplo ou botão "Fechar").
  5. Sistema calcula a área e exibe.
  6. Servidor confirma o polígono.
- **Fluxos alternativos:**
  - 1a. Edição de polígono existente: servidor seleciona polígono → sistema exibe vértices editáveis → servidor arrasta, adiciona ou remove vértices.
  - 4a. Polígono não fecha corretamente (autointerseção) → erro "Polígono inválido: geometria contém autointerseção."
- **Pós-condições:** Polígono criado/editado e salvo com geometria válida.
- **Exceções:** Geometria inválida → bloquear salvamento e indicar o ponto de erro visualmente.

**Critérios de aceite:**
- [ ] Snap a vértices de polígonos vizinhos (tolerância configurável)
- [ ] Guias de alinhamento horizontal/vertical
- [ ] Edição de vértices: mover, adicionar, remover
- [ ] Redimensionar e rotacionar polígono existente
- [ ] Validação de geometria (sem autointerseção) antes de salvar
- [ ] Cálculo automático de área ao criar/editar
- [ ] Desfazer/refazer (Ctrl+Z / Ctrl+Y)

**Dependências:** `depends_on: [REQ-03-001]`

---

#### REQ-03-004 — Associação de Polígonos a Entidades
**Derives:** BIZ-03-007

**Descrição:** Cada polígono deve poder ser associado a uma ou mais camadas e a entidades de outros domínios (imóvel, zona, área de fiscalização).

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Polígono criado; camada de destino existente.
- **Fluxo principal:**
  1. Servidor seleciona polígono recém-criado ou existente.
  2. No painel de propriedades, define a camada de destino.
  3. Associa o polígono a uma entidade (ex.: inscrição imobiliária, código de zona).
  4. Sistema valida a associação e salva.
- **Fluxos alternativos:**
  - 3a. Polígono associado a múltiplas camadas simultaneamente.
  - 3b. Associação a entidade inexistente → erro "Entidade não encontrada."
- **Pós-condições:** Polígono vinculado à camada e entidade; visível na camada correspondente.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Polígono pode pertencer a uma ou mais camadas
- [ ] Polígono pode ser vinculado a entidades de domínios: Cadastro, Zoneamento, Fiscalização
- [ ] Remoção de associação não exclui o polígono nem a entidade

**Dependências:** `depends_on: [REQ-03-003, REQ-03-005]`

---

## 3. Camadas

---

#### REQ-03-005 — Gestão de Camadas
**Derives:** BIZ-03-008, BIZ-03-009, BIZ-03-010, BIZ-03-011

**Descrição:** Criação e gestão de camadas com três níveis de visibilidade (pública, privada do usuário, restrita a perfis/setores), metadados configuráveis, controle de quais camadas são visíveis no Portal do Cidadão e ordenação (z-index).

**Use Case:**
- **Ator:** Administrador do Sistema
- **Pré-condições:** Administrador autenticado com perfil de gestão de camadas.
- **Fluxo principal:**
  1. Administrador acessa módulo de gestão de camadas.
  2. Cria nova camada definindo: nome, descrição, visibilidade (pública/privada/restrita), cor/estilo, zoom mínimo/máximo.
  3. Se visibilidade = restrita, seleciona perfis/setores autorizados.
  4. Define z-index (ordem de sobreposição).
  5. Marca se a camada deve ser visível no Portal do Cidadão.
  6. Salva a camada.
- **Fluxos alternativos:**
  - 2a. Visibilidade = privada → camada visível apenas para o usuário que a criou.
  - 4a. Reordenação de camadas via drag-and-drop no painel.
- **Pós-condições:** Camada criada e visível conforme permissões.
- **Exceções:** Nome de camada duplicado no mesmo tenant → erro "Já existe uma camada com este nome."

**Critérios de aceite:**
- [ ] Três níveis de visibilidade: pública, privada do usuário, restrita a perfis/setores
- [ ] Metadados configuráveis: nome, descrição, cor/estilo, zoom mín/máx
- [ ] Controle de visibilidade no Portal do Cidadão por camada
- [ ] Ordenação (z-index) com interface drag-and-drop
- [ ] Servidor vê camadas conforme seu perfil; cidadão vê apenas camadas públicas habilitadas

**Dependências:** `depends_on: [REQ-03-001, REQ-01-001]`

---

## 4. Medição e Coordenadas

---

#### REQ-03-006 — Ferramentas de Medição
**Derives:** BIZ-03-012, BIZ-03-013

**Descrição:** Medição de distância entre pontos (em metros) e cálculo de área de polígonos (em m²).

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Mapa carregado.
- **Fluxo principal:**
  1. Servidor ativa ferramenta de medição (distância ou área).
  2. Para distância: clica dois ou mais pontos; sistema exibe segmentos com distância parcial e total.
  3. Para área: desenha polígono temporário; sistema exibe a área em m².
  4. Servidor encerra a medição.
- **Pós-condições:** Medição exibida; dados não persistidos (ferramenta auxiliar).
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Distância exibida em metros com 2 casas decimais
- [ ] Área exibida em m² com 2 casas decimais
- [ ] Medição de distância suporta múltiplos segmentos com total acumulado
- [ ] Medição não persiste após desativação da ferramenta

**Dependências:** `depends_on: [REQ-03-001]`

---

#### REQ-03-007 — Sistema de Coordenadas
**Derives:** BIZ-03-014, BIZ-03-015, BIZ-03-016, BIZ-03-017

**Descrição:** Suporte a UTM e Lat/Long, datum SIRGAS 2000, exibição em tempo real das coordenadas do cursor com alternância de sistema, e navegação direta por coordenadas.

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Mapa carregado.
- **Fluxo principal:**
  1. Sistema exibe coordenadas do cursor no rodapé do mapa (formato padrão: Lat/Long).
  2. Servidor alterna para UTM via toggle.
  3. Servidor clica em "Ir para coordenada".
  4. Informa coordenadas (UTM ou Lat/Long).
  5. Sistema centraliza o mapa na posição informada.
- **Fluxos alternativos:**
  - 4a. Coordenada fora dos limites do município → alerta "Coordenada fora da área do município."
  - 4b. Formato inválido → erro "Formato de coordenada inválido. Esperado: [-XX.XXXXXX, -XX.XXXXXX]"
- **Pós-condições:** Mapa posicionado na coordenada informada.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Coordenadas do cursor exibidas em tempo real
- [ ] Alternância UTM ↔ Lat/Long via toggle
- [ ] Navegação por coordenada (input manual) em ambos os formatos
- [ ] Datum base: SIRGAS 2000
- [ ] Validação de formato de coordenada com mensagem de erro clara

| Formato | Exemplo válido | Exemplo inválido |
|---------|---------------|-----------------|
| Lat/Long | -26.916351, -49.066116 | 26.916351 (falta longitude) |
| UTM | 697532.45 E, 7019845.12 N, Fuso 22S | 697532.45 (falta N e fuso) |

**Dependências:** `depends_on: [REQ-03-001, REQ-02-004]`

---

## 5. Importação e Integração

---

#### REQ-03-008 — Importação de Arquivos Geoespaciais
**Derives:** BIZ-03-018, BIZ-03-019, BIZ-03-020, BIZ-03-021

**Descrição:** Importação de KMZ, KML, Shapefile e TXT com coordenadas, com validação de datum, conversão para SIRGAS 2000, geração de camadas/polígonos editáveis e importação em massa de lotes.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Arquivo geoespacial disponível; camada de destino definida ou a ser criada.
- **Fluxo principal:**
  1. Servidor acessa "Importar arquivo geoespacial".
  2. Faz upload do arquivo (KMZ, KML, SHP ou TXT).
  3. Sistema detecta formato e datum de origem.
  4. Se datum ≠ SIRGAS 2000, converte automaticamente e informa o usuário.
  5. Sistema exibe preview dos elementos no mapa.
  6. Servidor seleciona camada de destino (existente ou nova).
  7. Confirma importação.
  8. Polígonos/pontos criados como elementos editáveis na camada.
- **Fluxos alternativos:**
  - 2a. Formato não suportado → erro "Formato não suportado. Formatos aceitos: KMZ, KML, SHP, TXT."
  - 3a. Datum não identificável → solicitar seleção manual do datum.
  - 5a. Elementos com geometria inválida → listados em relatório de erros; importação parcial permitida.
- **Pós-condições:** Elementos importados e editáveis no mapa.
- **Exceções:** Arquivo corrompido → erro "Falha ao processar arquivo. Verifique a integridade."

**Critérios de aceite:**
- [ ] Formatos suportados: KMZ, KML, Shapefile (.shp + .dbf + .shx + .prj), TXT com coordenadas
- [ ] Detecção automática de datum com conversão para SIRGAS 2000
- [ ] Preview antes da importação
- [ ] Importação gera polígonos editáveis
- [ ] Importação em massa de lotes via TXT estruturado
- [ ] Relatório pós-importação: total, importados, convertidos, rejeitados

**Dependências:** `depends_on: [REQ-03-001, REQ-03-005, REQ-03-007]`

---

#### REQ-03-009 — Visualização de Imagens de Satélite
**Derives:** BIZ-03-022, BIZ-03-023, BIZ-03-024

**Descrição:** Exibição de imagens de satélite como camada de fundo, com suporte a múltiplos períodos históricos para análise temporal. Provedor externo.

**Use Case:**
- **Ator:** Servidor Público (Meio Ambiente)
- **Pré-condições:** Módulo de satélite habilitado no tenant; imagens disponíveis.
- **Fluxo principal:**
  1. Servidor ativa camada de imagem de satélite.
  2. Sistema carrega imagem mais recente disponível como fundo.
  3. Servidor acessa seletor de período.
  4. Seleciona duas datas para comparação.
  5. Sistema exibe side-by-side ou slider para comparação temporal.
- **Fluxos alternativos:**
  - 1a. Módulo desabilitado → opção não disponível no menu.
  - 3a. Apenas uma data disponível → comparação não disponível.
- **Pós-condições:** Imagens exibidas para análise visual.
- **Exceções:** Imagem não disponível para a área → mensagem "Sem cobertura de satélite para esta região."

**Critérios de aceite:**
- [ ] Imagem de satélite renderiza como camada de fundo
- [ ] Seletor de período lista datas disponíveis
- [ ] Comparação temporal com side-by-side ou slider
- [ ] Funciona com imagens fornecidas via API ou arquivo importado
- [ ] Módulo pode ser desabilitado por tenant sem erros

**Dependências:** `depends_on: [REQ-03-001, REQ-02-013]`

---

#### REQ-03-010 — Integração Street View
**Derives:** BIZ-03-025, BIZ-03-026

**Descrição:** Visualização Street View (Google) a partir de ponto selecionado, disponível para servidor e cidadão.

**Use Case:**
- **Ator:** Servidor Público / Cidadão
- **Pré-condições:** API Key do Google Street View configurada no tenant.
- **Fluxo principal:**
  1. Usuário clica com botão direito em ponto do mapa (ou usa botão "Street View").
  2. Sistema abre painel com imagem Street View da posição.
  3. Usuário navega pela imagem 360° dentro do painel.
- **Fluxos alternativos:**
  - 2a. Sem cobertura Street View → mensagem "Street View não disponível para esta localização."
  - 1a. API Key não configurada → opção oculta no menu.
- **Pós-condições:** Imagem Street View exibida.
- **Exceções:** Quota de API excedida → erro "Limite de requisições Street View atingido."

**Critérios de aceite:**
- [ ] Street View abre a partir de qualquer ponto com cobertura
- [ ] Disponível no módulo servidor e no Portal do Cidadão
- [ ] Mensagem adequada quando sem cobertura
- [ ] API Key configurável por tenant

**Dependências:** `depends_on: [REQ-03-001]`

---

## 6. Exportação

---

#### REQ-03-011 — Exportação e Impressão de Mapa
**Derives:** BIZ-03-027

**Descrição:** Impressão e exportação de imagem do mapa (recorte da área visível ou imóvel específico) em PDF ou imagem.

**Use Case:**
- **Ator:** Servidor Público / Cidadão
- **Pré-condições:** Mapa carregado com área de interesse visível.
- **Fluxo principal:**
  1. Usuário clica em "Exportar/Imprimir".
  2. Seleciona modo: área visível ou imóvel selecionado.
  3. Seleciona formato: PDF ou PNG.
  4. Sistema gera o arquivo com mapa, camadas ativas e legenda.
  5. Arquivo disponibilizado para download.
- **Pós-condições:** Arquivo exportado.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Formatos: PDF e PNG
- [ ] Modos: recorte da área visível ou imóvel específico (com destaque)
- [ ] Inclui camadas ativas e legenda
- [ ] PDF em formato adequado para impressão A4

**Dependências:** `depends_on: [REQ-03-001]`
