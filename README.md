# Project TMDB

Projeto de Engenharia de Dados desenvolvido para construção de um pipeline utilizando dados do **TMDB (The Movie Database)**.

A infraestrutura local é executada com **Docker Compose** e utiliza **Apache Airflow**, **PostgreSQL** e **MinIO**.

---

## Arquitetura

Atualmente, o projeto utiliza os seguintes componentes:

* **Apache Airflow** — orquestração das DAGs e pipelines.
* **PostgreSQL** — banco de metadados do Airflow.
* **MinIO** — armazenamento de objetos compatível com S3.
* **MinIO Client (`mc`)** — inicialização e criação automática de buckets.
* **Docker / Docker Compose** — containerização e gerenciamento do ambiente.
* **Python / PySpark** — processamento e transformação dos dados.

```text
                +------------------+
                |      Airflow     |
                |      :8080       |
                +---------+--------+
                          |
             +------------+------------+
             |                         |
             v                         v
    +----------------+        +----------------+
    |   PostgreSQL   |        |     MinIO      |
    |     :5432      |        | :9000 / :9001 |
    +----------------+        +--------+-------+
                                       |
                                       v
                              +----------------+
                              | Bucket landing |
                              +----------------+
```

---

## Tecnologias

| Tecnologia           | Finalidade                                 |
| -------------------- | ------------------------------------------ |
| Python 3.11          | Desenvolvimento do pipeline                |
| Apache Airflow 2.9.2 | Orquestração                               |
| PostgreSQL 15        | Banco de metadados do Airflow              |
| MinIO                | Object Storage compatível com S3           |
| PySpark              | Processamento distribuído                  |
| Pandas               | Manipulação e análise de dados             |
| Boto3                | Integração com serviços compatíveis com S3 |
| Docker               | Containerização                            |
| Docker Compose       | Gerenciamento dos serviços                 |

---

## Estrutura do projeto

```text
project-TMDB/
│
├── airflow/
│   └── dags/
│
├── config_airflow/
│   └── airflow.Dockerfile
│
├── docker-compose.yml
├── requirements.txt
└── README.md
```

### `airflow/dags`

Diretório destinado às DAGs do Apache Airflow.

Esse diretório é montado no container em:

```text
/usr/local/airflow/dags
```

### `config_airflow/airflow.Dockerfile`

Dockerfile responsável pela criação da imagem customizada do Airflow.

```dockerfile
FROM apache/airflow:2.9.2-python3.11

COPY requirements.txt /requirements.txt

RUN pip install --no-cache-dir -r /requirements.txt
```

### `requirements.txt`

Dependências Python adicionais utilizadas pelo projeto:

```text
pyspark
pandas
minio
boto3>=1.34.136
```

O Apache Airflow não precisa ser declarado no `requirements.txt`, pois já está presente na imagem base.

---

## Serviços

### MinIO

Servidor de armazenamento de objetos utilizado no projeto.

Imagem:

```text
quay.io/minio/minio
```

Portas:

| Porta  | Serviço     |
| ------ | ----------- |
| `9000` | API S3      |
| `9001` | Console Web |

Credenciais utilizadas no ambiente local:

```text
Usuário: minioadmin
Senha: minio@1234!
```

> As credenciais atuais são utilizadas apenas no ambiente local de desenvolvimento. Em ambientes compartilhados ou de produção, devem ser armazenadas em variáveis de ambiente ou serviços de gerenciamento de secrets.

---

### MinIO Client

O serviço `minio_mc` utiliza o MinIO Client para configurar o ambiente após a inicialização do servidor.

Ele realiza as seguintes ações:

1. Aguarda o MinIO estar disponível.
2. Configura um alias para o servidor.
3. Cria o bucket `landing`, caso ainda não exista.
4. Encerra a execução.

Por isso, após uma inicialização bem-sucedida, é esperado encontrar:

```text
minio_mc   Exited (0)
```

O código `0` indica que o processo foi concluído com sucesso.

---

### PostgreSQL

Banco utilizado pelo Apache Airflow.

Imagem:

```text
postgres:15
```

Configuração local:

```text
Database: airflow
User: post_airflow
Port: 5432
```

Os dados são persistidos no volume:

```text
postgres_data
```

---

### Apache Airflow

A imagem customizada do Airflow é baseada em:

```text
apache/airflow:2.9.2-python3.11
```

O serviço é disponibilizado na porta:

```text
8080
```

A conexão interna com o PostgreSQL utiliza:

```text
postgresql+psycopg2://post_airflow:airflow_123@postgres-airflow:5432/airflow
```

---

# Como executar

## 1. Clonar o repositório

```bash
git clone <URL_DO_REPOSITORIO>
```

Entre na pasta:

```bash
cd project-TMDB
```

---

## 2. Validar o Docker Compose

Antes de iniciar os containers:

```bash
docker compose config
```

Esse comando valida o arquivo `docker-compose.yml` e exibe a configuração interpretada pelo Docker Compose.

---

## 3. Construir a imagem do Airflow

```bash
docker compose build airflow
```

Após uma construção bem-sucedida, será exibido algo semelhante a:

```text
Image project-tmdb-airflow Built
```

---

## 4. Iniciar os serviços

```bash
docker compose up -d
```

A opção `-d` executa os containers em segundo plano.

---

## 5. Verificar os containers

```bash
docker compose ps
```

Os principais serviços devem aparecer como:

```text
airflow            Up
minio              Up
postgres-airflow   Up
```

Para visualizar também containers finalizados:

```bash
docker compose ps -a
```

O serviço `minio_mc` deve normalmente aparecer como:

```text
Exited (0)
```

---

# Acessos locais

## Apache Airflow

```text
http://localhost:8080
```

## MinIO Console

```text
http://localhost:9001
```

## MinIO API

```text
http://localhost:9000
```

## PostgreSQL

```text
Host: localhost
Port: 5432
Database: airflow
User: post_airflow
```

---

# Logs

Visualizar logs do Airflow:

```bash
docker compose logs airflow
```

Visualizar apenas as últimas 50 linhas:

```bash
docker compose logs airflow --tail=50
```

Acompanhar os logs em tempo real:

```bash
docker compose logs -f airflow
```

Visualizar os logs de todos os serviços:

```bash
docker compose logs
```

---

# Encerrando o ambiente

Para interromper e remover os containers:

```bash
docker compose down
```

Os volumes persistentes são mantidos.

Para remover também os volumes:

```bash
docker compose down -v
```

> **Atenção:** o uso de `-v` remove os dados persistidos do PostgreSQL e do MinIO.

---

# Status do projeto

* [x] Docker Desktop configurado
* [x] Docker Compose configurado
* [x] PostgreSQL 15 configurado
* [x] MinIO configurado
* [x] MinIO Client configurado
* [x] Bucket `landing` criado automaticamente
* [x] Apache Airflow 2.9.2 configurado
* [x] Imagem customizada do Airflow
* [x] Dependências Python instaladas
* [x] Containers executando corretamente
* [ ] Configuração e validação das DAGs
* [ ] Integração com a API do TMDB
* [ ] Ingestão dos dados
* [ ] Armazenamento dos dados brutos no MinIO
* [ ] Processamento e transformação
* [ ] Construção das próximas camadas do pipeline

---

# Próximas etapas

As próximas etapas do projeto incluem:

* integração com a API do TMDB;
* criação das DAGs no Apache Airflow;
* ingestão automatizada dos dados;
* armazenamento da camada bruta no MinIO;
* transformação dos dados com Python e PySpark;
* construção das próximas camadas do Data Lake;
* implementação de controles de qualidade e monitoramento do pipeline.

---

# Autor

**Cleber Furtado**

Projeto desenvolvido para estudo e aplicação prática de conceitos de **Engenharia de Dados**, **Data Lake**, **orquestração de pipelines**, **processamento de dados** e **arquitetura baseada em containers**.
