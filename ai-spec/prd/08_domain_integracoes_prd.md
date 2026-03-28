---
document_type: prd
domain: integracoes
derives_from: brd/08_domain_integracoes_brd.md
file: prd/08_domain_integracoes_prd.md
requirements:
  - id: REQ-08-001
    title: "Integração Bidirecional com ERP Municipal"
    depends_on: [REQ-02-015, REQ-04-001]
  - id: REQ-08-002
    title: "Resolução de Conflitos GEO-ERP"
    depends_on: [REQ-08-001]
  - id: REQ-08-003
    title: "Modo Degradado de Integração"
    depends_on: [REQ-08-001]
  - id: REQ-08-004
    title: "Integração com SINTER/CIB"
    depends_on: [REQ-02-002, REQ-04-001]
  - id: REQ-08-005
    title: "Integração com Google Street View"
    depends_on: [REQ-03-001]
  - id: REQ-08-006
    title: "Importação de Imagens de Satélite"
    depends_on: [REQ-03-009, REQ-02-013]
  - id: REQ-08-007
    title: "Integração com Sistemas de Protocolo"
    depends_on: [REQ-07-010]
---

# PRD-08 — Domínio: Integrações

## 1. Integração com ERP Municipal

---

#### REQ-08-001 — Integração Bidirecional com ERP Municipal
**Derives:** BIZ-08-001, BIZ-08-002, BIZ-08-003, BIZ-08-004, BIZ-08-005

**Descrição:** Integração bidirecional via webservices (API REST) com o ERP de gestão municipal para sincronização de imóveis, pessoas e empresas. Arquitetura de adaptadores configuráveis por município.

**Use Case:**
- **Ator:** Sistema (automático) / Administrador do Sistema (configuração)
- **Pré-condições:** Adaptador do ERP configurado (URL, credenciais, mapeamento de campos).
- **Fluxo principal (GEO → ERP):**
  1. Servidor altera cadastro de imóvel no GEO.
  2. Sistema detecta alteração e aciona adaptador do ERP.
  3. Adaptador traduz os dados para o formato do ERP.
  4. Envia via webservice.
  5. ERP confirma recebimento.
- **Fluxo principal (ERP → GEO):**
  1. ERP notifica alteração (webhook) ou GEO consulta periodicamente (polling).
  2. Adaptador recebe dados e traduz para formato interno.
  3. Sistema atualiza cadastro no GEO.
- **Fluxos alternativos:**
  - 4a. ERP rejeita dados → erro logado; sinalização ao servidor.
  - 1b. Configuração de novo adaptador: administrador informa URL, credenciais, mapeamento de campos.
- **Pós-condições:** Dados sincronizados entre GEO e ERP.
- **Exceções:** ERP indisponível → ver REQ-08-003 (modo degradado).

**Critérios de aceite:**
- [ ] Sincronização bidirecional: imóveis, pessoas (proprietários), empresas
- [ ] Adaptador configurável por município: URL, credenciais, mapeamento de campos
- [ ] Novo adaptador pode ser adicionado sem alterar código core
- [ ] Suporte a webhook e polling como mecanismos de sincronização
- [ ] Log de todas as transações (sucesso e erro)

**Dependências:** `depends_on: [REQ-02-015, REQ-04-001]`

---

#### REQ-08-002 — Resolução de Conflitos GEO-ERP
**Derives:** BIZ-08-006

**Descrição:** Mecanismo de detecção e resolução de conflitos quando dados divergem entre GEO e ERP.

**Use Case:**
- **Ator:** Administrador do Sistema
- **Pré-condições:** Sincronização ativa; conflito detectado (mesmo registro alterado em ambos os lados).
- **Fluxo principal:**
  1. Sistema detecta conflito: campo X alterado no GEO e no ERP com valores diferentes.
  2. Registra o conflito na fila de resolução.
  3. Administrador acessa fila de conflitos.
  4. Para cada conflito, visualiza: registro afetado, valor GEO, valor ERP, data de cada alteração.
  5. Seleciona resolução: manter GEO, manter ERP, ou valor manual.
  6. Sistema aplica resolução e sincroniza.
- **Fluxos alternativos:**
  - 5a. Regra automática configurada (ex.: "ERP sempre prevalece para dados tributários") → conflito resolvido sem intervenção.
- **Pós-condições:** Conflito resolvido; dados consistentes.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Detecção automática de conflitos
- [ ] Fila de conflitos pendentes com detalhes (valores GEO vs ERP)
- [ ] Resolução manual: manter GEO, manter ERP, valor manual
- [ ] Resolução automática por regras configuráveis
- [ ] Histórico de conflitos resolvidos

**Dependências:** `depends_on: [REQ-08-001]`

---

#### REQ-08-003 — Modo Degradado de Integração
**Derives:** BIZ-08-007

**Descrição:** O GEO deve continuar operando com dados locais quando o ERP estiver indisponível, enfileirando alterações para sincronização posterior.

**Use Case:**
- **Ator:** Sistema (automático)
- **Pré-condições:** ERP indisponível (health check falha).
- **Fluxo principal:**
  1. Sistema detecta indisponibilidade do ERP.
  2. Altera status de integração para "Degradado".
  3. Notifica administrador.
  4. Operações no GEO continuam normalmente com dados locais.
  5. Alterações que seriam sincronizadas são enfileiradas.
  6. Quando ERP retorna, sistema processa fila automaticamente.
- **Pós-condições:** Dados sincronizados após retorno do ERP.
- **Exceções:** Fila muito grande (>1000 itens) → alerta ao administrador.

**Critérios de aceite:**
- [ ] GEO funciona integralmente sem ERP (modo degradado)
- [ ] Indicador visual de "ERP offline" para servidores
- [ ] Fila de sincronização pendente com contagem visível
- [ ] Sincronização automática ao restabelecimento
- [ ] Processamento da fila em ordem cronológica

**Dependências:** `depends_on: [REQ-08-001]`

---

## 2. Integração com SINTER

---

#### REQ-08-004 — Integração com SINTER/CIB
**Derives:** BIZ-08-008, BIZ-08-009, BIZ-08-010, BIZ-08-011

**Descrição:** Integração com SINTER (Decreto 8.764/2016) para envio de imóveis novos (obtenção de CIB) e atualizações cadastrais. CIB armazenado no cadastro.

**Use Case:**
- **Ator:** Sistema (automático, disparado por evento de cadastro)
- **Pré-condições:** Integração SINTER configurada; imóvel com dados mínimos.
- **Fluxo principal (novo imóvel):**
  1. Imóvel novo criado no GEO.
  2. Sistema monta payload conforme schema SINTER.
  3. Envia ao SINTER via API.
  4. SINTER retorna CIB.
  5. Sistema armazena CIB no cadastro do imóvel.
- **Fluxo principal (atualização):**
  1. Imóvel existente tem alteração relevante (área, proprietário, endereço).
  2. Sistema envia atualização ao SINTER.
  3. SINTER confirma recebimento.
- **Fluxos alternativos:**
  - 3a. SINTER indisponível → enfileira para retentativa (máx. 3, intervalo exponencial).
  - 3b. SINTER rejeita dados → erro logado com detalhes; servidor alertado.
- **Pós-condições:** CIB atribuído ao imóvel; atualizações propagadas.
- **Exceções:** Dados incompletos → imóvel salvo sem CIB; pendência sinalizada.

**Critérios de aceite:**
- [ ] Envio automático de imóveis novos ao SINTER
- [ ] CIB retornado e armazenado no cadastro
- [ ] Envio de atualizações cadastrais relevantes
- [ ] Retentativa automática em caso de falha (3x, backoff exponencial)
- [ ] Log de todas as transações com SINTER
- [ ] Versão da API SINTER configurável (REQ-02-016)

**Dependências:** `depends_on: [REQ-02-002, REQ-04-001]`

---

## 3. Integração com Serviços de Imagem

---

#### REQ-08-005 — Integração com Google Street View
**Derives:** BIZ-08-012, BIZ-08-013

**Descrição:** Consumo da API Google Street View para exibição de imagens de nível de rua, disponível para servidor e cidadão.

**Use Case:**
- **Ator:** Servidor Público / Cidadão
- **Pré-condições:** API Key configurada no tenant.
- **Fluxo principal:**
  1. Usuário solicita Street View de uma posição no mapa.
  2. Sistema envia coordenadas à API Google Street View.
  3. API retorna imagem panorâmica 360°.
  4. Sistema renderiza em painel integrado ao mapa.
- **Fluxos alternativos:**
  - 3a. Sem cobertura → mensagem adequada.
- **Pós-condições:** Imagem Street View exibida.
- **Exceções:** Quota excedida → "Limite de Street View atingido. Tente mais tarde."

**Critérios de aceite:**
- [ ] Visualização 360° a partir de coordenadas
- [ ] Disponível em módulo servidor e Portal do Cidadão
- [ ] API Key configurável por tenant
- [ ] Mensagem adequada quando sem cobertura ou quota excedida

**Dependências:** `depends_on: [REQ-03-001]`

---

#### REQ-08-006 — Importação de Imagens de Satélite
**Derives:** BIZ-08-014, BIZ-08-015, BIZ-08-016

**Descrição:** Importação de imagens de satélite de provedores externos (API ou arquivo), georreferenciadas em SIRGAS 2000, com modelo flexível de aquisição.

**Use Case:**
- **Ator:** Administrador do Sistema
- **Pré-condições:** Imagem disponível (arquivo ou API configurada).
- **Fluxo principal (arquivo):**
  1. Administrador acessa "Importar imagem de satélite".
  2. Faz upload de arquivo georreferenciado (GeoTIFF ou similar).
  3. Sistema valida georreferenciamento e compatibilidade com SIRGAS 2000.
  4. Imagem adicionada ao catálogo de imagens com data e área de cobertura.
- **Fluxo principal (API):**
  1. Administrador configura provedor de imagens (URL, credenciais).
  2. Sistema consome imagens via API conforme região e período solicitados.
- **Pós-condições:** Imagem disponível como camada de fundo no mapa.
- **Exceções:** Imagem sem georreferenciamento → erro "Imagem não georreferenciada. Forneça arquivo com metadados espaciais."

**Critérios de aceite:**
- [ ] Importação por arquivo (GeoTIFF) e por API
- [ ] Validação de georreferenciamento e datum
- [ ] Catálogo de imagens com data e cobertura
- [ ] Modelo flexível: API ou arquivo, provedor configurável

**Dependências:** `depends_on: [REQ-03-009, REQ-02-013]`

---

## 4. Integração com Protocolo

---

#### REQ-08-007 — Integração com Sistemas de Protocolo
**Derives:** BIZ-08-017, BIZ-08-018

**Descrição:** Encaminhamento de protocolos abertos pelo cidadão para sistemas de protocolo terceiros do município, opcional e configurável (link ou API).

**Use Case:**
- **Ator:** Sistema (automático, disparado por abertura de protocolo)
- **Pré-condições:** Integração de protocolo configurada pelo administrador.
- **Fluxo principal:**
  1. Cidadão abre protocolo via Portal (REQ-07-010).
  2. Sistema verifica se município possui integração de protocolo configurada.
  3. Se sim, encaminha dados do protocolo via API do sistema terceiro.
  4. Sistema terceiro retorna número de protocolo externo.
  5. Número exibido ao cidadão.
- **Fluxos alternativos:**
  - 2a. Sem integração configurada → protocolo registrado apenas internamente no GEO.
  - 3a. API do sistema terceiro falha → protocolo registrado no GEO com flag "pendente de encaminhamento".
- **Pós-condições:** Protocolo encaminhado ou registrado internamente.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Integração opcional e configurável por município
- [ ] Modos: link (redirecionamento) ou API (envio direto)
- [ ] Número de protocolo externo exibido ao cidadão
- [ ] Fallback para registro interno se integração falhar
- [ ] Reenvio automático de protocolos pendentes quando integração retornar

**Dependências:** `depends_on: [REQ-07-010]`
