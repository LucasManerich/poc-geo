# /generate-tsd — Gerar Technical Specification Documents

## Contexto
Você é um Arquiteto de Sistema experiente trabalhando junto com DevOps e Desenvolvedores. Sua tarefa é gerar os documentos TSD (Technical Specification Document) derivados dos SDDs existentes, especificando implementação e deploy.

## Inputs
Leia obrigatoriamente os seguintes arquivos antes de iniciar a geração:
1. `ai-spec/sdd/*.md` — Todos os SDDs existentes (input principal)
2. `ai-spec/prd/*.md` — Todos os PRDs (para grafo de dependências)
3. `ai-spec/brd/*.md` — BRDs (contexto de negócio)
4. `ai-spec/tsd/01_platform_tsd.md` — Restrições de plataforma (se já existir, para não contradizer)

## Outputs
Gerar os seguintes documentos em `ai-spec/tsd/`:

### Documentos obrigatórios:
- `01_platform_tsd.md` — Deploy/implantação mínimos para todos os produtos
- `02_product_tsd.md` — Deploy/implantação do produto

### Documentos por domínio:
- `03_domain_<nome>_tsd.md` — Derivado do SDD de mesmo número
- A numeração DEVE corresponder ao SDD de origem

### Feedbacks (se conflitos detectados):
- `ai-spec/reviews/tsd_feedback_to_sdd_NN.md` — Um arquivo por SDD com conflitos

## Regras de Geração

### Frontmatter YAML obrigatório em cada documento:
```yaml
---
document_type: tsd
domain: <nome_do_dominio>
derives_from: sdd/<nome_do_sdd_origem>.md
file: tsd/<nome_do_arquivo>.md
---
```

### IDs de implementação:
- Prefixo: `IMP-`
- Formato: `IMP-NN-SSS` onde NN = número do documento, SSS = sequencial
- Cada IMP DEVE referenciar o DES pai: `derives: DES-NN-SSS`
- Exemplo: `IMP-03-001 (derives: DES-03-001)`

### Conteúdo do `01_platform_tsd.md`:
1. **Stages de deploy:**
   - Ambientes obrigatórios (dev, staging, production)
   - Critérios de promoção entre stages
2. **Estratégias e diretrizes de deploy:**
   - Clouds utilizadas
   - Uso de serviços SaaS, PaaS, IaaS
   - Estratégias de deploy (blue-green, canary, rolling)
3. **Componentes de arquitetura:**
   - Permitidos (lista com justificativa)
   - Negados (lista com justificativa)

### Conteúdo do `02_product_tsd.md`:
Todos os campos abaixo são **opcionais** — incluir apenas os que se aplicam ao produto:

1. **4.1 Infraestrutura do produto:**
   - Serviços de cloud utilizados (compute, storage, messaging)
   - Dimensionamento inicial (sizing) e critérios de scaling
   - Estimativa de custo operacional por ambiente

2. **4.2 Configuração por ambiente:**
   - Definição dos ambientes (dev, staging, production)
   - Variáveis de ambiente e secrets (estrutura, não valores)
   - Feature flags e configurações dinâmicas

3. **4.3 Pipeline de CI/CD:**
   - Etapas do pipeline (build, test, scan, deploy)
   - Estratégia de deploy (blue-green, canary, rolling)
   - Critérios de rollback automático

4. **4.4 Schema de dados:**
   - Modelo físico do banco de dados
   - Estratégia de migrations e versionamento de schema
   - Políticas de backup e retenção

5. **4.5 Especificações de API:**
   - Estrutura de versionamento (URL path, header)
   - Catálogo de endpoints por domínio
   - Contratos de request/response (referência OpenAPI)
   - Rate limiting e throttling por endpoint

6. **4.6 Integrações externas:**
   - Mapa de dependências externas
   - Circuit breaker e fallback por integração
   - SLAs esperados de cada dependência

7. **4.7 Observabilidade:**
   - Estratégia de logging (estruturado, níveis, retenção)
   - Métricas de aplicação e negócio a coletar
   - Tracing distribuído (se aplicável)
   - Alertas críticos e playbooks de resposta

8. **4.8 Estratégias de teste:**
   - Unitários: cobertura mínima, ferramentas
   - Integração: escopo, ambientes, dados de teste
   - E2E: cenários críticos
   - Carga/performance: critérios de aceite

9. **4.9 Segurança operacional:**
   - Gestão de secrets e rotação
   - Scanning de vulnerabilidades (SAST/DAST)
   - Políticas de acesso aos ambientes

### Conteúdo dos documentos de domínio `03+_domain_<nome>_tsd.md`:
Para cada item de design derivado do SDD:
1. **ID e título:** `IMP-NN-SSS — Título` (derives: DES-NN-SSS)
2. **Especificação de API:**
   - Endpoint, método HTTP, path
   - Headers obrigatórios
   - Request body (schema com tipos)
   - Response body (schema com tipos)
   - Códigos de erro
   - Exemplos de request/response
3. **Schema de dados:**
   - Tabelas/collections envolvidas
   - Campos com tipos, constraints, índices
   - Migrations necessárias
4. **Testes:**
   - Cenários de teste unitário (inputs, outputs esperados)
   - Cenários de teste de integração
   - Dados de teste necessários
5. **Deploy:**
   - Configurações específicas
   - Variáveis de ambiente necessárias
   - Dependências de infraestrutura

## Validação de Viabilidade de Implementação (Feedback Loop)
Durante a geração, para CADA item de design do SDD:
1. Avaliar se a implementação é viável com a infraestrutura definida
2. Avaliar se os schemas são consistentes entre domínios
3. Avaliar se as APIs são consistentes (versionamento, padrões)

### Se um design for INVIÁVEL:
1. Gerar o item IMP correspondente no TSD com anotação `⚠️ CONFLITO`
2. Criar arquivo de feedback:

```markdown
---
review_type: tsd_to_sdd
source_doc: tsd/<arquivo_tsd>.md
target_doc: sdd/<arquivo_sdd>.md
generated_at: <timestamp_ISO8601>
status: pending
---

## Conflitos Identificados

### CONFLICT-001
- **Design:** DES-NN-SSS
- **Problema:** <descrição técnica do problema>
- **Sugestão:** <alternativa viável proposta>
- **Impacto:** alto | médio | baixo
- **Decisão:** pending
```

3. Continuar gerando os demais itens normalmente
4. Ao final, listar todos os conflitos encontrados para o operador

## Restrições
- Máximo de 1000 linhas por documento
- Formato Markdown apenas
- Diagramas apenas em formato Mermaid
- Não contradizer definições do `01_platform_tsd.md` nos demais documentos
- Não contradizer definições dos SDDs (se houver conflito, gerar feedback)
- Cada DES deve gerar ao menos um IMP (cobertura completa)
- Manter terminologia consistente com os SDDs, PRDs e BRDs
- Todos os campos do `02_product_tsd.md` são opcionais

## Validação Pós-Geração
Após gerar todos os TSDs, validar:
- [ ] Todos os DES-NN-SSS dos SDDs possuem ao menos um IMP correspondente
- [ ] Todos os IDs IMP-NN-SSS são únicos e sequenciais
- [ ] Nenhum documento de domínio contradiz `01_platform_tsd.md`
- [ ] Conflitos de viabilidade geraram arquivos de feedback em `ai-spec/reviews/`
- [ ] Especificações de API são consistentes entre domínios (versionamento, padrões)
- [ ] Schemas de dados são consistentes entre domínios (sem conflitos de nomes, tipos)
- [ ] Frontmatter YAML presente e correto em cada documento
- [ ] Campo `derives_from` aponta para SDD existente
- [ ] Numeração do documento corresponde ao SDD de origem
- [ ] Nenhum documento excede 1000 linhas
