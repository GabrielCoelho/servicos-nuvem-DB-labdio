# Guia Prático: Instância Gerenciada de SQL do Azure

## 📑 Sumário
- [Introdução](#introdução)
- [Pré-requisitos](#pré-requisitos)
- [Criação da Instância Gerenciada](#criação-da-instância-gerenciada)
  - [Configurações Básicas](#configurações-básicas)
  - [Computação e Armazenamento](#computação-e-armazenamento)
  - [Configurações de Rede](#configurações-de-rede)
  - [Segurança](#segurança)
  - [Configurações Adicionais](#configurações-adicionais)
- [Monitoramento da Implantação](#monitoramento-da-implantação)
- [Criação de Banco de Dados](#criação-de-banco-de-dados)
- [Recuperando Detalhes de Conexão](#recuperando-detalhes-de-conexão)
- [Dicas e Boas Práticas](#dicas-e-boas-práticas)
- [Recursos Adicionais](#recursos-adicionais)

## 📚 Introdução

A Instância Gerenciada de SQL do Azure é um serviço de banco de dados SQL na nuvem que combina a mais ampla compatibilidade com o mecanismo de banco de dados do SQL Server com todas as vantagens de uma plataforma como serviço (PaaS) totalmente gerenciada. Este serviço é ideal para migrações de bancos de dados SQL Server on-premises para a nuvem, oferecendo compatibilidade quase completa com o SQL Server local, sem a necessidade de redesenhar seus aplicativos.

Este guia cobre o processo detalhado de criação e configuração de uma Instância Gerenciada de SQL do Azure, desde os pré-requisitos até a conexão ao banco de dados.

## 🔧 Pré-requisitos

Antes de começar, você precisará:

- **Uma assinatura do Azure**: Se não tiver, crie uma [conta gratuita](https://azure.microsoft.com/pt-br/free/)
- **PowerShell ou CLI do Azure**:
  - Para PowerShell: Módulo Az.SQL mais recente
  - Para CLI: Versão mais recente da CLI do Azure
- **Verificar as limitações**:
  - [Regiões compatíveis](https://learn.microsoft.com/pt-br/azure/azure-sql/managed-instance/resource-limits#regional-resource-limitations)
  - [Tipos de assinatura compatíveis](https://learn.microsoft.com/pt-br/azure/azure-sql/managed-instance/resource-limits#supported-subscription-types)

> **Observação**: Experimente a Instância Gerenciada de SQL do Azure sem custo e obtenha 720 horas de vCore em uma Instância Gerenciada de Uso Geral com até 100 bancos de dados nos primeiros 12 meses.

## 🚀 Criação da Instância Gerenciada

A criação de uma Instância Gerenciada de SQL pode ser feita através do portal do Azure, PowerShell ou CLI do Azure. Este guia foca no processo via portal.

> **Importante**: A implantação de uma instância gerenciada é uma operação de longa duração. A implantação da primeira instância na sub-rede normalmente demora muito mais do que a implantação em uma sub-rede com instâncias existentes.

### 🔹 Acessando o Portal

1. Entre no [portal do Azure](https://portal.azure.com)
2. No menu esquerdo, selecione **SQL do Azure** (ou pesquise "SQL do Azure" se não estiver na lista)
3. Clique em **+ Criar** para abrir a página "Selecionar opção de implantação do SQL"
4. Escolha **Instância única** na lista suspensa e selecione **Criar**

### 🔹 Configurações Básicas

Na guia **Básico**, preencha as informações obrigatórias:

| Configuração | Descrição |
|-------------|-----------|
| Assinatura | Sua assinatura do Azure com permissão para criar recursos |
| Grupo de recursos | Selecione ou crie um novo grupo de recursos |
| Nome da instância gerenciada | Escolha um nome válido (consulte [regras de nomenclatura](https://learn.microsoft.com/pt-br/azure/azure-resource-manager/management/resource-name-rules)) |
| Região | Selecione a região onde deseja criar a instância |
| Pertence a um pool de instâncias? | Selecione "Sim" para criar dentro de um pool de instâncias existente |
| Método de autenticação | Escolha "Autenticação SQL" para fins de início rápido |
| Logon de administrador | Defina um nome de usuário (não use "serveradmin", pois é uma função reservada) |
| Senha | Defina uma senha forte (mínimo 16 caracteres, atendendo aos requisitos de complexidade) |

### 🔹 Computação e Armazenamento

Em **Detalhes da instância gerenciada**, selecione **Configurar instância gerenciada** na seção **Computação + armazenamento**:

| Configuração | Valor Recomendado | Descrição |
|-------------|-------------------|-----------|
| Camada de Serviço | Uso Geral | Atende à maioria das cargas de trabalho; existe também a camada "Uso Geral de Última Geração" |
| Geração do hardware | Série Standard (Gen 5) | Define os limites de computação e memória |
| vCores | 8 (padrão) ou conforme necessidade | Recursos de computação que serão dedicados à instância |
| Armazenamento em GB | Baseado no tamanho esperado dos dados | Capacidade de armazenamento da instância |
| Licença do SQL Server | Pagamento conforme o uso ou use o Benefício Híbrido do Azure | Escolha conforme sua situação de licenciamento |
| Redundância do armazenamento de backup | Armazenamento com redundância geográfica (padrão) | Oferece a melhor proteção para seus backups |

Após configurar, clique em **Aplicar** e **Próximo**.

### 🔹 Configurações de Rede

Na guia **Rede**, configure:

| Configuração | Recomendação | Descrição |
|-------------|--------------|-----------|
| Rede virtual/sub-rede | Crie nova ou use existente | A sub-rede deve atender aos [requisitos de rede](https://learn.microsoft.com/pt-br/azure/azure-sql/managed-instance/connectivity-architecture-overview) |
| Tipo de conexão | Conforme sua necessidade | Proxy ou redirecionado |
| Ponto de extremidade público | Desabilitar (mais seguro) | Habilite apenas se precisar acessar de fora da VNet |
| Permitir acesso de | Sem Acesso (padrão) | Se habilitar ponto de extremidade público, escolha as regras de acesso |

### 🔹 Segurança

Para fins de início rápido, mantenha as configurações padrão na guia **Segurança**.

### 🔹 Configurações Adicionais

Na guia **Configurações adicionais**, você pode definir:

| Configuração | Descrição |
|-------------|-----------|
| Ordenação | Escolha a ordenação adequada ao seu ambiente (importante para migração) |
| Fuso horário | Selecione o fuso horário que a instância observará |
| Replicação geográfica | Normalmente "Não" para configuração inicial |
| Janela de manutenção | Defina quando a manutenção automática deve ocorrer |

### 🔹 Marcações (Tags)

Adicione tags para organizar seus recursos. Recomenda-se ao menos:
- **Proprietário**: Identifica quem criou o recurso
- **Ambiente**: Identifica se é produção, desenvolvimento, teste, etc.

### 🔹 Revisão e Criação

Após revisar todas as configurações, clique em **Criar** para iniciar a implantação.

## 📊 Monitoramento da Implantação

1. Selecione o ícone **Notificações** para acompanhar o status
2. Clique em **Implantação em andamento** para ver mais detalhes
3. A implantação pode levar várias horas, especialmente a primeira instância em uma sub-rede

> **Dica**: Se fechar o navegador, você pode monitorar o progresso através da página **Visão geral** da instância no portal do Azure.

## 💾 Criação de Banco de Dados

Após a implantação, você pode criar um banco de dados:

1. Acesse a instância gerenciada no portal do Azure
2. Na página **Visão geral**, clique em **+ Novo banco de dados**
3. Dê um nome para o banco de dados na guia **Básico**
4. Na guia **Fonte de dados**, selecione "Nenhuma" para um banco vazio
5. Configure as opções restantes conforme necessário
6. Clique em **Criar** para implantar o banco de dados

## 🔌 Recuperando Detalhes de Conexão

Para se conectar à instância:

1. Acesse a instância gerenciada no portal
2. Na guia **Visão Geral**, localize a propriedade **Host**
3. Copie o nome do host para usar em suas conexões

O valor copiado representa um FQDN no formato: `nome_do_seu_host.a1b2c3d4e5f6.database.windows.net`

## 💡 Dicas e Boas Práticas

- **Rede**: Planeje cuidadosamente sua configuração de rede antes da implantação
- **Segurança**: Mantenha o ponto de extremidade público desabilitado quando possível
- **Desempenho**: 
  - Escolha a camada de serviço e os vCores adequados à sua carga de trabalho
  - Dimensione o armazenamento baseado nas projeções de crescimento
- **Migração**:
  - Use o Serviço de Migração de Banco de Dados do Azure para migrações
  - Verifique a ordenação do banco de origem antes da migração
- **Monitoramento**:
  - Configure alertas para uso de recursos e desempenho
  - Use o Observador de Banco de Dados para monitoramento contínuo

## 📘 Recursos Adicionais

- [Configurar uma máquina virtual do Azure](https://learn.microsoft.com/pt-br/azure/virtual-machines/windows/quick-create-portal)
- [Visão geral da migração: De SQL Server para Instância Gerenciada de SQL](https://learn.microsoft.com/pt-br/azure/azure-sql/migration-guides/managed-instance/sql-server-to-managed-instance-overview)
- [Configurar uma conexão ponto a site](https://learn.microsoft.com/pt-br/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal)
- [Monitorar usando o observador de banco de dados](https://learn.microsoft.com/pt-br/azure/azure-sql/database/database-watcher-overview)
- [Serviço de Migração de Banco de Dados do Azure](https://learn.microsoft.com/pt-br/azure/dms/dms-overview)
- [Comando T-SQL RESTORE](https://learn.microsoft.com/pt-br/sql/t-sql/statements/restore-statements-transact-sql)
