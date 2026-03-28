---
document_type: prd
domain: plataforma
derives_from: brd/01_platform_brd.md
file: prd/01_platform_prd.md
requirements:
  - id: REQ-01-001
    title: "Sistema RBAC de Controle de Acesso"
    depends_on: []
  - id: REQ-01-002
    title: "Gestão de Termo de Uso"
    depends_on: []
  - id: REQ-01-003
    title: "Banner de Privacidade para Acesso Anônimo"
    depends_on: []
  - id: REQ-01-004
    title: "Página de Política de Privacidade"
    depends_on: [REQ-01-018]
  - id: REQ-01-005
    title: "Portal de Direitos do Titular LGPD"
    depends_on: [REQ-01-001]
  - id: REQ-01-006
    title: "Registro de Operações de Tratamento"
    depends_on: [REQ-01-019]
  - id: REQ-01-007
    title: "Interface de Consulta de Logs de Auditoria"
    depends_on: [REQ-01-001, REQ-01-019]
  - id: REQ-01-008
    title: "Exportação Integral de Dados por Município"
    depends_on: [REQ-01-018]
  - id: REQ-01-009
    title: "Minimização de Coleta de Dados"
    depends_on: []
  - id: REQ-01-010
    title: "Tratamento Diferenciado de Dados Sensíveis"
    depends_on: [REQ-01-001]
  - id: REQ-01-011
    title: "Anonimização em Indicadores e Relatórios"
    depends_on: []
  - id: REQ-01-012
    title: "Mascaramento em Exportações de Dados"
    depends_on: [REQ-01-001]
  - id: REQ-01-013
    title: "Pseudonimização em Logs"
    depends_on: [REQ-01-019]
  - id: REQ-01-014
    title: "Conformidade com Licenças de Provedores Cartográficos"
    depends_on: []
  - id: REQ-01-015
    title: "Titularidade de Documentos e Dados"
    depends_on: []
  - id: REQ-01-016
    title: "Aviso Legal ao Cidadão"
    depends_on: []
  - id: REQ-01-017
    title: "Proteção contra Download em Massa"
    depends_on: []
  - id: REQ-01-018
    title: "Isolamento Multi-tenant"
    depends_on: []
  - id: REQ-01-019
    title: "Trilha de Auditoria Imutável"
    depends_on: []
  - id: REQ-01-020
    title: "Backup e Recuperação"
    depends_on: [REQ-01-018]
---

# PRD-01 — Requisitos da Plataforma

## 1. Requisitos Funcionais

### 1.1 Gestão de Acessos e Autenticação

---

#### REQ-01-001 — Sistema RBAC de Controle de Acesso
**Derives:** BIZ-01-020, BIZ-01-022, BIZ-01-023

**Descrição:** O sistema deve implementar controle de acesso baseado em perfis (RBAC) com permissões granulares por módulo, funcionalidade e setor do município, incluindo hierarquia de aprovação para perfis administrativos e expiração de sessão.

**Use Case:**

- **Ator principal:** Administrador do Sistema
- **Pré-condições:** O administrador está autenticado com perfil de gestão de acessos.
- **Fluxo principal:**
  1. Administrador acessa o módulo de gestão de perfis.
  2. Cria novo perfil definindo: nome, descrição, módulos permitidos, funcionalidades habilitadas e setores associados.
  3. Define permissões granulares (leitura, escrita, exclusão) por funcionalidade.
  4. Salva o perfil.
  5. Associa o perfil a um ou mais usuários.
- **Fluxos alternativos:**
  - 3a. Se o perfil inclui acesso total (administrador), o sistema requer aprovação de administrador de nível superior antes de ativar.
  - 5a. Ao associar perfil a usuário já logado, as permissões são atualizadas na próxima requisição.
- **Pós-condições:** Perfil criado e ativo; usuários associados operam com as permissões definidas.
- **Exceções:**
  - Tentativa de criar perfil com acesso total sem aprovação superior → erro "Requer aprovação de administrador de nível superior".
  - Sessão expira após período configurável de inatividade → redirecionamento para tela de login.

**Critérios de aceite:**
- [ ] Perfis podem ser criados com permissões granulares por módulo/funcionalidade/setor
- [ ] Perfil com acesso total requer aprovação de administrador superior
- [ ] Sessão expira após tempo configurável de inatividade (padrão: 30 min)
- [ ] Alteração de perfil reflete nas permissões do usuário sem necessidade de relogin
- [ ] Não é possível atribuir permissões a módulos inexistentes

**Dependências:** `depends_on: []`

---

#### REQ-01-002 — Gestão de Termo de Uso
**Derives:** BIZ-01-006, BIZ-01-007, BIZ-01-008

**Descrição:** O sistema deve exibir Termo de Uso ao cidadão no primeiro acesso, exigir aceite explícito, registrar o aceite com metadados e solicitar novo aceite quando o termo for atualizado.

**Use Case:**

- **Ator principal:** Cidadão (autenticado)
- **Pré-condições:** Cidadão realiza login pela primeira vez ou o Termo de Uso foi atualizado desde o último aceite.
- **Fluxo principal:**
  1. Cidadão faz login no Portal.
  2. Sistema verifica se existe aceite válido para a versão corrente do Termo de Uso.
  3. Se não existe, exibe o Termo de Uso completo com botão "Aceitar".
  4. Cidadão lê e clica em "Aceitar".
  5. Sistema registra: data/hora, versão do termo, identificação do usuário e IP.
  6. Cidadão é redirecionado para a tela principal.
- **Fluxos alternativos:**
  - 4a. Cidadão clica em "Recusar" → sessão encerrada; não pode acessar funcionalidades que envolvam dados pessoais.
  - 2a. Aceite existente para versão corrente → fluxo encerrado, acesso liberado.
- **Pós-condições:** Aceite registrado; cidadão pode acessar funcionalidades do Portal.
- **Exceções:**
  - Falha ao registrar aceite → exibir mensagem de erro e solicitar nova tentativa.

**Critérios de aceite:**
- [ ] Termo exibido no primeiro acesso e após atualização do termo
- [ ] Registro contém: data/hora (UTC), versão do termo, ID do usuário
- [ ] Cidadão não pode acessar funcionalidades que envolvam dados pessoais sem aceite
- [ ] Administrador pode atualizar o texto do Termo de Uso, gerando nova versão

**Dependências:** `depends_on: []`

---

#### REQ-01-003 — Banner de Privacidade para Acesso Anônimo
**Derives:** BIZ-01-009

**Descrição:** Para acessos anônimos (sem login), o sistema deve exibir banner informando as condições de uso e os dados coletados (IP, navegador).

**Use Case:**

- **Ator principal:** Cidadão (anônimo)
- **Pré-condições:** Usuário acessa o Portal sem autenticação.
- **Fluxo principal:**
  1. Usuário acessa qualquer página pública do Portal.
  2. Sistema exibe banner fixo informando: dados coletados automaticamente (IP, navegador, cookies), finalidade e link para Política de Privacidade.
  3. Usuário pode fechar o banner (aceite implícito) ou clicar no link da Política de Privacidade.
  4. Preferência de fechamento é armazenada em cookie local.
- **Fluxos alternativos:**
  - 4a. Se cookie de aceite já existe, banner não é exibido novamente.
- **Pós-condições:** Usuário informado sobre coleta de dados; navegação liberada.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Banner exibido em toda página pública para visitantes sem cookie de aceite
- [ ] Banner contém link para Política de Privacidade
- [ ] Após fechar, banner não reaparece na mesma sessão/navegador
- [ ] Banner não bloqueia navegação (informativo, não bloqueante)

**Dependências:** `depends_on: []`

---

### 1.2 Gestão de Privacidade e LGPD

---

#### REQ-01-004 — Página de Política de Privacidade
**Derives:** BIZ-01-013, BIZ-01-014

**Descrição:** Política de Privacidade acessível a partir de qualquer página, descrevendo dados coletados, finalidade, tempo de retenção e contato com o DPO. Personalizável por município.

**Use Case:**

- **Ator principal:** Cidadão (autenticado ou anônimo)
- **Pré-condições:** Nenhuma.
- **Fluxo principal:**
  1. Usuário clica no link "Política de Privacidade" disponível no rodapé de qualquer página.
  2. Sistema carrega o conteúdo da Política de Privacidade do município corrente (tenant).
  3. Exibe: dados coletados, finalidade, tempo de retenção, nome do Encarregado (DPO), canal de atendimento e endereço.
- **Fluxos alternativos:**
  - 3a. Se o município não personalizou a Política, exibir template padrão da plataforma.
- **Pós-condições:** Política de Privacidade exibida ao usuário.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Link acessível no rodapé de toda página (pública e autenticada)
- [ ] Conteúdo personalizável por município: DPO, canal de atendimento, endereço
- [ ] Template padrão exibido quando não houver personalização
- [ ] Página renderiza sem autenticação

**Dependências:** `depends_on: [REQ-01-018]`

---

#### REQ-01-005 — Portal de Direitos do Titular LGPD
**Derives:** BIZ-01-004

**Descrição:** Interface para que o titular de dados exerça seus direitos previstos no Art. 18 da LGPD: acesso, correção, anonimização, eliminação e portabilidade de dados pessoais.

**Use Case:**

- **Ator principal:** Cidadão (autenticado)
- **Pré-condições:** Cidadão autenticado no Portal.
- **Fluxo principal:**
  1. Cidadão acessa seção "Meus Dados / Privacidade".
  2. Sistema exibe opções: Consultar meus dados, Solicitar correção, Solicitar anonimização, Solicitar eliminação, Solicitar portabilidade.
  3. Cidadão seleciona uma opção.
  4. Sistema registra a solicitação e encaminha para análise do administrador.
  5. Administrador processa a solicitação e registra a resposta.
  6. Cidadão recebe notificação com o resultado.
- **Fluxos alternativos:**
  - 3a. "Consultar meus dados" → exibe diretamente os dados pessoais armazenados, sem necessidade de aprovação.
- **Pós-condições:** Solicitação registrada e encaminhada para processamento.
- **Exceções:**
  - Solicitação de eliminação de dados vinculados a obrigação legal → sistema informa que a eliminação não é possível com fundamentação.

**Critérios de aceite:**
- [ ] Cidadão pode consultar todos os dados pessoais armazenados
- [ ] Solicitações de correção/anonimização/eliminação/portabilidade geram registro
- [ ] Administrador recebe fila de solicitações pendentes
- [ ] Eliminação bloqueada quando houver base legal que exija retenção (com justificativa)

**Dependências:** `depends_on: [REQ-01-001]`

---

#### REQ-01-006 — Registro de Operações de Tratamento
**Derives:** BIZ-01-002, BIZ-01-003

**Descrição:** O sistema deve documentar a base legal de cada operação de tratamento de dados pessoais e manter registro das operações de tratamento (ROPA) conforme Art. 37 da LGPD.

**Use Case:**

- **Ator principal:** Sistema (automático) / Administrador
- **Pré-condições:** Trilha de auditoria ativa (REQ-01-019).
- **Fluxo principal:**
  1. Para cada operação que envolva dados pessoais, o sistema registra automaticamente: tipo de tratamento, base legal aplicável, dados envolvidos e finalidade.
  2. Administrador pode consultar o ROPA consolidado com filtros por período, tipo de operação e base legal.
- **Pós-condições:** Registro de operações de tratamento mantido e consultável.
- **Exceções:** Nenhuma.

**Critérios de aceite:**
- [ ] Cada operação sobre dados pessoais gera registro com base legal documentada
- [ ] ROPA consultável pelo administrador com filtros
- [ ] Bases legais suportadas: consentimento, obrigação legal, execução de políticas públicas, legítimo interesse
- [ ] Registros exportáveis para atendimento a fiscalizações da ANPD

**Dependências:** `depends_on: [REQ-01-019]`

---

### 1.3 Auditoria e Relatórios

---

#### REQ-01-007 — Interface de Consulta de Logs de Auditoria
**Derives:** BIZ-01-027

**Descrição:** Interface administrativa para consulta de logs de auditoria, com filtros por período, usuário e tipo de ação.

**Use Case:**

- **Ator principal:** Administrador do Sistema
- **Pré-condições:** Administrador autenticado com perfil de auditoria; trilha de auditoria ativa.
- **Fluxo principal:**
  1. Administrador acessa módulo de auditoria.
  2. Define filtros: período (data início/fim), usuário, tipo de ação (criação, alteração, exclusão, consulta), módulo.
  3. Sistema retorna lista paginada de registros.
  4. Administrador pode expandir um registro para ver detalhes (valores anterior/posterior).
  5. Administrador pode exportar resultado filtrado em CSV ou PDF.
- **Pós-condições:** Logs consultados; exportação opcional realizada.
- **Exceções:**
  - Filtro sem resultados → mensagem informativa.

**Critérios de aceite:**
- [ ] Filtros: período, usuário, tipo de ação, módulo
- [ ] Resultados paginados com no mínimo 50 registros por página
- [ ] Detalhe do registro mostra valores anterior e posterior
- [ ] Exportação em CSV e PDF
- [ ] Apenas perfis com permissão de auditoria podem acessar

**Dependências:** `depends_on: [REQ-01-001, REQ-01-019]`

---

#### REQ-01-008 — Exportação Integral de Dados por Município
**Derives:** BIZ-01-029

**Descrição:** O sistema deve permitir exportação integral dos dados geoespaciais e cadastrais de um município para garantir portabilidade em caso de rescisão contratual.

**Use Case:**

- **Ator principal:** Administrador do Sistema (nível plataforma)
- **Pré-condições:** Solicitação formal de exportação; administrador autenticado com perfil máximo.
- **Fluxo principal:**
  1. Administrador seleciona o município e aciona "Exportar dados integrais".
  2. Sistema gera pacote contendo: base cadastral (imóveis, empresas, pessoas), dados geoespaciais (polígonos, camadas), documentos anexos, configurações (zoneamento, PGV, templates).
  3. Pacote disponibilizado para download em formato aberto (CSV + GeoJSON/Shapefile).
- **Pós-condições:** Pacote completo gerado e disponibilizado.
- **Exceções:**
  - Volume de dados muito grande → sistema gera exportação assíncrona e notifica quando pronto.

**Critérios de aceite:**
- [ ] Exportação inclui todos os dados do município (cadastral, geoespacial, documentos, configurações)
- [ ] Formatos abertos: CSV para tabular, GeoJSON ou Shapefile para geoespacial
- [ ] Exportação não inclui dados de outros municípios (respeita isolamento)
- [ ] Processo assíncrono para volumes grandes com notificação de conclusão

**Dependências:** `depends_on: [REQ-01-018]`

---

## 2. Requisitos Não Funcionais

### 2.1 Segurança e Privacidade de Dados

---

#### REQ-01-009 — Minimização de Coleta de Dados
**Derives:** BIZ-01-001

**Descrição:** Cada formulário e operação deve coletar apenas os dados pessoais estritamente necessários para sua finalidade.

**Critério de aceite:** Todo campo de dados pessoais em formulários deve estar mapeado a uma finalidade documentada. Campos não essenciais devem ser opcionais ou removidos.

---

#### REQ-01-010 — Tratamento Diferenciado de Dados Sensíveis
**Derives:** BIZ-01-005

**Descrição:** Dados pessoais sensíveis (origem racial/étnica, saúde, biometria) devem possuir controles adicionais de acesso e auditoria.

**Critério de aceite:** Acesso a campos sensíveis restrito a perfis explicitamente autorizados. Toda consulta a dados sensíveis gera log de auditoria específico.

**Dependências:** `depends_on: [REQ-01-001]`

---

#### REQ-01-011 — Anonimização em Indicadores e Relatórios
**Derives:** BIZ-01-010

**Descrição:** Dados utilizados para indicadores e relatórios agregados devem ser anonimizados.

**Critério de aceite:** Indicadores e relatórios agregados não contêm dados que permitam identificação direta ou indireta do titular. Validado por revisão de cada endpoint de indicadores.

---

#### REQ-01-012 — Mascaramento em Exportações de Dados
**Derives:** BIZ-01-011

**Descrição:** Exportações em massa (CSV, Excel, TXT) para usuários sem perfil administrativo devem omitir ou mascarar dados pessoais sensíveis.

**Critério de aceite:**

| Dado | Perfil Administrativo | Perfil Operacional | Perfil Público |
|------|----------------------|-------------------|----------------|
| CPF | Exibido completo | Mascarado (***.XXX.XXX-**) | Omitido |
| Nome proprietário | Exibido | Exibido | Omitido |
| Endereço | Exibido | Exibido | Exibido |
| Área do terreno | Exibido | Exibido | Exibido |

**Dependências:** `depends_on: [REQ-01-001]`

---

#### REQ-01-013 — Pseudonimização em Logs
**Derives:** BIZ-01-012

**Descrição:** Logs e trilhas de auditoria devem pseudonimizar dados pessoais quando possível, substituindo identificadores diretos por tokens reversíveis apenas por administradores autorizados.

**Critério de aceite:** Logs não contêm CPF, nome completo ou dados sensíveis em texto plano. Administrador autorizado pode despseudonimizar quando necessário para investigação.

**Dependências:** `depends_on: [REQ-01-019]`

---

### 2.2 Propriedade Intelectual

---

#### REQ-01-014 — Conformidade com Licenças de Provedores Cartográficos
**Derives:** BIZ-01-015

**Descrição:** O sistema deve respeitar os termos de licença de provedores de dados cartográficos e imagens (Google, Esri, OpenStreetMap, Mapbox).

**Critério de aceite:** Atribuições de copyright exibidas conforme exigência de cada provedor. Uso de APIs conforme limites e termos contratados.

---

#### REQ-01-015 — Titularidade de Documentos e Dados
**Derives:** BIZ-01-016, BIZ-01-018

**Descrição:** Documentos gerados incluem identificação do município emissor no rodapé. Dados importados do ERP mantêm titularidade do município de origem.

**Critério de aceite:** Todo documento/certidão gerado contém no rodapé: brasão/nome do município, nome da plataforma e data de emissão. Metadados de dados importados incluem município de origem.

---

#### REQ-01-016 — Aviso Legal ao Cidadão
**Derives:** BIZ-01-017

**Descrição:** O cidadão deve ser informado que a reprodução ou uso indevido de dados obtidos via plataforma pode constituir infração legal.

**Critério de aceite:** Aviso legal visível na Política de Privacidade, no Termo de Uso e no rodapé de documentos emitidos.

---

#### REQ-01-017 — Proteção contra Download em Massa
**Derives:** BIZ-01-019

**Descrição:** O sistema não deve permitir download em massa de imagens de satélite ou bases cartográficas, respeitando restrições de redistribuição.

**Critério de aceite:** Não existe endpoint que permita download de tiles/imagens de satélite em lote. Exportação de imagem limitada a recortes individuais de área visível. Rate limiting aplicado a requisições de imagens.

---

### 2.3 Infraestrutura e Governança

---

#### REQ-01-018 — Isolamento Multi-tenant
**Derives:** BIZ-01-024, BIZ-01-025

**Descrição:** Isolamento completo de dados e configurações entre municípios. Nenhum tenant pode acessar dados de outro.

**Critério de aceite:**
- Toda query de dados inclui filtro obrigatório por tenant_id (a nível de framework, não apenas aplicação).
- Configurações de camadas, documentos, zoneamento e perfis são independentes por tenant.
- Teste de penetração entre tenants não revela dados cruzados.

---

#### REQ-01-019 — Trilha de Auditoria Imutável
**Derives:** BIZ-01-021, BIZ-01-026

**Descrição:** Toda ação que modifique dados cadastrais, fiscais ou territoriais gera registro de auditoria imutável contendo: usuário, data/hora, ação, valores anterior/posterior.

**Critério de aceite:**
- Registros de auditoria armazenados em storage append-only (não podem ser alterados ou excluídos).
- Cada registro contém: user_id, timestamp (UTC), action_type, entity_type, entity_id, old_value (JSON), new_value (JSON).
- Retenção mínima de 5 anos ou conforme legislação aplicável.

---

### 2.4 Disponibilidade e Continuidade

---

#### REQ-01-020 — Backup e Recuperação
**Derives:** BIZ-01-028

**Descrição:** Mecanismos de backup e recuperação que garantam continuidade operacional em caso de falha.

**Critério de aceite:**
- Backup automatizado diário de todos os dados (cadastral, geoespacial, configurações).
- RPO (Recovery Point Objective) ≤ 24 horas.
- RTO (Recovery Time Objective) ≤ 4 horas.
- Teste de recuperação documentado e executado pelo menos trimestralmente.

**Dependências:** `depends_on: [REQ-01-018]`
