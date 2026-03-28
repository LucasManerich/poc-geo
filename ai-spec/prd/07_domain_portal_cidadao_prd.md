---
document_type: prd
domain: portal_cidadao
derives_from: brd/07_domain_portal_cidadao_brd.md
file: prd/07_domain_portal_cidadao_prd.md
requirements:
  - id: REQ-07-001
    title: "Acesso Web ao Portal"
    depends_on: [REQ-03-001]
  - id: REQ-07-002
    title: "Acesso Anônimo"
    depends_on: [REQ-07-001, REQ-01-003]
  - id: REQ-07-003
    title: "Acesso Autenticado com Imóveis Vinculados"
    depends_on: [REQ-07-001, REQ-01-001, REQ-01-002]
  - id: REQ-07-004
    title: "Login via Gov.Br"
    depends_on: [REQ-07-003, REQ-01-018]
  - id: REQ-07-005
    title: "Captcha Cloudflare Turnstile"
    depends_on: [REQ-07-001, REQ-01-018]
  - id: REQ-07-006
    title: "Consulta de Imóveis pelo Cidadão"
    depends_on: [REQ-07-001, REQ-04-006]
  - id: REQ-07-007
    title: "Emissão de Certidões Online"
    depends_on: [REQ-07-003, REQ-04-001, REQ-02-005]
  - id: REQ-07-008
    title: "Solicitação de ITBI"
    depends_on: [REQ-07-003, REQ-04-001, REQ-05-006, REQ-02-009]
  - id: REQ-07-009
    title: "Interação com Fiscalizações"
    depends_on: [REQ-07-003, REQ-06-001]
  - id: REQ-07-010
    title: "Abertura de Protocolos"
    depends_on: [REQ-07-003]
  - id: REQ-07-011
    title: "Camadas Públicas e Impressão"
    depends_on: [REQ-07-001, REQ-03-005, REQ-03-011]
---

# PRD-07 — Domínio: Portal do Cidadão

## 1. Acesso e Autenticação

---

#### REQ-07-001 — Acesso Web ao Portal
**Derives:** BIZ-07-001

**Descrição:** O Portal do Cidadão deve ser acessível via navegador web, sem instalação de software adicional, em desktop e dispositivos móveis.

**Use Case:**
- **Ator:** Cidadão (autenticado ou anônimo)
- **Pré-condições:** Navegador web moderno com acesso à internet.
- **Fluxo principal:**
  1. Cidadão acessa a URL do Portal do município.
  2. Sistema carrega a interface com mapa e opções de serviço.
- **Pós-condições:** Portal carregado e funcional.
- **Exceções:** Navegador incompatível → mensagem "Atualize seu navegador para acessar o Portal."

**Critérios de aceite:**
- [ ] Funciona em Chrome, Firefox, Edge e Safari (últimas 2 versões)
- [ ] Responsivo: funciona em desktop, tablet e smartphone
- [ ] Carregamento inicial < 5 segundos em conexão 4G
- [ ] Sem necessidade de plugins ou instalações

**Dependências:** `depends_on: [REQ-03-001]`

---

#### REQ-07-002 — Acesso Anônimo
**Derives:** BIZ-07-002

**Descrição:** Acesso sem login para funcionalidades públicas: consulta por endereço/inscrição/matrícula, visualização de layers públicas e impressão de imagem.

**Use Case:**
- **Ator:** Cidadão (anônimo)
- **Pré-condições:** Portal carregado.
- **Fluxo principal:**
  1. Cidadão acessa Portal sem fazer login.
  2. Sistema exibe banner de privacidade (REQ-01-003).
  3. Cidadão navega pelo mapa, visualiza camadas públicas.
  4. Pesquisa imóvel por endereço, inscrição ou matrícula.
  5. Visualiza informações públicas do imóvel.
- **Fluxos alternativos:**
  - 4a. Tenta acessar funcionalidade restrita (certidão, ITBI) → redirecionado para login.
- **Pós-condições:** Cidadão acessa informações públicas sem autenticação.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Consulta por endereço, inscrição e matrícula sem login
- [ ] Visualização de layers públicas
- [ ] Impressão de imagem do imóvel
- [ ] Banner de privacidade exibido (REQ-01-003)
- [ ] Funcionalidades restritas redirecionam para login

**Dependências:** `depends_on: [REQ-07-001, REQ-01-003]`

---

#### REQ-07-003 — Acesso Autenticado com Imóveis Vinculados
**Derives:** BIZ-07-003, BIZ-07-004

**Descrição:** Login com exibição automática dos imóveis vinculados ao CPF do cidadão no mapa.

**Use Case:**
- **Ator:** Cidadão (autenticado)
- **Pré-condições:** Cidadão cadastrado no sistema.
- **Fluxo principal:**
  1. Cidadão clica em "Entrar".
  2. Sistema exibe Termo de Uso (se primeiro acesso ou atualização — REQ-01-002).
  3. Cidadão se autentica (CPF + senha ou Gov.Br).
  4. Sistema consulta imóveis vinculados ao CPF.
  5. Mapa centraliza e destaca os imóveis do cidadão.
  6. Menu exibe serviços autenticados: Certidões, ITBI, Fiscalizações, Protocolos.
- **Fluxos alternativos:**
  - 4a. Nenhum imóvel vinculado ao CPF → mensagem "Nenhum imóvel encontrado vinculado ao seu CPF."
  - 3a. Credenciais inválidas → erro "CPF ou senha incorretos."
- **Pós-condições:** Cidadão autenticado com imóveis visíveis; serviços habilitados.
- **Exceções:** Conta bloqueada → "Conta bloqueada. Procure a prefeitura."

**Critérios de aceite:**
- [ ] Login por CPF + senha ou Gov.Br (REQ-07-004)
- [ ] Imóveis vinculados ao CPF destacados automaticamente no mapa
- [ ] Termo de Uso exibido quando necessário
- [ ] Mensagem adequada quando sem imóveis vinculados
- [ ] Serviços autenticados habilitados após login

**Dependências:** `depends_on: [REQ-07-001, REQ-01-001, REQ-01-002]`

---

#### REQ-07-004 — Login via Gov.Br
**Derives:** BIZ-07-005

**Descrição:** Autenticação via Login Único do Gov.Br (Federal), com credenciais de integração (client_id, secret) configuráveis por tenant.

**Use Case:**
- **Ator:** Cidadão (autenticado)
- **Pré-condições:** Integração Gov.Br configurada pelo administrador; cidadão possui conta Gov.Br.
- **Fluxo principal:**
  1. Cidadão clica em "Entrar com Gov.Br".
  2. Sistema redireciona para página de login do Gov.Br (OAuth 2.0).
  3. Cidadão se autentica no Gov.Br.
  4. Gov.Br retorna token com CPF e nível de confiabilidade.
  5. Sistema vincula a sessão ao CPF e carrega imóveis.
- **Fluxos alternativos:**
  - 2a. Gov.Br indisponível → mensagem "Serviço Gov.Br temporariamente indisponível. Use login por CPF e senha."
  - 4a. CPF não cadastrado localmente → sistema cria registro básico do cidadão.
- **Pós-condições:** Cidadão autenticado via Gov.Br.
- **Exceções:** Falha no callback OAuth → erro "Falha na autenticação. Tente novamente."

**Critérios de aceite:**
- [ ] Integração OAuth 2.0 conforme especificações do Gov.Br
- [ ] client_id e secret configuráveis por tenant
- [ ] Fallback para login local se Gov.Br indisponível
- [ ] Nível de confiabilidade do Gov.Br armazenado (bronze/prata/ouro)
- [ ] Auto-criação de registro local quando CPF não existe

**Dependências:** `depends_on: [REQ-07-003, REQ-01-018]`

---

#### REQ-07-005 — Captcha Cloudflare Turnstile
**Derives:** BIZ-07-006

**Descrição:** Mecanismo de Captcha via Cloudflare Turnstile em formulários públicos e tela de login, com credenciais exclusivas por tenant.

**Use Case:**
- **Ator:** Cidadão (anônimo ou na tela de login)
- **Pré-condições:** Credenciais Turnstile configuradas pelo administrador.
- **Fluxo principal:**
  1. Cidadão acessa formulário público ou tela de login.
  2. Sistema renderiza widget Cloudflare Turnstile.
  3. Turnstile valida automaticamente (sem interação na maioria dos casos).
  4. Token de validação enviado junto com o formulário.
  5. Backend verifica token via API Cloudflare.
- **Fluxos alternativos:**
  - 3a. Turnstile solicita desafio interativo → cidadão resolve.
  - 5a. Token inválido → erro "Validação de segurança falhou. Tente novamente."
  - 2a. Credenciais não configuradas → Captcha não renderizado (formulário funciona sem).
- **Pós-condições:** Submissão do formulário validada contra bots.
- **Exceções:** Cloudflare indisponível → formulário funciona sem Captcha (degradação graciosa).

**Critérios de aceite:**
- [ ] Turnstile renderizado em formulários públicos e login
- [ ] Site key e secret key configuráveis por tenant
- [ ] Degradação graciosa se Cloudflare indisponível
- [ ] Validação server-side do token

**Dependências:** `depends_on: [REQ-07-001, REQ-01-018]`

---

## 2. Consulta e Serviços

---

#### REQ-07-006 — Consulta de Imóveis pelo Cidadão
**Derives:** BIZ-07-007, BIZ-07-008, BIZ-07-009

**Descrição:** Consulta por CPF (logado), endereço, cadastro, matrícula ou inscrição. Exibição de informações públicas autorizadas; dados restritos apenas para cidadão autenticado e vinculado.

**Use Case:**
- **Ator:** Cidadão (autenticado ou anônimo)
- **Pré-condições:** Portal carregado.
- **Fluxo principal:**
  1. Cidadão insere critério de busca na barra de pesquisa.
  2. Sistema retorna lista de imóveis correspondentes.
  3. Cidadão seleciona imóvel; mapa centraliza.
  4. Sistema exibe informações públicas: endereço, área, zoneamento.
- **Fluxos alternativos:**
  - 4a. Cidadão autenticado e vinculado ao imóvel → exibe também: dados do proprietário, débitos detalhados.
  - 4b. Cidadão autenticado mas não vinculado → exibe apenas dados públicos.
- **Pós-condições:** Informações do imóvel exibidas conforme nível de acesso.
- **Exceções:** Nenhum resultado → "Nenhum imóvel encontrado."

**Critérios de aceite:**
- [ ] Busca por: CPF (logado), endereço, cadastro, matrícula, inscrição
- [ ] Dados públicos visíveis a todos: endereço, área, zoneamento
- [ ] Dados restritos visíveis apenas para proprietário autenticado: CPF, débitos, proprietário

| Dado | Anônimo | Autenticado (não vinculado) | Autenticado (vinculado) |
|------|---------|-----------------------------|------------------------|
| Endereço | Visível | Visível | Visível |
| Área | Visível | Visível | Visível |
| Zoneamento | Visível | Visível | Visível |
| Proprietário | Oculto | Oculto | Visível |
| Débitos | Oculto | Oculto | Visível |

**Dependências:** `depends_on: [REQ-07-001, REQ-04-006]`

---

#### REQ-07-007 — Emissão de Certidões Online
**Derives:** BIZ-07-010, BIZ-07-011, BIZ-07-012, BIZ-07-013

**Descrição:** Emissão online de certidões a partir de catálogo configurável pela municipalidade. Templates, regras de elegibilidade e taxas configuráveis. Certidão com código de verificação (QR code).

**Use Case:**
- **Ator:** Cidadão (autenticado)
- **Pré-condições:** Cidadão autenticado; imóvel vinculado ao CPF; catálogo de certidões configurado.
- **Fluxo principal:**
  1. Cidadão seleciona imóvel.
  2. Acessa "Certidões".
  3. Sistema exibe catálogo de certidões disponíveis (ex.: confrontação, débitos, avaliação, uso do solo, localização, certidão negativa).
  4. Cidadão seleciona tipo de certidão.
  5. Sistema verifica elegibilidade (ex.: sem débitos para certidão negativa).
  6. Gera certidão PDF com QR code de verificação.
  7. Disponibiliza para download.
- **Fluxos alternativos:**
  - 5a. Não elegível → mensagem "Certidão não disponível: [motivo]." (ex.: "Existem débitos pendentes.")
  - 5b. Certidão com taxa → exibe valor e guia de pagamento antes da emissão.
- **Pós-condições:** Certidão emitida e registrada no histórico.
- **Exceções:** Template não configurado → certidão não aparece no catálogo.

**Critérios de aceite:**
- [ ] Catálogo de certidões configurável pelo administrador
- [ ] Template, dados incluídos, regras de elegibilidade e taxa configuráveis
- [ ] PDF com QR code de verificação de autenticidade
- [ ] Meta: 3+ tipos em 6 meses, 6+ tipos em 12 meses
- [ ] Verificação online de autenticidade via QR code/código
- [ ] Histórico de certidões emitidas por cidadão

**Dependências:** `depends_on: [REQ-07-003, REQ-04-001, REQ-02-005]`

---

#### REQ-07-008 — Solicitação de ITBI
**Derives:** BIZ-07-014, BIZ-07-015, BIZ-07-016

**Descrição:** Solicitação online de ITBI com dados da transação, cálculo automático conforme legislação municipal e geração de guia de recolhimento.

**Use Case:**
- **Ator:** Cidadão (autenticado)
- **Pré-condições:** Cidadão autenticado; imóvel existente; alíquota configurada.
- **Fluxo principal:**
  1. Cidadão acessa "Solicitar ITBI".
  2. Seleciona imóvel da transação.
  3. Informa: valor da transação, dados do comprador e vendedor.
  4. Sistema obtém valor venal do imóvel (PGV — REQ-05-006).
  5. Calcula ITBI: alíquota × max(valor declarado, valor venal).
  6. Exibe memória de cálculo.
  7. Gera guia de recolhimento (boleto ou PIX).
- **Fluxos alternativos:**
  - 5a. Isenção aplicável → "ITBI isento: [motivo]."
  - 4a. Valor venal não disponível (PGV incompleta) → usar valor declarado com alerta.
- **Pós-condições:** Guia gerada; solicitação registrada.
- **Exceções:** Alíquota não configurada → "ITBI não disponível para este município."

**Critérios de aceite:**
- [ ] Dados da transação: imóvel, comprador, vendedor, valor
- [ ] Base de cálculo: maior entre valor declarado e venal (configurável)
- [ ] Alíquota e isenções configuráveis por município (REQ-02-009)
- [ ] Memória de cálculo exibida ao cidadão
- [ ] Guia de recolhimento gerada (PDF)

**Dependências:** `depends_on: [REQ-07-003, REQ-04-001, REQ-05-006, REQ-02-009]`

---

#### REQ-07-009 — Interação com Fiscalizações
**Derives:** BIZ-07-017, BIZ-07-018

**Descrição:** Visualização de fiscalizações em andamento e resposta a notificações com envio de documentos e questionamentos.

**Use Case:**
- **Ator:** Cidadão (autenticado)
- **Pré-condições:** Fiscalização existente vinculada a imóvel do cidadão.
- **Fluxo principal:**
  1. Cidadão acessa "Minhas Fiscalizações".
  2. Sistema lista fiscalizações em andamento dos imóveis vinculados ao CPF.
  3. Cidadão seleciona fiscalização e visualiza: motivo, status, documentos emitidos.
  4. Clica em "Responder".
  5. Anexa documentos e escreve questionamento/resposta.
  6. Submete.
- **Pós-condições:** Resposta registrada na fiscalização; servidor fiscal notificado.
- **Exceções:** Nenhuma fiscalização encontrada → "Nenhuma fiscalização em andamento para seus imóveis."

**Critérios de aceite:**
- [ ] Lista de fiscalizações dos imóveis vinculados ao CPF
- [ ] Visualização de documentos emitidos (notificação, auto, embargo)
- [ ] Envio de resposta com texto e anexos
- [ ] Servidor fiscal recebe notificação da resposta

**Dependências:** `depends_on: [REQ-07-003, REQ-06-001]`

---

#### REQ-07-010 — Abertura de Protocolos
**Derives:** BIZ-07-019, BIZ-07-020

**Descrição:** Abertura de protocolos/solicitações junto à prefeitura, com suporte a integração com sistemas de protocolo terceiros.

**Use Case:**
- **Ator:** Cidadão (autenticado)
- **Pré-condições:** Cidadão autenticado.
- **Fluxo principal:**
  1. Cidadão acessa "Abrir Protocolo".
  2. Seleciona tipo de solicitação (divergência cadastral, reclamação, solicitação geral).
  3. Descreve a solicitação e anexa documentos.
  4. Submete.
  5. Sistema registra protocolo internamente.
- **Fluxos alternativos:**
  - 5a. Município com sistema de protocolo terceiro configurado → GEO encaminha via API e retorna número do protocolo externo.
- **Pós-condições:** Protocolo registrado; número fornecido ao cidadão.
- **Exceções:** Integração com sistema terceiro falha → protocolo registrado internamente com aviso "Protocolo registrado localmente. O encaminhamento será realizado em breve."

**Critérios de aceite:**
- [ ] Tipos de solicitação configuráveis
- [ ] Texto descritivo + anexos
- [ ] Número de protocolo gerado
- [ ] Integração com sistema terceiro: opcional, configurável por município
- [ ] Fallback para registro interno se integração falhar

**Dependências:** `depends_on: [REQ-07-003]`

---

## 3. Visualização

---

#### REQ-07-011 — Camadas Públicas e Impressão
**Derives:** BIZ-07-021, BIZ-07-022

**Descrição:** Controle de visibilidade de camadas públicas pelo cidadão e impressão/exportação de imagem do imóvel.

**Use Case:**
- **Ator:** Cidadão (autenticado ou anônimo)
- **Pré-condições:** Portal carregado com camadas públicas configuradas.
- **Fluxo principal:**
  1. Cidadão abre painel de camadas.
  2. Ativa/desativa camadas públicas (zoneamento, equipamentos, etc.).
  3. Seleciona imóvel.
  4. Clica em "Imprimir / Exportar".
  5. Sistema gera imagem do mapa com imóvel destacado (PDF ou PNG).
- **Pós-condições:** Imagem exportada.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Cidadão pode ativar/desativar camadas públicas
- [ ] Apenas camadas marcadas como públicas pelo administrador aparecem
- [ ] Impressão em PDF e PNG
- [ ] Imóvel destacado na imagem exportada

**Dependências:** `depends_on: [REQ-07-001, REQ-03-005, REQ-03-011]`
