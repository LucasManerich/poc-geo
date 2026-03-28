---
document_type: brd
domain: plataforma
file: brd/01_platform_brd.md
---

# BRD-01 — Regras de Negócio Corporativas da Plataforma

## 1. Objetivo

Este documento define as regras de negócio corporativas aplicáveis transversalmente a todos os módulos e municípios atendidos pela plataforma GEO. São regras de dados, privacidade, copyright e governança que devem ser respeitadas independentemente do produto ou funcionalidade específica.

---

## 2. Regras de Dados e Privacidade (LGPD)

### 2.1 Lei Geral de Proteção de Dados (Lei 13.709/2018)

| ID | Requisito |
|----|-----------|
| BIZ-01-001 | O sistema deve coletar apenas os dados pessoais estritamente necessários para a finalidade de cada operação (princípio da minimização). |
| BIZ-01-002 | Todo tratamento de dados pessoais deve possuir base legal documentada conforme Art. 7º da LGPD (consentimento, obrigação legal, execução de políticas públicas, etc.). |
| BIZ-01-003 | A plataforma deve manter registro das operações de tratamento de dados pessoais (Art. 37 da LGPD). |
| BIZ-01-004 | O sistema deve permitir que o titular de dados exerça seus direitos previstos no Art. 18 da LGPD (acesso, correção, anonimização, eliminação, portabilidade). |
| BIZ-01-005 | Dados pessoais sensíveis (origem racial/étnica, saúde, dados biométricos) devem receber tratamento diferenciado com controles adicionais de acesso e auditoria. |

### 2.2 Termo de Uso e Recurso de Aceite

| ID | Requisito |
|----|-----------|
| BIZ-01-006 | A plataforma deve exibir Termo de Uso ao cidadão no primeiro acesso e exigir aceite explícito antes de permitir interações que envolvam dados pessoais. |
| BIZ-01-007 | O aceite do Termo de Uso deve ser registrado com data/hora, versão do termo e identificação do usuário. |
| BIZ-01-008 | A plataforma deve permitir atualização do Termo de Uso e exigir novo aceite quando houver alteração substancial. |
| BIZ-01-009 | Para acesso anônimo (sem login), o sistema deve informar as condições de uso e os dados coletados (ex.: IP, navegador) via banner de privacidade ou equivalente. |

### 2.3 Anonimização e Dados Sensíveis

| ID | Requisito |
|----|-----------|
| BIZ-01-010 | Dados utilizados para indicadores e relatórios agregados devem ser anonimizados, impossibilitando a identificação do titular. |
| BIZ-01-011 | Exportações de dados em massa (CSV, Excel, TXT) para usuários sem perfil administrativo devem omitir ou mascarar dados pessoais sensíveis (CPF, nome completo do proprietário). |
| BIZ-01-012 | O sistema deve implementar mecanismo de pseudonimização para logs e trilhas de auditoria quando necessário. |

### 2.4 Política de Privacidade

| ID | Requisito |
|----|-----------|
| BIZ-01-013 | A plataforma deve disponibilizar Política de Privacidade acessível a partir de qualquer página, descrevendo quais dados são coletados, finalidade, tempo de retenção e formas de contato com o Encarregado de Dados (DPO). |
| BIZ-01-014 | Cada município deve poder personalizar a Política de Privacidade com suas informações específicas (nome do Encarregado, canal de atendimento, endereço). |

---

## 3. Regras Legais de Copyright e Propriedade Intelectual

| ID | Requisito |
|----|-----------|
| BIZ-01-015 | Dados cartográficos e imagens de satélite utilizados na plataforma devem respeitar as licenças de uso dos provedores originais (Google, Esri, provedores de imagens). |
| BIZ-01-016 | Documentos e certidões gerados pela plataforma são de titularidade do município emissor; o sistema deve incluir identificação do município e da plataforma no rodapé dos documentos. |
| BIZ-01-017 | O cidadão deve ser informado de que a reprodução ou uso indevido de dados obtidos via plataforma pode constituir infração legal. |
| BIZ-01-018 | Dados cadastrais importados dos sistemas municipais (ERP) mantêm a titularidade do município de origem. |
| BIZ-01-019 | A plataforma não deve permitir download em massa de imagens de satélite ou bases cartográficas, respeitando restrições de redistribuição dos provedores. |

---

## 4. Governança Corporativa e Mecanismos de Controle

### 4.1 Controle de Acesso

| ID | Requisito |
|----|-----------|
| BIZ-01-020 | O sistema deve implementar controle de acesso baseado em perfis (RBAC), com permissões granulares por módulo, funcionalidade e setor do município. |
| BIZ-01-021 | Toda ação que modifique dados cadastrais, fiscais ou territoriais deve gerar registro de auditoria contendo: usuário, data/hora, ação realizada e valores anterior/posterior. |
| BIZ-01-022 | Perfis administrativos devem ser restritos; a criação de perfis com acesso total deve requerer aprovação de administrador de nível superior. |
| BIZ-01-023 | Sessões de usuário devem expirar após período configurável de inatividade. |

### 4.2 Segregação de Dados por Município (Multi-tenancy)

| ID | Requisito |
|----|-----------|
| BIZ-01-024 | A plataforma deve garantir isolamento completo dos dados entre municípios; nenhum município pode acessar dados de outro. |
| BIZ-01-025 | Configurações de camadas, documentos, zoneamento e perfis de acesso são independentes por município. |

### 4.3 Auditoria e Rastreabilidade

| ID | Requisito |
|----|-----------|
| BIZ-01-026 | Logs de auditoria devem ser imutáveis e retidos pelo período mínimo exigido pela legislação aplicável. |
| BIZ-01-027 | O sistema deve fornecer interface de consulta de logs de auditoria para administradores autorizados, com filtros por período, usuário e tipo de ação. |

### 4.4 Disponibilidade e Continuidade

| ID | Requisito |
|----|-----------|
| BIZ-01-028 | A plataforma deve possuir mecanismos de backup e recuperação de dados que garantam a continuidade operacional em caso de falha. |
| BIZ-01-029 | Dados geoespaciais e cadastrais devem ser passíveis de exportação integral pelo município em caso de rescisão contratual (portabilidade de dados). |
