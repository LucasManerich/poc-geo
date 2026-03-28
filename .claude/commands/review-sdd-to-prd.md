# /review-sdd-to-prd — Review de Consistência SDD vs PRD

## Contexto
Você é um Arquiteto de Software realizando uma revisão de consistência entre os documentos SDD e PRD. Seu objetivo é identificar conflitos, requisitos órfãos e inconsistências entre as duas fases.

## Inputs
Leia obrigatoriamente:
1. `ai-spec/sdd/*.md` — Todos os SDDs
2. `ai-spec/prd/*.md` — Todos os PRDs

## Análises a Realizar

### 1. Cobertura de Requisitos
Para cada REQ-NN-SSS nos PRDs, verificar:
- Existe ao menos um DES-NN-SSS correspondente nos SDDs?
- Se NÃO → registrar como **REQUISITO SEM DESIGN** (gap)

Para cada DES-NN-SSS nos SDDs, verificar:
- O REQ referenciado em `derives` existe no PRD correspondente?
- Se NÃO → registrar como **DESIGN ÓRFÃO** (inconsistência)

### 2. Viabilidade Técnica
Para cada REQ que possui DES correspondente:
- O design proposto atende aos critérios de aceite do requisito?
- Os requisitos não funcionais (performance, segurança) são endereçados?
- As integrações descritas no PRD são possíveis com a arquitetura do SDD?

### 3. Consistência de Dados
- Modelos de dados nos SDDs são consistentes com os requisitos dos PRDs?
- Campos obrigatórios descritos nos use cases estão presentes nos schemas?

### 4. Consistência de Stack
- `02_product_sdd.md` não contradiz `01_platform_sdd.md`?
- Bibliotecas adicionais no produto são compatíveis com a stack de plataforma?

## Output
Gerar arquivos de feedback em `ai-spec/reviews/` para cada PRD com conflitos:

```markdown
---
review_type: sdd_to_prd
source_doc: sdd/<arquivo_sdd>.md
target_doc: prd/<arquivo_prd>.md
generated_at: <timestamp_ISO8601>
status: pending
---

## Resumo
- Total de requisitos analisados: X
- Requisitos cobertos: X
- Requisitos sem design (gaps): X
- Designs órfãos: X
- Conflitos de viabilidade: X

## Gaps — Requisitos sem Design

### GAP-001
- **Requisito:** REQ-NN-SSS — "Título"
- **Observação:** <motivo provável da ausência>

## Designs Órfãos

### ORPHAN-001
- **Design:** DES-NN-SSS — "Título"
- **Referencia:** REQ-NN-SSS (não encontrado no PRD)

## Conflitos de Viabilidade

### CONFLICT-001
- **Requisito:** REQ-NN-SSS
- **Problema:** <descrição>
- **Sugestão:** <alternativa>
- **Impacto:** alto | médio | baixo
- **Decisão:** pending
```

Se NENHUM conflito, gap ou inconsistência for encontrado, informar ao operador:
"Review concluído. Nenhuma inconsistência encontrada entre SDDs e PRDs."

## Restrições
- Este command NÃO altera nenhum documento existente
- Apenas gera relatórios de review em `ai-spec/reviews/`
- O operador decide as ações a tomar com base no relatório
