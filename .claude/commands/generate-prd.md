# /generate-prd — Gerar Product Requirement Documents

## Contexto
Você é um Analista de Negócio experiente trabalhando junto com Gerentes de Produto. Sua tarefa é gerar os documentos PRD (Product Requirement Document) derivados dos BRDs existentes.

## Inputs
Leia obrigatoriamente os seguintes arquivos antes de iniciar a geração:
1. `ai-spec/brd/*.md` — Todos os BRDs existentes (input principal)
2. `ai-spec/inputs/executive_summary.md` — Sumário Executivo (contexto)
3. `ai-spec/inputs/diagrams/*.mermaid` — Diagramas de entrada (se existirem)
4. `ai-spec/reviews/sdd_feedback_to_prd_*.md` — Feedbacks pendentes do SDD (se existirem, para regeneração)

## Outputs
Gerar os seguintes documentos em `ai-spec/prd/`:

### Documentos obrigatórios:
- `01_platform_prd.md` — Requisitos aplicáveis a todos os produtos
- `02_product_prd.md` — Requisitos gerais do produto

### Documentos por domínio:
- `03_domain_<nome>_prd.md` — Derivado do BRD de mesmo número
- A numeração DEVE corresponder ao BRD de origem (BRD 03 → PRD 03)

## Regras de Geração

### Frontmatter YAML obrigatório em cada documento:
```yaml
---
document_type: prd
domain: <nome_do_dominio>
derives_from: brd/<nome_do_brd_origem>.md
file: prd/<nome_do_arquivo>.md
requirements:
  - id: REQ-NN-SSS
    title: "Título do requisito"
    depends_on: []
  - id: REQ-NN-SSS
    title: "Título do requisito"
    depends_on: [REQ-NN-SSS]
---
```

### IDs de requisitos:
- Prefixo: `REQ-`
- Formato: `REQ-NN-SSS` onde NN = número do documento, SSS = sequencial
- Cada REQ DEVE referenciar o BIZ pai: `derives: BIZ-NN-SSS`
- Exemplo: `REQ-03-001 (derives: BIZ-03-001)`

### Grafo de dependências:
- Cada requisito declara `depends_on: []` com lista de IDs dos requisitos que precisam estar implementados antes
- O grafo NÃO pode conter ciclos — validar antes de finalizar
- Requisitos sem dependência recebem `depends_on: []`
- Dependências podem cruzar domínios (REQ-03-001 pode depender de REQ-04-002)

### Conteúdo do `01_platform_prd.md`:
1. Requisitos funcionais de plataforma:
   - Gestão de acessos e autenticação
   - Gestão de usuários e perfis
   - Relatórios e auditoria padrão
2. Requisitos não funcionais de plataforma:
   - Segurança
   - Performance
   - Usabilidade
   - Manutenibilidade
   - Disponibilidade

### Conteúdo do `02_product_prd.md`:
1. Requisitos funcionais simples que não pertencem a domínios específicos
2. Requisitos que são transversais a múltiplos domínios
3. Mapa consolidado de dependências entre todos os domínios

### Conteúdo dos documentos de domínio `03+_domain_<nome>_prd.md`:
Para cada requisito funcional:
1. **ID e título:** `REQ-NN-SSS — Título` (derives: BIZ-NN-SSS)
2. **Descrição:** o que o requisito entrega
3. **Use Case:**
   - Ator principal
   - Pré-condições
   - Fluxo principal (passo a passo)
   - Fluxos alternativos
   - Pós-condições
   - Exceções
4. **Critérios de aceite:** lista verificável de condições que devem ser atendidas
5. **Dependências:** `depends_on: [REQ-NN-SSS, ...]`

Para requisitos não funcionais do domínio:
1. **ID e título:** `REQ-NN-SSS — Título`
2. **Descrição e métrica esperada**
3. **Critério de aceite mensurável**

### Comportamentos específicos:
- Documentar regras de negócio complexas como tabelas de decisão
- Documentar validações de dados com exemplos de valores válidos/inválidos
- Documentar mensagens de erro esperadas

## Regeneração a partir de Feedback do SDD
Se existirem arquivos `ai-spec/reviews/sdd_feedback_to_prd_*.md` com status `pending`:
1. Ler cada conflito identificado
2. Aplicar a sugestão do SDD ao requisito afetado
3. Manter o requisito original como histórico (comentário markdown)
4. Atualizar o frontmatter com o requisito modificado
5. Alterar o status do conflito no arquivo de review para `applied`
6. Revalidar o grafo de dependências após alterações

## Restrições
- Máximo de 1000 linhas por documento
- Formato Markdown apenas
- Diagramas apenas em formato Mermaid
- Não criar requisitos que não derivem de um BIZ existente nos BRDs
- Cada BIZ deve gerar ao menos um REQ (cobertura completa)
- Manter terminologia consistente com os BRDs

## Validação Pós-Geração
Após gerar todos os PRDs, validar:
- [ ] Todos os BIZ-NN-SSS dos BRDs possuem ao menos um REQ correspondente
- [ ] Todos os IDs REQ-NN-SSS são únicos e sequenciais
- [ ] Grafo de dependências não contém ciclos
- [ ] Cada requisito funcional possui use case com pré/pós-condição
- [ ] Cada requisito funcional possui critérios de aceite
- [ ] Frontmatter YAML presente e correto em cada documento
- [ ] Campo `derives_from` aponta para BRD existente
- [ ] Numeração do documento corresponde ao BRD de origem
- [ ] Nenhum documento excede 1000 linhas
