# 🚀 Configuração do Ambiente Apache Airflow com PostgreSQL utilizando Docker

## 📋 Visão Geral

Este guia apresenta o passo a passo para configurar um ambiente do **Apache Airflow** utilizando:

- Docker
- PostgreSQL
- Ubuntu 24.04
- Python 3.12
- Pandas

Ao final do processo, o ambiente estará pronto para executar DAGs conectadas a um banco PostgreSQL.

---

# 1️⃣ Criando o Container do PostgreSQL

Execute o comando abaixo para subir um container PostgreSQL:

```bash
docker run \
  --name aula_postgres \
  -e POSTGRES_PASSWORD=1234 \
  -p 5432:5432 \
  -d postgres
```

Verifique se o container está em execução:

```bash
docker ps
```

---

# 2️⃣ Criando Banco, Usuário e Permissões

Acesse o PostgreSQL e execute os comandos abaixo.

## Criar banco de dados

```sql
CREATE DATABASE airflow;
```

## Criar usuário

```sql
CREATE USER airflow WITH PASSWORD '1234';
```

## Conceder permissões

```sql
GRANT ALL PRIVILEGES ON DATABASE airflow TO airflow;

ALTER DATABASE airflow OWNER TO airflow;

GRANT CONNECT, TEMP ON DATABASE airflow TO airflow;

GRANT USAGE, CREATE ON SCHEMA public TO airflow;

ALTER SCHEMA public OWNER TO airflow;
```

---

# 3️⃣ Criando o Container Ubuntu para o Airflow

Crie um container Ubuntu que será utilizado para instalar e executar o Airflow:

```bash
docker run \
  -d \
  -p 8080:8080 \
  --name aula_airflow \
  ubuntu:24.04 \
  tail -f /dev/null
```

---

# 4️⃣ Acessando o Container

Entre no container em modo interativo:

```bash
docker exec -it aula_airflow /bin/bash
```

---

# 5️⃣ Atualizando o Sistema e Instalando Dependências

Execute os comandos abaixo dentro do container:

```bash
export DEBIAN_FRONTEND=noninteractive

apt-get update

apt-get upgrade -y

apt-get install -y \
    python3-pip \
    python3-venv \
    python3-dev \
    libpq-dev \
    libssl-dev \
    nano \
    sudo
```

---

# 6️⃣ Configurando Aliases do Python

Edite o arquivo `.bashrc`:

```bash
nano ~/.bashrc
```

Localize a seção:

```text
# some more ls aliases
```

ou

```text
# other aliases
```

Adicione as linhas abaixo:

```bash
alias python=python3
alias pip=pip3
```

Salve o arquivo e recarregue as configurações:

```bash
source ~/.bashrc
```

---

# 7️⃣ Validando a Instalação do Python

Verifique as versões instaladas:

```bash
python --version
pip --version
```

Instale o suporte ao ambiente virtual do Python 3.12:

```bash
apt-get install -y python3.12-venv
```

---

# 8️⃣ Criando o Script de Instalação do Airflow

No diretório `/home`, crie o arquivo:

```bash
nano /home/install-airflow.sh
```

Conteúdo do script:

```bash
#!/bin/bash
set -e

# Versões fixas
AIRFLOW_VERSION=3.0.2
PYTHON_VERSION=3.12

CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

# Criação do usuário airflow
id -u airflow >/dev/null 2>&1 || useradd -m -s /bin/bash airflow

# Ambiente virtual
python3 -m venv /home/airflow/venv

/home/airflow/venv/bin/pip install --upgrade pip

# Instalação do Airflow com suporte ao PostgreSQL
/home/airflow/venv/bin/pip install \
"apache-airflow[postgres]==${AIRFLOW_VERSION}" \
--constraint "${CONSTRAINT_URL}"

# Configuração do AIRFLOW_HOME
export AIRFLOW_HOME=/home/airflow/airflow

mkdir -p "$AIRFLOW_HOME"

chown -R airflow:airflow /home/airflow

echo "Instalação concluída."
```

---

# 9️⃣ Criando o Script de Inicialização do Airflow

Ainda dentro do diretório `/home`, crie:

```bash
nano /home/start-airflow.sh
```

Conteúdo:

```bash
#!/bin/bash
set -e

sudo -u airflow -H bash -lc '
  export AIRFLOW_HOME=/home/airflow/airflow
  source /home/airflow/venv/bin/activate
  airflow standalone
'
```

Este script:

- Ativa o ambiente virtual Python;
- Define o AIRFLOW_HOME;
- Executa o Airflow em modo standalone;
- Inicia Webserver e Scheduler automaticamente.

---

# 🔟 Tornando os Scripts Executáveis

Conceda permissão de execução:

```bash
chmod +x /home/install-airflow.sh

chmod +x /home/start-airflow.sh
```

---

# 1️⃣1️⃣ Executando a Instalação do Airflow

Execute o script de instalação:

```bash
/home/install-airflow.sh
```

Após a conclusão, inicie o Airflow:

```bash
/home/start-airflow.sh
```

A interface ficará disponível em:

```text
http://localhost:8080
```

---

# 1️⃣2️⃣ Criando a Tabela de Destino

Execute o script abaixo no banco PostgreSQL para criar a tabela utilizada pela DAG de contratos.

```sql
CREATE TABLE IF NOT EXISTS contratos (
    id INTEGER PRIMARY KEY,
    cod_concedente TEXT,
    cod_financiador TEXT,
    cod_gestora TEXT,
    cod_orgao TEXT,
    cod_secretaria TEXT,
    descricao_modalidade TEXT,
    descricao_objeto TEXT,
    descricao_tipo TEXT,
    descricao_url TEXT,
    data_assinatura TEXT,
    data_processamento TEXT,
    data_termino TEXT,
    flg_tipo INTEGER,
    isn_parte_destino BIGINT,
    isn_sic BIGINT,
    isn_modalidade BIGINT,
    isn_entidade BIGINT,
    num_spu TEXT,
    valor_contrato NUMERIC(18,2),
    tipo_objeto TEXT,
    num_spu_licitacao TEXT,
    descricao_justificativa TEXT,
    valor_can_rstpg NUMERIC(18,2),
    data_publicacao_portal TEXT,
    descricao_url_pltrb TEXT,
    descricao_url_ddisp TEXT,
    descricao_url_inexg TEXT,
    cod_plano_trabalho TEXT,
    num_certidao TEXT,
    descriaco_edital TEXT,
    cpf_cnpj_financiador TEXT,
    num_contrato TEXT,
    valor_original_concedente NUMERIC(18,2),
    valor_original_contrapartida NUMERIC(18,2),
    valor_atualizado_concedente NUMERIC(18,2),
    valor_atualizado_contrapartida NUMERIC(18,2),
    created_at TEXT,
    updated_at TEXT,
    plain_num_contrato TEXT,
    calculated_valor_aditivo NUMERIC(18,2),
    calculated_valor_ajuste NUMERIC(18,2),
    calculated_valor_empenhado NUMERIC(18,2),
    calculated_valor_pago NUMERIC(18,2),
    contract_type TEXT,
    infringement_status INTEGER,
    cod_financiador_including_zeroes TEXT,
    accountability_status TEXT,
    plain_cpf_cnpj_financiador TEXT,
    descricao_situacao TEXT,
    data_publicacao_doe TEXT,
    descricao_nome_credor TEXT,
    isn_parte_origem TEXT,
    data_auditoria TEXT,
    data_termino_original TEXT,
    data_inicio TEXT,
    data_rescisao TEXT,
    confidential BOOLEAN,
    gestor_contrato TEXT,
    data_finalizacao_prestacao_contas TEXT,
    sequential INTEGER,
    emergency BOOLEAN,
    law TEXT,
    has_non_profit_transfer BOOLEAN,
    nome_fiscal TEXT,
    emenda_parlamentar TEXT,
    ano_emenda_parlamentar INTEGER,
    codigo_emenda_parlamentar TEXT
);
```

---

# 1️⃣3️⃣ Instalando a Biblioteca Pandas

Ative o ambiente virtual:

```bash
source /home/airflow/venv/bin/activate
```

Instale o Pandas:

```bash
pip install pandas
```

Valide a instalação:

```bash
pip show pandas
```

---

# 1️⃣4️⃣ Enviando a DAG para o Container

Copie o arquivo da DAG para o diretório monitorado pelo Airflow.

## Linux

```bash
docker cp ./dag_convenios.py aula_airflow:/home/airflow/airflow/dags
```

## Windows

```powershell
docker cp .\dag_convenios.py aula_airflow:/home/airflow/airflow/dags
```

Após a cópia, confirme:

```bash
docker exec -it aula_airflow ls /home/airflow/airflow/dags
```

---

# 1️⃣5️⃣ Configurando a Comunicação entre Containers

Por padrão, containers diferentes não se comunicam diretamente. Para permitir a comunicação entre o Airflow e o PostgreSQL, crie uma rede Docker.

## Criar rede

```bash
docker network create minha_rede
```

## Conectar PostgreSQL

```bash
docker network connect minha_rede aula_postgres
```

## Conectar Airflow

```bash
docker network connect minha_rede aula_airflow
```

---

# ✅ Teste de Conectividade

Entre no container do Airflow:

```bash
docker exec -it aula_airflow bash
```

Instale o cliente PostgreSQL (caso necessário):

```bash
apt-get update
apt-get install -y postgresql-client
```

Teste a conexão:

```bash
psql \
-h aula_postgres \
-U airflow \
-d airflow
```

Se conectar com sucesso, a integração entre Airflow e PostgreSQL está funcionando corretamente.

---

# 🎯 Resultado Esperado

Ao concluir todas as etapas, você terá:

✅ PostgreSQL executando em container Docker  
✅ Banco `airflow` criado e configurado  
✅ Usuário `airflow` com permissões adequadas  
✅ Ambiente Ubuntu dedicado ao Airflow  
✅ Python 3.12 configurado  
✅ Apache Airflow instalado e funcional  
✅ Pandas disponível para processamento de dados  
✅ DAGs sendo carregadas automaticamente  
✅ Comunicação entre Airflow e PostgreSQL via rede Docker  
✅ Estrutura pronta para desenvolvimento e execução de pipelines de dados
