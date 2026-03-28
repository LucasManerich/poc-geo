# /generate-brd — Gerar Business Requirement Documents

## Contexto
Você é um Gerente de Produto experiente. Sua tarefa é gerar os documentos BRD (Business Requirement Document) a partir do Sumário Executivo do projeto.

## Inputs
Leia obrigatoriamente os seguintes arquivos antes de iniciar a geração:
1. `ai-spec/inputs/executive_summary.md` — Sumário Executivo (input principal)
2. `ai-spec/inputs/diagrams/*.mermaid` — Diagramas de entrada (se existirem)
3. `ai-spec/inputs/references/*.md` — Referências e benchmarks (se existirem)

## Outputs
Gerar os seguintes documentos em `ai-spec/brd/`:

### Documentos obrigatórios:
- `01_platform_brd.md` — Regras de negócio corporativas para todos os produtos
- `02_product_brd.md` — Regras de negócio específicas do produto

### Documentos por domínio:
- `03_domain_<nome>_brd.md` — Um documento por bounded context identificado (DDD)
- Numerar sequencialmente a partir de 03

## Regras de Geração

### Frontmatter YAML obrigatório em cada documento:
```yaml
---
document_type: brd
domain: <nome_do_dominio>
file: brd/<nome_do_arquivo>.md
---
```

### IDs de requisitos de negócio:
- Prefixo: `BIZ-`
- Formato: `BIZ-NN-SSS` onde NN = número do documento, SSS = sequencial
- Exemplo: `BIZ-03-001`, `BIZ-03-002`

### Conteúdo do `01_platform_brd.md`:
1. Definições de regras de dados (LGPD, termo de uso e recurso de aceite do termo de uso, anonimização, dados sensíveis, políticas de privacidade)
2. Definições sobre regras legais de copyright
3. Restrições corporativas de governança e mecanismos de controle

### Conteúdo do `02_product_brd.md`:
1. Sumário executivo do produto (derivado do input)
2. Objetivos de negócio e KPIs (se fornecidos no sumário)
3. Legislação própria aplicada ao produto
4. Regulamentos e normas aplicáveis ao produto
5. Escopo incluído e excluído
6. Personas/atores do negócio
7. Presunções e riscos inerentes ao produto
8. Mapa de domínios DDD identificados (lista todos os bounded contexts)

### Conteúdo dos documentos de domínio `03+_domain_<nome>_brd.md`:
1. Descrição do bounded context
2. Requisitos de negócio do domínio (com IDs BIZ-NN-SSS)
3. Personas envolvidas neste domínio
4. Presunções e riscos específicos
5. Exemplos de uso
6. Referências e benchmarks aplicáveis
7. Diagramas mermaid relevantes ao domínio

### Identificação de Bounded Contexts (DDD):
- Analisar o Sumário Executivo para identificar domínios distintos
- Cada domínio deve ter responsabilidades coesas e fronteiras claras
- Considerar: linguagem ubíqua, agregados, serviços de domínio
- Documentar as relações entre domínios (Context Map)

## Restrições
- Máximo de 1000 linhas por documento
- Se um domínio exceder esse limite, decompor em sub-domínios
- Formato Markdown apenas
- Diagramas apenas em formato Mermaid
- Não inventar requisitos que não estejam implícitos ou explícitos no Sumário Executivo
- Manter consistência de terminologia com o Sumário Executivo
- Não presuma nenhuma informação, EM CASO DE DÚVIDAS SEMPRE PERGUNTE

## Validação Pós-Geração
Após gerar todos os BRDs, validar:
- [ ] Todos os itens do escopo do Sumário Executivo estão cobertos por ao menos um BRD
- [ ] Todos os IDs BIZ-NN-SSS são únicos e sequenciais
- [ ] Todos os bounded contexts identificados possuem documento próprio
- [ ] O mapa de domínios no `02_product_brd.md` lista todos os domínios gerados
- [ ] Frontmatter YAML presente e correto em cada documento
- [ ] Nenhum documento excede 1000 linhas
