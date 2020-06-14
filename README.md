# Projeto Antenas
## Devops - Laboratório Projeto BD

### Integrantes: Leonardo Lins, Marcos Kisto, Bruna.

Link Trello: https://trello.com/b/yYt0V2L3/antenas   
Link burndown chart: https://docs.google.com/spreadsheets/d/13EVLT6vyxxCn-7awbnz2QzUW1QQzmkQq-0VSWZxgUMI/edit?usp=sharing   
Endereço jenkins cloud: http://34.66.76.1:8080/   
Monitoramento: https://console.cloud.google.com/monitoring/dashboards?authuser=1&project=canvas-verve-240811  

Endereço aplicação no ar: 

## Tabela das tecnologias implementadas e em adequação até a última sprint

![Tabela de Tecnologias](https://github.com/Marcoskisto/antenas-integracao/blob/master/InfraEstrutura/Tabela%20de%20Tecnologias.png)

## Detalhes sobre as Tecnologias aplicadas

### 1. Source Control Mgmt
Para o versionamento e controle do código fonte, está sendo utilizadas as tecnologias GIT e GITHUB:

### 2. Cloud
A aplicação está sendo hospedada em um servidor atravez do Google Cloud.
#### Instalação e configuração da máquina virtual no Google Cloud
##### 1. Criar uma instância de VM do Debian 9 no GoogleCloud;
Ao colocar a instância em execução, o acesso à VM é feito por SSH pela própria página do googleCloud onde é aberto um terminal. É através desse terminal que serão feitos os passos a seguir para configuração do ambiente e deploy do projeto.
##### 2. Instalar os pacotes no Debian 9:
* Instalar GIT - **sudo apt-get install git**
* Instalar o Docker;
* Instalar o docker-compose;

##### 3. Habilitar o Acesso por HTTP:
* Abrir o GoogleCloud Console;
* Em recursos, clicar em **Compute Engine**;
* Clicar no nome da Instância da VM criada;
* Na parte superior, clicar em **Editar**;
* Rolar a página para baixo e em **Firewalls**, marcar a opção **Permitir Tráfego HTTP**;
* Na política de controle do Firewall, liberar as portas 8080 a 8089, 5000, 9000, 9090,  9100, 9990, 19990, 19999.

### 3. Containers
A inicialização e controle dos containers é realizado através do docker-compose. Para isso foi criado um arquivo docker-compose.yml. O conteúdo do docker-compose.yml:
**[docker-compose](https://github.com/Marcoskisto/antenas-integracao/blob/master/InfraEstrutura/docker-compose.yml)**
* Clonar o Projeto:
    * git clone http://https://github.com/Marcoskisto/antenas-integracao.git
* Inicializar o swarm:
    * docker swarm init
    * cd antenas-integracao/InfraEstrutura/
    * docker stack deploy -c docker-compose.yml antenas-stack (antes desse comando deve-se acessar a pasta de Dockerfiles e criar as imagens docker)   

Com esse docker-compose foram criados os seguintes containers:
#### 1. Container webServer_antenas
 Necessário para rodar a aplicação e torná-la disponível na web. Este servidor trata-se de um tomcat versão 8.0 e será substituído por um wildfly em uma futura srpint. A aplicação é disponibilizada na porta 8088;
#### 2. Container Mongodb do Projeto
Utilizado como banco de dados da aplicação internamente através da porta padrão 27017;
#### 3. Container MongodbExpress 
Para administração interativa do Mongodb pela porta 8081;
#### 4. Conainer Jenkins 
Responsável pelo Continuous Integration. Realiza o deploy automático da aplicação no servidor fazendo o build para que o container "webserver-antenas" execute a aplicação. A administração do jenkins é feita através da porta 8082;
#### 5. Container Netdata
É o container usado para análise do estado atual do host e dos serviços em execução.
#### 6. Container Node_exporter
É o container fundamental para o monitoramento. Esse serviço extrai as informações da máquina linux (/proc) transformando-as em métricas que são repassadas ao Prometheus
#### 7. Container Promethus
É o serviço que busca as métricas no Node_exporter realiza sua persistência em uma base de dados. Cria uma time history das métricas extraídas pelo Node_exporter e as disponibiliza para o Grafana.  
O prometheus também envia alertas configurados de sistema para o cAdvisor. Por exemplo, se um container for derrubado o Prometheus informa o cAdvisor que por sua vez pode enviar emails e alertas pelo RocketChat.
#### 8. Container Grafana (Porta 3000)
Esse container se comunica com o Prometheus( Porta 9090), um serviço de monitoramento instalado no host que funciona em conjunto com o netdata( Porta 19990).
#### 9. AlertManager
É o gerenciador dos Alertas do Prometheus
#### 10. Ansible

## 4. Continuous Integration
Para o continuous Integration foi escolhido o Jenkins que após um commit no github, é feito um deploy de um arquivo .jar em um volume compartilhado com o container webserver que é responsável por rodar a aplicação.

##### 4.1. Comunicação do Github para o Jenkins
A comunicação entre o Github e o Jenkins ocorre em função de uma API. 
Depois do item anterior, é necessário configurar o Github para enviar uma requisição para o Jenkins em nosso servidor. Isso é feito seguindo os seguintes passos:
1. Acessar o projeto no Github;
1. Clicar em **Settings**;
1. Clicar em **Webhooks**;
1. Clicar em **Add**;
1. Em **Payload Url**, inserir: http://_Ip_do_Servidor:_porta_Jenkins_>/github-webhook/
1. Em **Content Type** escolher **application/json**;
1. Logo abaixo marcar a opção **Send me everything**; em uma pasta que também é um volume do container antenas-server.

##### 4.2. Configurar o Jenkins com a API do github
1. Abrir o Jenkins;
1. Clicar no nome do Projeto;
1. Clicar em Configurar;
1. Na Aba **Trigger de builds**, marcar a opção "**GitHub hook trigger for GITScm polling**";
1. Na aba **Gerenciamento de código fonte**, em credentials adicionar a credencial do github;

#### 4.3. Configurar o pipeline de teste e integração no jenkins

## 5. Collaboration
Para a colaboração entre os integrantes do time está sendo utilizado o Trello, e o próprio github para compartilhamento de informações e distribuição das tarefas, além do Rocket.Chat onde informações do projeto podem ser discutidas e onde os alertas de monitoramento são postados automaticamente pela stack de monitoramento.

## 6. Monitoring  
O monitoramento do projeto será realizado integrando três sistemas:

1. Netdata - Realizará o monitoramento em tempo real de várias métricas do host e seus serviços;  
1. Node_exporter - disponibiliza métricas para o Prometheus  
1. Prometheus - A cada intervalo de tempo (10s) irá consultar o Node_exporter sobre as métricas e fará uma base de dados do histórico dessas   
métricas;

### Configurar o Netdata:

### Configurar o Grafana:

## 7. Testing
O teste está sendo implementado por JUnit. O Jenkis realizará um pipeline que utilizará essa rotina para ser analisada no SonarQube
## 8. Analytics
O Grafana obterá do Prometheus as métricas monitoradas e gerará os Dashboards para análise da saúde da infraestrutura utilizada;  
O SonarQube possibilitará análise em virtude do pipeline de testes do projeto.

## 9. Database Automation
## 10. Release Orchestration
## 11. Security
## 12. Configuration
## 13. AIOps


<br><br><br><br><br><br><br><br><br><br><br>
###### Revisão:
Startup dos containers
`cronjob -e`
Será aberto um arquivo de texto e no final deve-se acrescentar:
`@reboot docker-comopse up -d`

Instalar o Prometheus:

* Instalação conforme tutorial: https://github.com/badtuxx/giropops-monitoring
* Liberar a porta **9090** no googleCloud

Instalação do **netdata**
Caso necessite instalar o netdata diretamente no host pode-se realizar os seguintes passos, mas no projeto isso já está sendo feito via containers:

* Instalar dependências: # apt-get install autoconf autoconf-archive autogen automake cmake libjson-c-dev libjudy-dev liblz4-dev libmnl-dev libssl-dev libuv1-dev netcat pkg-config python3-pymongo uuid-dev zlib1g-dev 
* Instalar o Netdata: curl -fsSL https://my-netdata.io/kickstart.sh | bash
* Liberar no firewall do googleCloud a porta **199990**

Criar usuário prometheus:
useradd --nocreate-home --shell /bin/false prometheus
