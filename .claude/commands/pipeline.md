# /pipeline — Execução Completa do Fluxo AI-Spec

## Contexto
Este command executa o fluxo completo de geração de documentos: BRD → PRD → SDD → TSD → Implementação. O pipeline para automaticamente quando detecta conflitos entre fases, gerando relatório e aguardando decisão do operador.

## Pré-requisito
O arquivo `ai-spec/inputs/executive_summary.md` deve existir e estar preenchido.

## Fluxo de Execução

### FASE 1 — Geração dos BRDs
1. Executar a lógica descrita em `/generate-brd`
2. Informar ao operador: "BRDs gerados. Deseja revisar antes de prosseguir para PRDs?"
3. Se operador aprovar → prosseguir para Fase 2
4. Se operador solicitar ajustes → aplicar e regenerar BRDs

### FASE 2 — Geração dos PRDs
1. Executar a lógica descrita em `/generate-prd`
2. Validar grafo de dependências (ausência de ciclos)
3. Informar ao operador: "PRDs gerados. Deseja revisar antes de prosseguir para SDDs?"
4. Se operador aprovar → prosseguir para Fase 3

### FASE 3 — Geração dos SDDs
1. Executar a lógica descrita em `/generate-sdd`
2. **GATE DE CONFLITOS:** Se conflitos foram detectados:
   a. Apresentar relatório de conflitos ao operador
   b. Para cada conflito, perguntar: "Aceitar sugestão ou rejeitar?"
   c. Se aceitar → marcar como `applied`, regenerar PRD afetado, regenerar SDD
   d. Se rejeitar → marcar como `rejected`, manter SDD com anotação
   e. Repetir até todos os conflitos serem resolvidos
3. Se sem conflitos → prosseguir para Fase 4

### FASE 4 — Geração dos TSDs
1. Executar a lógica descrita em `/generate-tsd`
2. **GATE DE CONFLITOS:** Mesmo mecanismo da Fase 3
   a. Se conflitos → apresentar, resolver, regenerar SDDs afetados
   b. Se sem conflitos → prosseguir para Fase 5
3. Após resolução, validar consistência com SDDs regenerados

### FASE 5 — Implementação
1. Informar ao operador: "Todas as especificações foram geradas e validadas. Deseja executar a implementação?"
2. Se operador aprovar → executar a lógica descrita em `/execute-tsd`
3. Se operador negar → finalizar pipeline na documentação

## Relatório Final
Ao concluir, apresentar resumo:

```
═══════════════════════════════════════════════════════════════
                   PIPELINE AI-SPEC — RELATÓRIO FINAL
═══════════════════════════════════════════════════════════════

Documentos gerados:
  BRDs: X arquivos
  PRDs: X arquivos
  SDDs: X arquivos
  TSDs: X arquivos

Rastreabilidade:
  Requisitos de negócio (BIZ): X
  Requisitos funcionais (REQ): X
  Especificações de design (DES): X
  Especificações de implementação (IMP): X

Conflitos resolvidos:
  SDD → PRD: X aplicados, X rejeitados
  TSD → SDD: X aplicados, X rejeitados

Reviews gerados: X arquivos em ai-spec/reviews/

Implementação: [executada | não executada]
═══════════════════════════════════════════════════════════════
```

## Restrições
- O pipeline SEMPRE para nos gates de conflito — nunca resolve automaticamente
- O operador tem a decisão final sobre aceitar ou rejeitar cada conflito
- Se o operador cancelar o pipeline em qualquer fase, os documentos já gerados são mantidos
- O pipeline pode ser retomado manualmente executando os commands individuais a partir da fase onde parou
