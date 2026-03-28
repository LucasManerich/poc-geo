# AI-Spec — Modelo de Desenvolvimento Orientado por IA

## Quick Start

### 1. Preparar o Sumário Executivo
Edite o arquivo `ai-spec/inputs/executive_summary.md` com as informações do seu projeto/feature.

### 2. Executar o fluxo

**Modo granular (recomendado para primeira execução):**
```
/generate-brd     → Gera BRDs a partir do sumário executivo
/generate-prd     → Gera PRDs derivados dos BRDs
/generate-sdd     → Gera SDDs derivados dos PRDs (com validação de viabilidade)
/generate-tsd     → Gera TSDs derivados dos SDDs (com validação de viabilidade)
/execute-tsd      → Implementa código a partir dos TSDs
```

**Modo pipeline (execução contínua com gates):**
```
/pipeline          → Executa BRD → PRD → SDD → TSD → Implementação
                     Para automaticamente em conflitos
```

**Commands de review (opcionais):**
```
/review-sdd-to-prd → Análise de consistência SDD vs PRD
/review-tsd-to-sdd → Análise de consistência TSD vs SDD
```

## Estrutura de Pastas

```
ai-spec/
├── inputs/                  ← Artefatos de entrada
│   ├── executive_summary.md ← Sumário Executivo (input principal)
│   ├── diagrams/            ← Diagramas mermaid
│   └── references/          ← Benchmarks e referências
├── brd/                     ← Business Requirement Documents
├── prd/                     ← Product Requirement Documents
├── sdd/                     ← Software Design Documents
├── tsd/                     ← Technical Specification Documents
└── reviews/                 ← Feedbacks entre fases
    └── archive/             ← Reviews resolvidos
```

## Sistema de Rastreabilidade

Cada requisito possui um ID rastreável desde a origem até a implementação:

| Fase | Prefixo | Exemplo |
|------|---------|---------|
| BRD | `BIZ-` | `BIZ-03-001` |
| PRD | `REQ-` | `REQ-03-001` (derives: BIZ-03-001) |
| SDD | `DES-` | `DES-03-001` (derives: REQ-03-001) |
| TSD | `IMP-` | `IMP-03-001` (derives: DES-03-001) |

## Convenções

- Documentos numerados sequencialmente: `01_platform`, `02_product`, `03+_domain_*`
- Numeração consistente entre pastas (BRD 03 → PRD 03 → SDD 03 → TSD 03)
- Máximo de 1000 linhas por documento
- Diagramas apenas em formato Mermaid
- Frontmatter YAML obrigatório em cada documento
