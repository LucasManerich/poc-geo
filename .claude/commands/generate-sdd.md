# /generate-sdd — Gerar Software Design Documents

## Contexto
Você é um Arquiteto de Software experiente trabalhando junto com Tech Leads e Analistas de Sistema. Sua tarefa é gerar os documentos SDD (Software Design Document) derivados dos PRDs existentes, validando viabilidade técnica.

## Inputs
Leia obrigatoriamente os seguintes arquivos antes de iniciar a geração:
1. `ai-spec/prd/*.md` — Todos os PRDs existentes (input principal)
2. `ai-spec/brd/*.md` — Todos os BRDs (contexto de negócio)
3. `ai-spec/inputs/diagrams/*.mermaid` — Diagramas de entrada (se existirem)
4. `ai-spec/sdd/01_platform_sdd.md` — Restrições de plataforma (se já existir, para não contradizer)
5. `ai-spec/reviews/tsd_feedback_to_sdd_*.md` — Feedbacks pendentes do TSD (se existirem, para regeneração)

## Outputs
Gerar os seguintes documentos em `ai-spec/sdd/`:

### Documentos obrigatórios:
- `01_platform_sdd.md` — Definições de tecnologia para todos os produtos
- `02_product_sdd.md` — Definições de tecnologia do produto

### Documentos por domínio:
- `03_domain_<nome>_sdd.md` — Derivado do PRD de mesmo número
- A numeração DEVE corresponder ao PRD de origem

### Feedbacks (se conflitos detectados):
- `ai-spec/reviews/sdd_feedback_to_prd_NN.md` — Um arquivo por PRD com conflitos

## Regras de Geração

### Frontmatter YAML obrigatório em cada documento:
```yaml
---
document_type: sdd
domain: <nome_do_dominio>
derives_from: prd/<nome_do_prd_origem>.md
file: sdd/<nome_do_arquivo>.md
---
```

### IDs de design:
- Prefixo: `DES-`
- Formato: `DES-NN-SSS` onde NN = número do documento, SSS = sequencial
- Cada DES DEVE referenciar o REQ pai: `derives: REQ-NN-SSS`
- Exemplo: `DES-03-001 (derives: REQ-03-001)`

### Conteúdo do `01_platform_sdd.md`:
1. **Stack tecnológica imposta:**
   - Linguagens e versões permitidas
   - Frameworks e bibliotecas padrão
   - Ferramentas de build e empacotamento
2. **Considerações de deploy obrigatórias:**
   - Containerização (Docker, etc.)
   - Orquestração (Kubernetes, etc.)
   - Ambientes obrigatórios
3. **Premissas arquiteturais:**
   - Boas práticas de desenvolvimento obrigatórias
   - Documentação ativa no código (JSDoc, docstrings, etc.)
   - Padrões de commit e branching

### Conteúdo do `02_product_sdd.md`:
1. **Estrutura de domínios:**
   - Mapa completo de bounded contexts do produto
   - Relações entre domínios (Context Map em Mermaid)
   - Módulos compartilhados (shared kernel)
2. **Stack do produto** (sem contradizer `01_platform_sdd.md`):
   - Bibliotecas específicas do produto
   - Justificativa para cada escolha
3. **Considerações de deploy do produto** (sem contradizer `01_platform_sdd.md`)
4. **Design patterns:**
   - Patterns permitidos (lista com contexto de quando usar)
   - Patterns proibidos (lista com justificativa)
5. **Code smells a evitar:**
   - Lista de anti-patterns monitorados

### Conteúdo dos documentos de domínio `03+_domain_<nome>_sdd.md`:
Para cada requisito derivado do PRD:
1. **ID e título:** `DES-NN-SSS — Título` (derives: REQ-NN-SSS)
2. **Visão geral da funcionalidade:**
   - O que será construído
   - Componentes envolvidos
3. **Arquitetura da funcionalidade:**
   - Diagrama mermaid do fluxo técnico
   - Camadas envolvidas (controller, service, repository, etc.)
   - Contratos de entrada/saída
4. **Modelo de dados do domínio:**
   - Entidades e value objects (DDD)
   - Diagrama ER em Mermaid
   - Regras de validação
5. **Considerações de segurança:**
   - Autenticação/autorização necessária
   - Validação de input
   - Proteção de dados sensíveis

## Validação de Viabilidade Técnica (Feedback Loop)
Durante a geração, para CADA requisito do PRD:
1. Avaliar se o requisito é tecnicamente viável com a stack definida
2. Avaliar se os requisitos não funcionais (performance, etc.) são atingíveis
3. Avaliar se as integrações descritas são possíveis

### Se um requisito for INVIÁVEL:
1. Gerar o item DES correspondente no SDD com anotação `⚠️ CONFLITO`
2. Criar arquivo de feedback:

```markdown
---
review_type: sdd_to_prd
source_doc: sdd/<arquivo_sdd>.md
target_doc: prd/<arquivo_prd>.md
generated_at: <timestamp_ISO8601>
status: pending
---

## Conflitos Identificados

### CONFLICT-001
- **Requisito:** REQ-NN-SSS
- **Problema:** <descrição técnica do problema>
- **Sugestão:** <alternativa viável proposta>
- **Impacto:** alto | médio | baixo
- **Decisão:** pending
```

3. Continuar gerando os demais itens normalmente
4. Ao final, listar todos os conflitos encontrados para o operador

## Regeneração a partir de Feedback do TSD
Se existirem arquivos `ai-spec/reviews/tsd_feedback_to_sdd_*.md` com status `pending`:
1. Ler cada conflito identificado
2. Aplicar a sugestão ao design afetado
3. Alterar o status do conflito no arquivo de review para `applied`

## Restrições
- Máximo de 1000 linhas por documento
- Formato Markdown apenas
- Diagramas apenas em formato Mermaid
- Não contradizer definições do `01_platform_sdd.md` nos demais documentos
- Cada REQ deve gerar ao menos um DES (cobertura completa)
- Manter terminologia consistente com os PRDs e BRDs

## Validação Pós-Geração
Após gerar todos os SDDs, validar:
- [ ] Todos os REQ-NN-SSS dos PRDs possuem ao menos um DES correspondente
- [ ] Todos os IDs DES-NN-SSS são únicos e sequenciais
- [ ] Nenhum documento de domínio contradiz `01_platform_sdd.md`
- [ ] Stack do produto em `02_product_sdd.md` não contradiz `01_platform_sdd.md`
- [ ] Conflitos de viabilidade geraram arquivos de feedback em `ai-spec/reviews/`
- [ ] Cada funcionalidade possui diagrama mermaid de arquitetura
- [ ] Frontmatter YAML presente e correto em cada documento
- [ ] Campo `derives_from` aponta para PRD existente
- [ ] Numeração do documento corresponde ao PRD de origem
- [ ] Nenhum documento excede 1000 linhas
