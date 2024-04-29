# Configuração do Prometheus para Monitoramento do PostgreSQL

Este guia detalha os passos necessários para configurar o Prometheus para monitorar uma instância do PostgreSQL em sua máquina. Antes de começar, verifique se atende aos seguintes requisitos:

## Pré-requisitos

- Uma máquina compatível com uma versão do Postgres Exporter.
- O PostgreSQL deve estar instalado e em execução na máquina. Para obter instruções sobre como instalar o PostgreSQL, consulte o [Guia de Instalação do PostgreSQL](https://www.postgresql.org/download/).
- O Prometheus deve estar instalado em seu ambiente. Consulte a [documentação oficial do Prometheus](https://prometheus.io/download/) para obter instruções sobre como instalá-lo.

## Etapa 1 - Instalação do PostgreSQL

Certifique-se de ter o PostgreSQL instalado em sua máquina. Se estiver usando Ubuntu, você pode instalar o PostgreSQL usando o gerenciador de pacotes apt:

```bash
sudo apt update
sudo apt install postgresql
```

Se desejar uma versão específica do PostgreSQL que não está nos repositórios padrão do Ubuntu, consulte as instruções para configurar o repositório apt do PostgreSQL conforme [descrito aqui](https://www.postgresql.org/download/linux/ubuntu/).

## Etapa 2 - Instalação do Prometheus

Aqui estão os passos para instalar e configurar o Prometheus:

1. **Atualizar o Sistema:**

   Antes de instalar qualquer software novo, atualize seu sistema com os pacotes mais recentes:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Criar um Usuário do Prometheus:**

   Por motivos de segurança, o Prometheus não deve ser executado como usuário root. Portanto, crie um usuário dedicado para o Prometheus:

   ```bash
   sudo useradd --no-create-home --shell /bin/false prometheus
   ```

3. **Baixar e Desempacotar o Prometheus:**

   Baixe a versão mais recente do Prometheus e desempacote-a:

   ```bash
   cd /tmp
   wget https://github.com/prometheus/prometheus/releases/download/v2.33.5/prometheus-2.33.5.linux-amd64.tar.gz
   tar xvf prometheus-2.33.5.linux-amd64.tar.gz
   ```

4. **Configurar o Prometheus:**

   Mova os arquivos de configuração e binários para os diretórios apropriados:

   ```bash
   sudo mv /tmp/prometheus-2.33.5.linux-amd64/prometheus /usr/local/bin/
   sudo mv /tmp/prometheus-2.33.5.linux-amd64/promtool /usr/local/bin/
   sudo mv /tmp/prometheus-2.33.5.linux-amd64/consoles /etc/prometheus
   sudo mv /tmp/prometheus-2.33.5.linux-amd64/console_libraries /etc/prometheus
   sudo mv /tmp/prometheus-2.33.5.linux-amd64/prometheus.yml /etc/prometheus
   ```

5. **Configurar um Serviço do Prometheus:**

   Crie um arquivo de serviço systemd para gerenciar o Prometheus:

   ```bash
   sudo nano /etc/systemd/system/prometheus.service
   ```

   Adicione o seguinte conteúdo ao arquivo `prometheus.service`:

   ```ini
   [Unit]
   Description=Prometheus Monitoring
   Wants=network-online.target
   After=network-online.target

   [Service]
   User=prometheus
   Group=prometheus
   Type=simple
   ExecStart=/usr/local/bin/prometheus \
       --config.file=/etc/prometheus/prometheus.yml \
       --storage.tsdb.path=/var/lib/prometheus/ \
       --web.console.templates=/etc/prometheus/consoles \
       --web.console.libraries=/etc/prometheus/console_libraries

   [Install]
   WantedBy=multi-user.target
   ```

6. **Iniciar e Habilitar o Serviço do Prometheus:**

   Recarregue o daemon systemd e inicie o serviço do Prometheus:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start prometheus
   sudo systemctl enable prometheus
   ```

7. **Verificar a Instalação do Prometheus:**

   Verifique se o Prometheus está em execução:

   ```bash
   sudo systemctl status prometheus
   ```

   Você pode acessar a interface web do Prometheus em seu navegador usando o endereço `http://seu_endereco_ip:9090`.

---

Este guia abrange a configuração do Postgres Exporter para coletar métricas do PostgreSQL e integrá-lo ao Prometheus para monitoramento contínuo.

## Etapa 3 - Configurando o Exportador Postgres e a Raspagem com o Prometheus

### 01: Configurando o Exportador Postgres

1. **Baixe o Binário do Postgres Exporter:**

   Baixe o binário do Postgres Exporter correspondente à sua arquitetura:

   ```bash
   wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.9.0/postgres_exporter-0.9.0.linux-amd64.tar.gz
   ```

   Substitua `v0.9.0` pela versão desejada. Consulte a página de lançamentos para obter a versão mais recente.

2. **Descompacte e Navegue para o Diretório:**

   Descompacte o arquivo baixado e acesse o diretório do Postgres Exporter:

   ```bash
   tar xvfz postgres_exporter-*.linux-amd64.tar.gz
   cd postgres_exporter-*.linux-amd64
   ```

3. **Configure as Credenciais do Postgres:**

   Defina a variável de ambiente `DATA_SOURCE_NAME` para acessar o servidor PostgreSQL:

   ```bash
   export DATA_SOURCE_NAME='postgresql://postgres:sua_senha@nome_do_host:5432/postgres?sslmode=disable'
   ```

   Substitua `postgres` pelo usuário do PostgreSQL, `sua_senha` pela senha do usuário e `nome_do_host` pelo nome do host.

4. **Inicie o Exportador:**

   Execute o Postgres Exporter:

   ```bash
   ./postgres_exporter
   ```

   Verifique se o exportador está em execução.

5. **Teste as Métricas:**

   Use o `curl` para verificar as métricas:

   ```bash
   curl http://localhost:9187/metrics
   ```

   Isso deve retornar as métricas do PostgreSQL no formato Prometheus.

### 02: Raspagem do Exportador Postgres usando o Prometheus

1. **Configure o Prometheus:**

   Adicione uma configuração de raspagem no arquivo `prometheus.yml`:

   ```yaml
   scrape_configs

:
     - job_name: postgres
       static_configs:
         - targets: ['endereco_IP_maquina_exportador_postgres:9187']
   ```

   Substitua `endereco_IP_maquina_exportador_postgres` pelo endereço IP da máquina que executa o Postgres Exporter.

2. **Inicie o Prometheus:**

   Execute o Prometheus com o arquivo de configuração:

   ```bash
   ./prometheus --config.file=./prometheus.yml
   ```

   Isso iniciará o Prometheus e configurará a raspagem do Postgres Exporter.

---

Este guia fornece uma introdução à configuração do Postgres Exporter e sua integração com o Prometheus para monitoramento contínuo das métricas do PostgreSQL. Para detalhes adicionais ou personalizações avançadas, consulte a documentação oficial do [Postgres Exporter](https://github.com/prometheus-community/postgres_exporter) e do [Prometheus](https://prometheus.io/docs/introduction/overview/).
