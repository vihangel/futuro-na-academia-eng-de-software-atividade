# O Aplicativo do Futuro na Academia

Sua equipe foi contratada por uma grande universidade para projetar um novo aplicativo móvel integrado para alunos e professores. O app deve ser um hub central para a vida acadêmica, indo muito além de simplesmente disponibilizar notas e materiais.

A universidade quer ser referência em tecnologia e inovação, mas não sabe exatamente o que é possível fazer. Cabe à sua equipe definir os requisitos fundamentais deste aplicativo.

## Requisitos Funcionais 
- O sistema deve permitir que alunos consultem as disciplinas em que estão matriculados, com a possibilidade de visualizar detalhes da disciplina como horários, professores, atividades, notas e frequência.
- O sistema deve permitir que alunos consultem o seu histórico acadêmico.
- O sistema deve permitir que professores gerenciem informações de suas disciplinas, como plano de ensino, atividades e notas.
- O sistema deve permitir que professores cadastrem projetos de pesquisa e extensão vinculados à universidade.
- O sistema deve exibir aos alunos projetos de pesquisa e extensão vinculados à universidade, com informações como editais e vagas.
- O sistema deve permitir aos alunos a inscrição em projetos de pesquisa e extensão vinculados à universidade.
- O sistema deve disponibilizar relatórios e acompanhamento das atividades de pesquisa e extensão para alunos e professores.
- O sistema deve permitir que alunos façam reservas de espaços comuns (salas de aula, laboratórios, auditórios) de acordo com disponibilidade.
- O sistema deve permitir que técnicos de institutos validem os pedidos de reservas de espaços comuns (salas de aula, laboratórios, auditórios) dos alunos.
- O sistema deve permitir que usuários (professores, alunos ou técnicos) acionem um alerta de segurança via botão dedicado, enviando automaticamente a geolocalização atual (latitude e longitude via GPS) e dados de identificação da ocorrência (descrição textual breve, tipo de emergência selecionado e identificação do usuário) para a equipe de segurança do campus.
- O sistema deve registrar automaticamente a presença de usuários em atividades acadêmicas por meio de mecanismos inteligentes, como geofencing ou integração com equipamentos de coleta de dados, confirmando a localização e horário sem intervenção manual.
- O sistema deve permitir que usuários recarreguem créditos para o restaurante universitário (RU) via interface dedicada, processando transações e atualizando o saldo em tempo real.
- O sistema deve notificar a equipe de segurança em tempo real sobre alertas acionados, incluindo um mapa interativo com a geolocalização e detalhes da ocorrência.
- Os usuários do sistema devem ser capazes de gerar relatórios de presença, incluindo dados de frequência por usuário e atividade.
- Os usuários devem ser capazes de vizualizar o histórico de transações de recarga de RU para usuários, incluindo datas, valores e métodos de pagamento utilizados.

## Requisitos Não-Funcionais 
- O sistema deve integrar-se ao sistema de monitoramento do campus para compartilhar dados de alertas de segurança em formato padronizado (API RESTful), garantindo compatibilidade e sincronização em até 5 segundos. (Integração do sistema e desempenho)
- No módulo de recarga de RU, o sistema deve integrar um broker de pagamentos que suporte múltiplas formas (cartão de crédito, boleto e PIX), processando transações com conformidade PCI DSS e taxa de falha inferior a 1%. (Integração do sistema)
- O sistema deve garantir privacidade de dados de geolocalização, armazenando-os criptografados (AES-256) e acessíveis apenas a usuários autorizados, em conformidade com LGPD. (Segurança)
- A interface do sistema deve ser acessível via dispositivos móveis (Android e iOS), com tempo de resposta médio inferior a 2 segundos para ações como acionamento de alerta ou recarga. (Desempenho)
- O sistema deve suportar escalabilidade para até 10.000 usuários simultâneos, com disponibilidade de 99,9% em horários de pico acadêmico. (Desempenho)
- O sistema deve ser aplicado no código de cada serviço para garantir que falhas de dependência não derrubem o serviço.(Disponibilidade)

## Arquitetura para Sistema de Controle de Atividades no Campus Universitário

Esta arquitetura é baseada na utilização de microserviços visando escalabilidade, modularidade e integração. Implementada com containers Docker orquestrados por Kubernetes com autoscaling, utiliza mensageria para tarefas que dependam de serviços de terceiros e autenticação via OpenID Connect (Google para alunos) e Keycloak SSO (colaboradores).
O sistema é composto por microserviços independentes, acessados via API Gateway (Kong API Gateway), com comunicação síncrona (REST/HTTP) e assíncrona (Kafka) para interação com serviços externos e notificações. A comunicação entre os microserviços e serviços externos utilizará o protocolo TLS (Transport Layer Security) visando a segurança da troca de dados. Bancos de dados dedicados por serviço garantem desacoplamento e o Kubernetes gerencia orquestração, com autoscaling baseado em métricas de CPU e utilização de memória.

### a) Autenticação:
A autenticação será controlada via Keycloak que manterá base própria de usuários e integração para fins de autenticação via OpenID Connect (Google);
A persistência será realizada em BD PostgreSQL;

### b) Academic Service:
Registra presença automaticamente via geofencing (Google Maps API) ou beacons.
Registra eventos, solicitações e consultas transacionais.
A persistência será realizada em BD PostgreSQL (índices temporais).

### c) Security Service:
Integra com monitoramento externo via REST API.
Pública eventos em Kafka para notificações.
A persistência será realizada em BD MongoDB (geoindexação).

### d) Notification Service:
Recebe as requisições dos demais módulos para fins de serviços
Envia notificações push (Firebase) ou e-mail (AWS SES) baseado em eventos Kafka.

### e) RU Recharge Service
Integração com broker de pagamentos (PagSeguro/Stripe: PIX, cartão, boleto).
Uso de mensageria por meio do Kafka para enfileirar transações assíncronas, garantindo consistência eventual.
A persistência será realizada em BD PostgreSQL (transações ACID).

### Justificativa
A arquitetura baseada em microserviços foi escolhida por sua capacidade de atender aos requisitos funcionais e não funcionais do sistema de controle de atividades no campus universitário, promovendo modularidade, escalabilidade e resiliência. A decomposição em serviços autônomos orquestrados por Kubernetes com autoscaling suporta até 10.000 usuários simultâneos, enquanto a mensageria via Kafka otimiza tarefas de pagamento e notificações. A autenticação via OpenID Connect (Google) para alunos e Keycloak SSO para colaboradores aliada a criptografia TLS para proteção dos dados trocados garante segurança. Integrações externas (monitoramento e pagamentos) são facilitadas por APIs RESTful, e a abordagem modular permite manutenção independente e evolução futura, alinhando-se a padrões largamente utilizados no mercado.
