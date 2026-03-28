---
document_type: prd
domain: cadastro_territorial
derives_from: brd/04_domain_cadastro_territorial_brd.md
file: prd/04_domain_cadastro_territorial_prd.md
requirements:
  - id: REQ-04-001
    title: "Manutenção do Cadastro Imobiliário"
    depends_on: [REQ-03-001, REQ-03-003]
  - id: REQ-04-002
    title: "Restrições Ambientais em Imóveis"
    depends_on: [REQ-04-001, REQ-02-008]
  - id: REQ-04-003
    title: "Viabilidade de Construção"
    depends_on: [REQ-04-001]
  - id: REQ-04-004
    title: "Gestão de Anexos de Imóvel"
    depends_on: [REQ-04-001]
  - id: REQ-04-005
    title: "Coordenadas e Importação em Massa de Lotes"
    depends_on: [REQ-04-001, REQ-03-007]
  - id: REQ-04-006
    title: "Consulta Multicritério de Imóveis"
    depends_on: [REQ-04-001]
  - id: REQ-04-007
    title: "Extração por Raio e Exportação"
    depends_on: [REQ-04-006, REQ-01-012]
  - id: REQ-04-008
    title: "Cadastro Mobiliário"
    depends_on: [REQ-04-001]
  - id: REQ-04-009
    title: "Manutenção de Logradouros"
    depends_on: [REQ-03-001]
  - id: REQ-04-010
    title: "Pontos de Interesse"
    depends_on: [REQ-03-001, REQ-03-005]
---

# PRD-04 — Domínio: Cadastro Territorial

## 1. Cadastro Imobiliário

---

#### REQ-04-001 — Manutenção do Cadastro Imobiliário
**Derives:** BIZ-04-001, BIZ-04-002, BIZ-04-008

**Descrição:** Consulta e manutenção completa do cadastro imobiliário (lotes e unidades) a partir da seleção do polígono no mapa. Cada imóvel é identificado por inscrição imobiliária e matrícula (únicos por município) e possui dados editáveis: características físicas, propriedade, CIB.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Mapa carregado com polígonos de lotes; servidor autenticado com perfil de edição cadastral.
- **Fluxo principal:**
  1. Servidor clica em polígono de lote no mapa.
  2. Sistema abre painel cadastral com dados do imóvel: inscrição, matrícula, CIB, tipo (lote/unidade), área do terreno, área construída, padrão construtivo, proprietário (CPF/CNPJ), endereço, bairro.
  3. Servidor edita campos desejados.
  4. Sistema valida os dados (inscrição única, CPF válido, áreas numéricas positivas).
  5. Servidor salva. Sistema registra auditoria.
- **Fluxos alternativos:**
  - 1a. Polígono sem imóvel associado → opção "Criar novo imóvel" com formulário completo.
  - 4a. Inscrição duplicada → erro "Inscrição imobiliária já existe neste município."
  - 4b. CPF/CNPJ inválido → erro "Documento do proprietário inválido."
- **Pós-condições:** Cadastro atualizado; log de auditoria gerado.
- **Exceções:** Falha de salvamento → dados não persistidos; mensagem de erro.

**Critérios de aceite:**
- [ ] Consulta de dados via clique no polígono
- [ ] Edição de todos os campos cadastrais listados
- [ ] Inscrição imobiliária e matrícula únicos por município
- [ ] Validação de CPF/CNPJ do proprietário
- [ ] Toda alteração gera registro de auditoria (REQ-01-019)

**Dependências:** `depends_on: [REQ-03-001, REQ-03-003]`

---

#### REQ-04-002 — Restrições Ambientais em Imóveis
**Derives:** BIZ-04-003

**Descrição:** Registro e manutenção de restrições ambientais vinculadas a imóveis: APP, área de risco, reserva legal.

**Use Case:**
- **Ator:** Servidor Público (Meio Ambiente)
- **Pré-condições:** Imóvel existente no cadastro.
- **Fluxo principal:**
  1. Servidor seleciona imóvel no mapa.
  2. Acessa aba "Restrições Ambientais".
  3. Adiciona restrição: tipo (APP/risco/reserva legal/outro), descrição, fundamentação legal.
  4. Salva.
- **Fluxos alternativos:**
  - 3a. Remoção de restrição → registra justificativa e data.
- **Pós-condições:** Restrição vinculada ao imóvel; visível nas consultas e no mapa.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Tipos de restrição: APP, risco, reserva legal, personalizáveis
- [ ] Restrição visível na consulta cadastral e como indicador no mapa
- [ ] Histórico de adição/remoção de restrições

**Dependências:** `depends_on: [REQ-04-001, REQ-02-008]`

---

#### REQ-04-003 — Viabilidade de Construção
**Derives:** BIZ-04-004

**Descrição:** Consulta e registro de informações de viabilidade de construção do imóvel.

**Use Case:**
- **Ator:** Servidor Público (Urbanismo/Planejamento)
- **Pré-condições:** Imóvel existente; zoneamento configurado (REQ-05-001).
- **Fluxo principal:**
  1. Servidor seleciona imóvel.
  2. Acessa aba "Viabilidade".
  3. Sistema exibe automaticamente: zona do imóvel, regras de permissibilidade, restrições ambientais.
  4. Servidor registra parecer de viabilidade.
- **Pós-condições:** Informação de viabilidade registrada e consultável.
- **Exceções:** Imóvel sem zona associada → alerta "Imóvel não possui zoneamento definido."

**Critérios de aceite:**
- [ ] Informações de zona e permissibilidade carregadas automaticamente
- [ ] Restrições ambientais exibidas junto à viabilidade
- [ ] Parecer de viabilidade registrado com data e servidor

**Dependências:** `depends_on: [REQ-04-001]`

---

#### REQ-04-004 — Gestão de Anexos de Imóvel
**Derives:** BIZ-04-005

**Descrição:** Anexar documentos e arquivos a um imóvel (plantas, fotos, laudos).

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Imóvel existente.
- **Fluxo principal:**
  1. Servidor seleciona imóvel e acessa aba "Anexos".
  2. Clica em "Adicionar anexo".
  3. Faz upload do arquivo, classifica por tipo (planta/foto/laudo/outro) e adiciona descrição.
  4. Sistema armazena o arquivo vinculado ao imóvel.
- **Fluxos alternativos:**
  - 3a. Arquivo excede tamanho máximo → erro "Arquivo excede o limite de [X] MB."
- **Pós-condições:** Arquivo anexado e listado no cadastro do imóvel.
- **Exceções:** Formato não aceito → erro com lista de formatos permitidos.

**Critérios de aceite:**
- [ ] Tipos: planta, foto, laudo, outro
- [ ] Formatos aceitos: PDF, JPG, PNG, DWG, DXF
- [ ] Lista de anexos com nome, tipo, data de upload e tamanho
- [ ] Download e visualização inline (para imagens e PDF)

**Dependências:** `depends_on: [REQ-04-001]`

---

#### REQ-04-005 — Coordenadas e Importação em Massa de Lotes
**Derives:** BIZ-04-006, BIZ-04-007

**Descrição:** Acesso às coordenadas geográficas dos vértices do lote com exportação, e importação em massa de coordenadas de lotes via arquivos estruturados.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Lotes com polígonos no mapa.
- **Fluxo principal (exportação):**
  1. Servidor seleciona lote.
  2. Acessa aba "Coordenadas".
  3. Sistema exibe lista de vértices (Lat/Long e UTM).
  4. Servidor exporta coordenadas (CSV ou TXT).
- **Fluxo principal (importação em massa):**
  1. Servidor acessa "Importar lotes em massa".
  2. Faz upload de TXT com coordenadas e inscrições.
  3. Sistema associa coordenadas a imóveis existentes por inscrição imobiliária.
  4. Lotes sem correspondência listados para revisão.
- **Pós-condições:** Coordenadas exportadas ou lotes importados/vinculados.
- **Exceções:** Formato TXT incorreto → erro com especificação esperada.

**Critérios de aceite:**
- [ ] Exibição de vértices em Lat/Long e UTM
- [ ] Exportação de coordenadas em CSV e TXT
- [ ] Importação em massa vincula por inscrição imobiliária
- [ ] Relatório: importados com sucesso, sem correspondência, com erro

**Dependências:** `depends_on: [REQ-04-001, REQ-03-007]`

---

## 2. Consulta e Extração

---

#### REQ-04-006 — Consulta Multicritério de Imóveis
**Derives:** BIZ-04-009

**Descrição:** Consulta de imóveis por múltiplos critérios: CPF/CNPJ, endereço, cadastro, matrícula ou inscrição imobiliária.

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Base cadastral disponível.
- **Fluxo principal:**
  1. Servidor abre painel de busca.
  2. Informa critério de busca (campo + valor).
  3. Sistema retorna lista de imóveis correspondentes.
  4. Servidor clica em resultado; mapa centraliza no polígono do imóvel.
- **Fluxos alternativos:**
  - 2a. Busca sem resultados → mensagem "Nenhum imóvel encontrado para o critério informado."
  - 2b. Imóvel encontrado mas sem polígono → exibe dados cadastrais sem centralizar mapa.
- **Pós-condições:** Imóvel localizado e selecionado.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Critérios: CPF/CNPJ, endereço (parcial), número de cadastro, matrícula, inscrição imobiliária
- [ ] Busca parcial por endereço (autocomplete)
- [ ] Resultado com link para o imóvel no mapa
- [ ] Imóveis sem polígono exibem dados sem posição no mapa

**Dependências:** `depends_on: [REQ-04-001]`

---

#### REQ-04-007 — Extração por Raio e Exportação
**Derives:** BIZ-04-010, BIZ-04-011, BIZ-04-012

**Descrição:** Extração de imóveis individual ou por raio geográfico, com exportação em CSV, TXT, Excel e PDF, respeitando regras de mascaramento de dados sensíveis.

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Base cadastral disponível.
- **Fluxo principal:**
  1. Servidor clica em ponto no mapa e define raio (em metros).
  2. Sistema lista todos os imóveis dentro do raio.
  3. Servidor seleciona formato de exportação (CSV, TXT, Excel ou PDF).
  4. Sistema aplica regras de mascaramento conforme perfil (REQ-01-012).
  5. Arquivo gerado e disponibilizado para download.
- **Fluxos alternativos:**
  - 1a. Extração individual: servidor seleciona imóvel específico para exportação.
  - 2a. Nenhum imóvel no raio → mensagem informativa.
- **Pós-condições:** Arquivo exportado com dados respeitando privacidade.
- **Exceções:** Raio muito grande (>5000m) → alerta "Raio muito amplo. Reduza para obter resultados mais precisos."

**Critérios de aceite:**
- [ ] Extração por raio com input visual (círculo no mapa)
- [ ] Formatos: CSV, TXT, Excel, PDF
- [ ] Mascaramento de CPF/dados sensíveis conforme perfil do usuário
- [ ] Extração individual e por raio
- [ ] Limite de raio configurável para evitar consultas excessivas

**Dependências:** `depends_on: [REQ-04-006, REQ-01-012]`

---

## 3. Cadastro Mobiliário

---

#### REQ-04-008 — Cadastro Mobiliário
**Derives:** BIZ-04-013, BIZ-04-014, BIZ-04-015

**Descrição:** Consulta e manutenção do cadastro de empresas vinculadas a imóveis via mapa, com filtro por atividade econômica.

**Use Case:**
- **Ator:** Servidor Público (Cadastro/Tributário)
- **Pré-condições:** Imóvel existente.
- **Fluxo principal:**
  1. Servidor seleciona imóvel comercial no mapa.
  2. Acessa aba "Empresas".
  3. Visualiza empresas vinculadas (razão social, CNPJ, CNAE).
  4. Pode adicionar nova empresa ou editar existente.
  5. Pode filtrar empresas por atividade econômica (CNAE) em busca global.
- **Pós-condições:** Cadastro mobiliário atualizado; empresa vinculada ao imóvel.
- **Exceções:** CNPJ duplicado → erro "Empresa com este CNPJ já cadastrada."

**Critérios de aceite:**
- [ ] Empresa vinculada a imóvel/endereço no mapa
- [ ] Dados: razão social, CNPJ, CNAE, endereço
- [ ] Filtro e busca por CNAE (código ou descrição)
- [ ] Extração de empresas por atividade econômica

**Dependências:** `depends_on: [REQ-04-001]`

---

## 4. Logradouros e POIs

---

#### REQ-04-009 — Manutenção de Logradouros
**Derives:** BIZ-04-016, BIZ-04-017

**Descrição:** Acesso e manutenção de dados de logradouros (nome, tipo, CEP, bairro) e características da via (pavimentação, largura, iluminação, rede de água/esgoto).

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Servidor autenticado.
- **Fluxo principal:**
  1. Servidor busca logradouro por nome ou CEP.
  2. Sistema exibe dados do logradouro.
  3. Servidor edita campos: nome, tipo (rua/avenida/etc.), CEP, bairro, pavimentação, largura, infraestrutura.
  4. Salva alterações.
- **Pós-condições:** Dados de logradouro atualizados.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Campos: nome, tipo, CEP, bairro, pavimentação, largura, iluminação, rede água/esgoto
- [ ] Busca por nome (parcial) e CEP
- [ ] Logradouro vinculado a imóveis do cadastro

**Dependências:** `depends_on: [REQ-03-001]`

---

#### REQ-04-010 — Pontos de Interesse
**Derives:** BIZ-04-018, BIZ-04-019

**Descrição:** Cadastro de POIs no mapa (hospitais, creches, escolas, pontes, praças) com nome, tipo/categoria, coordenadas e descrição.

**Use Case:**
- **Ator:** Servidor Público
- **Pré-condições:** Mapa carregado.
- **Fluxo principal:**
  1. Servidor ativa ferramenta "Novo ponto de interesse".
  2. Clica na posição do mapa.
  3. Preenche: nome, categoria (Educação/Saúde/Infraestrutura/etc.), descrição.
  4. Sistema registra coordenadas automaticamente e salva o POI.
  5. POI exibido na camada de equipamentos públicos.
- **Pós-condições:** POI criado e visível no mapa.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Campos: nome, tipo/categoria, coordenadas (auto), descrição
- [ ] Categorias configuráveis pelo administrador
- [ ] POIs exibidos em camada específica com ícones por categoria
- [ ] Busca de POIs por nome e categoria

**Dependências:** `depends_on: [REQ-03-001, REQ-03-005]`
