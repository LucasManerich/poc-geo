# Modelo de desenvolvimento de aplições orientadas por IA
Para a execução do fluxo a seguir, será sempre entregue um SUMÁRIO EXECUTIVO (como parâmetro de entrada) do produto NOVO ou feature de um produto JÁ EXISTENTE, onde devem ser construídos os documentos a seguir, conforme fluxo definido.

## Fluxo de Execução
1 - Os Gerentes de produto irão especificar documentos chamados BRDs;
2 - Após isso os analistas de negócio incluindo também os Gerentes de Produto, irão refinar a especificação derivando em sub-documentos chamados PRDs.
3 - Em seguida os Arquitetos de Produto, incluindo Tech Leads e Analistas de Sistema, revisam e refinam os PRDs, espeficicando os documentos SDDs, derivados;
4 - Por fim os Arquitetos de Sistema e profissionais de DEVOPS especificam os documentos TSDs, também derivados ods SDDs;

## Diretrizes gerais para a geração dos documentos
1 - O modelo se baseia em um fluxo de AI-Spec dividido em quatro etapas principais, cada uma gerando documentos em formato Markdown que serão utilizados nas próximas etapas;
2 - O fluxo é baseado em um modelo iterativo & incremental, onde feedbacks das fases seguintes podem ser utilizados como insumos para garantia da qualidade dos documentos gerados na fase anterior;
3 - Cada documento a ser gerado deverá estar no formato Markdow
4 - Cada documento a ser gerado poderá possuir máximo 1000 linhas
5 - Cada documento deverá ser gerado na seguinte estrutura de pastas:

  ai-spec/
  ai-spec/brd/
  ai-spec/brd/01_platform_brd.md
  ai-spec/brd/02_domain_X_brd.md

  ai-spec/prd/
  ai-spec/prd/01_platform_prd.md
  ai-spec/prd/02_general_prd.md
  ai-spec/prd/03_domain_x_prd.md

  ai-spec/sdd/
  ai-spec/sdd/01_platform_sdd.md
  ai-spec/sdd/02_general_sdd.md
  ai-spec/sdd/03_domain_x_sdd.md

  ai-spec/tsd/
  ai-spec/tsd/01_platform_tsd.md
  ai-spec/tsd/02_general_tsd.md
  ai-spec/tsd/03_domain_x_tsd.md

6 - Dentro do repositório do projeto deverá ser criada uma pasta chamada ai-spec, contendo subpastas com as documentações necessárias a implementação. As sub-pastas são: brd, prd, sdd e tsd
7 - Dentro de cada sub-pasta, os arquivos deverão ser numerados sequencialmente, iniciando do 01.
8 - Dentro de cada sub-pasta alguns arquivos padrões existirão e outros serão criados conforme domínios da aplicação
8.1 - Inicialmente os documentos 01_platform.md e 02_general.md, serão gerados por padrão contendo respectivamente as definições padrões de plataforma (genéricas a todos os produtos da empresa) e do produto (gerais para o produto em específico);
9 - A geração de documentos deverá ser realizada de tal modo que fiquem relacionados um ao outro, quando derivados. Então um BRD derivado para um PRD, deverá ter uma numeração sequencial igual, assim como outros documentos derivados;

## Documentos principais do modelo
1 - BRD - Business Requirment Document
2 - PRD - Product Design Document
3 - SDD - Software Design Document
4 - TSD - Technical Specification Document

### BRD
Analogia: WHY - O BRD define "O que fazer"
1 - Apresenta um sumário executivo, incluindo objetivos do negócio, escopo contemplado e itens que não estão contemplados no escopo;
2 - Define personas envolvidas (atores) no negócio;
3 - Apresenta presunções e riscos inerentes ao produto esperado;
4 - Apresenta exemplos de uso e documentos de referência para pesquisa;

5 - Deverá existir sempre ao menos um documento BRD por domínio dentro do produto;
6 - Os domínios do produto deverão ser estruturados de acordo com a metodologia de desenvolvimento DDD;

6 - O documento 01_platform_brd.md deve descrever elementos de negócio para todos os produtos da empresa, contendo:
6.1 - Definições de regras de dados como LGPD, incluindo anonimização de dados, uso de dados sensíveis;
6.2 - Definições sobre regras legais de copyrigth; 
6.3 - Restrições corportativas a nível de governança e mecanismos de controle;

7 - O documento 02_product_brd.md deve descrever elementos de negócio para produto em específico:
7.1 - Legislação própria aplicada ao produto, como prestação de contas para entidades de controle;
7.2 - Regulamentos e normas que se aplicam ao produto em esfera institucional;


### PRD
Analogia: HOW - Este representa a lista de features de desejos que se espera liberar em cada iteração
1 - Apresenta um documento de requisitos derivado da especificação do BRD
2 - Um BRD pode ter N PRDs derivadas, dependendo do tamanho do escopo de cada elemento
3 - Descreve um conjunto de requisitos funcionais
4 - Descreve um conjunto de requisitos não funcionais
5 - Descreve um conjunto de critérios de aceite para cada requisito funcional
6 - Descreve fluxos do usuário esperados dentro do sistema
7 - Descreve conjunto de comportamentos específicos

8 - O documento 01_platform_prd.md deve descrever elementos de requisitos de aplicação de todos os produtos da empresa, contendo:
8.1 - Requisitos funcionais como gestão de acessos, usuários, relatórios;
8.2 - Requisitos não funcionais como segurança, performance, usabilidade, manutenibilidade;

9 - O documento 02_product_prd.md deve descrever elementos de requisitos de aplicação em específico para o produto:
9.1 - Requisitos funcionais simples que não estejam descritos em outros PRDs derivados dos domínios do produto;


### SDD - Software Design Document
Analogia: Descreve o que deve ser construído, é o projeto arquitetural do produto 

1 - O documento 01_platform_sdd.md deve descrever elementos de tecnologia para todos os produtos da empresa, contendo:
1.1 - Definições de plataforma genéricas para todos os produtos;
1.2 - Considerações de stack tecnológica que devem ser impostas; 
1.3 - Considerações de deploy que devem ser impostas para todos os produtos da empresa;
1.4 - Premissas arquiteturais como boas práticas de desenvolvimento, documentação ativa no código;

2 - O documento 02_product_sdd.md deve descrever elementos para o produto em específico:
2.1 - Estrutura de domínios dentro do produto (de todos os domínios);
2.2 - Stack tecnológica de desenvolvimento do produto, sem contradizer definições de stack descritas no documento 01_platform_sdd.md;
2.3 - Considerações de deploy que devem ser seguidas para o produto, sem contradizer definições no documento 01_platform_sdd.md;
2.4 - Premissas arquiteturais como Design patterns, code smels;

3 - Descreve uma visão geral de cada funcionalidade a ser construída;
4 - É derivado de uma especificação contida em um documento PRD;
5 - Considerações de segurança aplicáveis a cada funcionalidade;

## TSD
Analogia: Descreve a implementação / deploy da aplicação
1 - Apresenta a stack de tecnologia a ser aplicada com base nos documentos SDDs
2 - Determina as especificações de APIs;

3 - O documento 01_platform_tsd.md deve descrever elementos de deploy/implantação mínimos para todos os produtos da empresa, contendo:
3.1 - Stages de deploy;
3.2 - Estratégias e diretrizes de deploy como clouds utiizadas, uso de serviços SAAS, PAAS, IAAS
3.3 - Componentes de arquitetura permissíveis ou negados;

4 - O documento 02_product_tsd.md deve descrever elementos deploy/implantação para o produto em específico:
4.1 - 

5 - Determinar o schema de dados;
6 - Estratégias de teste white box - unitário
7 - Estratégias de teste black box - integração
8 - Estratégias de verificação e validação
9 - Especificações de API, como estrutura de versionamento e rotas