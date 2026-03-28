# /review-tsd-to-sdd — Review de Consistência TSD vs SDD

## Contexto
Você é um Arquiteto de Sistema realizando uma revisão de consistência entre os documentos TSD e SDD. Seu objetivo é identificar conflitos, designs sem implementação e inconsistências entre as duas fases.

## Inputs
Leia obrigatoriamente:
1. `ai-spec/tsd/*.md` — Todos os TSDs
2. `ai-spec/sdd/*.md` — Todos os SDDs

## Análises a Realizar

### 1. Cobertura de Designs
Para cada DES-NN-SSS nos SDDs, verificar:
- Existe ao menos um IMP-NN-SSS correspondente nos TSDs?
- Se NÃO → registrar como **DESIGN SEM IMPLEMENTAÇÃO** (gap)

Para cada IMP-NN-SSS nos TSDs, verificar:
- O DES referenciado em `derives` existe no SDD correspondente?
- Se NÃO → registrar como **IMPLEMENTAÇÃO ÓRFÃ** (inconsistência)

### 2. Viabilidade de Implementação
Para cada DES que possui IMP correspondente:
- A infraestrutura definida suporta a implementação?
- Os schemas de dados são realizáveis com o banco escolhido?
- As APIs especificadas são consistentes com a arquitetura?

### 3. Consistência de Stack
- Stack de implementação no TSD é compatível com o SDD?
- Não há bibliotecas no TSD que conflitem com as definidas no SDD?

### 4. Consistência de APIs
- Endpoints nos TSDs de diferentes domínios não conflitam (paths duplicados)?
- Versionamento de API é consistente entre domínios?
- Schemas de request/response são compatíveis com os contratos do SDD?

### 5. Consistência de Dados
- Schemas físicos (TSD) são compatíveis com modelos de domínio (SDD)?
- Foreign keys e relações entre tabelas de domínios diferentes são consistentes?

## Output
Gerar arquivos de feedback em `ai-spec/reviews/` para cada SDD com conflitos:

```markdown
---
review_type: tsd_to_sdd
source_doc: tsd/<arquivo_tsd>.md
target_doc: sdd/<arquivo_sdd>.md
generated_at: <timestamp_ISO8601>
status: pending
---

## Resumo
- Total de designs analisados: X
- Designs cobertos: X
- Designs sem implementação (gaps): X
- Implementações órfãs: X
- Conflitos de viabilidade: X

## Gaps — Designs sem Implementação

### GAP-001
- **Design:** DES-NN-SSS — "Título"
- **Observação:** <motivo provável da ausência>

## Implementações Órfãs

### ORPHAN-001
- **Implementação:** IMP-NN-SSS — "Título"
- **Referencia:** DES-NN-SSS (não encontrado no SDD)

## Conflitos de Viabilidade

### CONFLICT-001
- **Design:** DES-NN-SSS
- **Problema:** <descrição>
- **Sugestão:** <alternativa>
- **Impacto:** alto | médio | baixo
- **Decisão:** pending
```

Se NENHUM conflito, gap ou inconsistência for encontrado, informar ao operador:
"Review concluído. Nenhuma inconsistência encontrada entre TSDs e SDDs."

## Restrições
- Este command NÃO altera nenhum documento existente
- Apenas gera relatórios de review em `ai-spec/reviews/`
- O operador decide as ações a tomar com base no relatório
