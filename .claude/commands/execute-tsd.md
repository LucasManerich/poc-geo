# /execute-tsd — Implementação Completa a partir dos TSDs

## Contexto
Você é um Desenvolvedor Sênior e Arquiteto de Sistema. Sua tarefa é implementar o código completo da aplicação a partir das especificações nos documentos TSD, SDD e PRD. Este command tem liberdade total para criar e alterar qualquer arquivo no contexto do projeto.

## Inputs
Leia obrigatoriamente os seguintes arquivos antes de iniciar:
1. `ai-spec/tsd/*.md` — Todos os TSDs (input principal — especificações de implementação)
2. `ai-spec/sdd/*.md` — Todos os SDDs (arquitetura e design)
3. `ai-spec/prd/*.md` — Todos os PRDs (requisitos e grafo de dependências)
4. `ai-spec/brd/*.md` — BRDs (contexto de negócio)

## Ordem de Execução

### 1. Resolver ordem via grafo de dependências
- Ler o grafo de dependências dos PRDs (campo `depends_on` nos requisitos)
- Realizar ordenação topológica (topological sort) do grafo
- Validar ausência de ciclos (se houver, PARAR e informar operador)
- Executar implementação na ordem resultante: requisitos sem dependência primeiro, depois os que dependem deles, e assim por diante

### 2. Para cada requisito na ordem topológica

Localizar o IMP correspondente no TSD e implementar:

#### a) Infraestrutura e configuração
- Criar/atualizar arquivos de configuração (docker-compose, env templates, etc.)
- Criar/atualizar configurações de CI/CD se especificadas no TSD
- Criar/atualizar variáveis de ambiente (estrutura, não valores sensíveis)

#### b) Schema de dados
- Criar migrations de banco de dados conforme especificado no TSD
- Respeitar tipos, constraints e índices definidos
- Manter consistência com migrations já existentes

#### c) Modelo de domínio
- Implementar entidades, value objects e agregados conforme SDD
- Implementar regras de validação conforme PRD (critérios de aceite)
- Seguir design patterns permitidos em `02_product_sdd.md`
- Evitar code smells listados em `02_product_sdd.md`

#### d) Lógica de negócio
- Implementar services, use cases e handlers
- Implementar regras de negócio conforme BRD e PRD
- Implementar validações de entrada conforme especificado
- Implementar tratamento de erros com códigos definidos no TSD

#### e) APIs
- Implementar endpoints conforme especificação do TSD
- Respeitar contratos de request/response
- Implementar rate limiting e throttling se especificados
- Implementar versionamento de API conforme padrão definido

#### f) Integrações
- Implementar clients para APIs externas conforme TSD seção 4.6
- Implementar circuit breaker e fallback quando especificados
- Implementar retry e error handling para integrações

#### g) Testes
- Implementar testes unitários para cada cenário descrito no TSD
- Implementar testes de integração para cenários descritos no TSD
- Criar fixtures e dados de teste necessários
- Respeitar cobertura mínima definida no TSD seção 4.8

#### h) Observabilidade
- Implementar logging estruturado conforme TSD seção 4.7
- Implementar métricas de aplicação e negócio
- Implementar tracing quando especificado

### 3. Validação pós-implementação
Após implementar todos os requisitos:
- Executar linting do código
- Executar testes unitários
- Verificar que todas as migrations estão válidas
- Verificar que todas as APIs implementadas correspondem ao TSD

## Regras de Implementação

### Aderência à Stack
- Utilizar APENAS tecnologias definidas em `01_platform_sdd.md` e `02_product_sdd.md`
- Se uma biblioteca adicional for necessária e não estiver listada, adicionar comentário `// TODO: validar biblioteca X com arquiteto`

### Qualidade de Código
- Seguir boas práticas definidas em `01_platform_sdd.md`
- Documentação ativa no código (JSDoc, docstrings, etc.) conforme definido
- Respeitar design patterns permitidos e evitar proibidos
- Evitar code smells listados

### Rastreabilidade no Código
- Cada arquivo/módulo principal deve conter em comentário o ID do IMP que o originou
- Exemplo: `// IMP-03-001 — Importação de extratos OFX/CSV`
- Isso permite rastrear código de volta à especificação

### Estrutura de Pastas do Código
- Seguir a estrutura de domínios definida em `02_product_sdd.md`
- Cada bounded context deve ter seu próprio diretório/módulo
- Código compartilhado (shared kernel) em diretório próprio

## Relatório de Execução
Ao concluir, apresentar:

```
═══════════════════════════════════════════════════════════════
              EXECUTE-TSD — RELATÓRIO DE IMPLEMENTAÇÃO
═══════════════════════════════════════════════════════════════

Requisitos implementados: X / Y total
  ✅ IMP-03-001 — Importação de extratos OFX/CSV
  ✅ IMP-03-002 — Motor de matching 1:1
  ✅ IMP-03-003 — Motor de matching 1:N
  ...

Arquivos criados: X
Arquivos modificados: X

Migrations criadas: X
Testes criados: X (unitários: X, integração: X)
Endpoints implementados: X

Pendências (TODOs):
  ⚠️ [IMP-03-005] Validar biblioteca X com arquiteto
  ...

Testes executados:
  ✅ Unitários: X passed, X failed
  ✅ Integração: X passed, X failed
═══════════════════════════════════════════════════════════════
```

## Restrições
- Este command pode criar e alterar QUALQUER arquivo no contexto do projeto
- Sem restrição aditiva — sobrescrita de arquivos existentes é permitida
- O operador DEVE revisar o código gerado após execução
- Não incluir valores sensíveis (secrets, passwords, tokens) — apenas estrutura
- Seguir estritamente a stack definida nos documentos de plataforma
