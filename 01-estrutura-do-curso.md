# JDBC com Java e PostgreSQL

Curso introdutório de **JDBC** utilizando **Java**, **Maven** e **PostgreSQL**.

Durante o curso, vamos construir um pequeno sistema de cadastro de clientes, entendendo como uma aplicação Java se comunica diretamente com um banco de dados e como podemos organizar essa comunicação utilizando separação de responsabilidades.

---

## 🎯 Objetivo do curso

Ao final do curso, o aluno deverá ser capaz de:

* compreender o que é JDBC;
* conectar uma aplicação Java ao PostgreSQL;
* utilizar `Connection` e `DriverManager`;
* criar uma `ConnectionFactory`;
* compreender os padrões Factory e Singleton;
* executar operações de CRUD;
* utilizar `PreparedStatement` e `ResultSet`;
* mapear registros do banco para objetos Java;
* aplicar o padrão DAO;
* utilizar interfaces para desacoplar implementações;
* criar uma camada de Service;
* separar persistência de regras de negócio;
* utilizar injeção de dependência pelo construtor;
* organizar uma aplicação Java em responsabilidades bem definidas.

---

# 📚 Estrutura do curso

O conteúdo será dividido em quatro arquivos:

```text
01-estrutura-do-curso.md
02-setup-e-conexao.md
03-crud-dao-e-interface.md
04-service-e-regras-de-negocio.md
```

O primeiro arquivo apresenta a estrutura geral, configurações iniciais e banco de dados utilizado durante o curso.

Os demais arquivos acompanham a evolução da aplicação.

---

# Parte 1 — Setup e Conexão

## Objetivo

Entender como uma aplicação Java estabelece uma conexão com o PostgreSQL utilizando JDBC e como podemos centralizar a criação dessas conexões.

## Conteúdo

* introdução ao JDBC;
* criação do projeto Maven;
* configuração do `pom.xml`;
* adição do driver PostgreSQL;
* criação do banco de dados;
* criação da tabela `cliente`;
* conceito de `Connection`;
* conceito de `DriverManager`;
* conexão direta com PostgreSQL;
* criação da `ConnectionFactory`;
* centralização da criação das conexões;
* conceito de Factory;
* conceito de Singleton;
* Singleton aplicado à `ConnectionFactory`;
* diferença entre Singleton da Factory e da `Connection`;
* `try-with-resources`;
* teste da conexão com o banco.

## Fluxo

```text
Aplicação
    ↓
ConnectionFactory
    ↓
DriverManager
    ↓
Connection
    ↓
PostgreSQL
```

> O Singleton será aplicado à `ConnectionFactory`, e não à `Connection`.
>
> Cada operação poderá solicitar uma nova conexão ao banco.

---

# Parte 2 — CRUD, DAO e Interface

## Objetivo

Aprender a executar operações de persistência utilizando JDBC e organizar o acesso ao banco utilizando o padrão DAO.

## Conteúdo

* criação da classe `Cliente`;
* correspondência entre tipos PostgreSQL e Java;
* `INSERT`;
* `SELECT`;
* `UPDATE`;
* `DELETE`;
* utilização de `PreparedStatement`;
* parâmetros utilizando `?`;
* `setString()`;
* `setInt()`;
* `setObject()`;
* `executeUpdate()`;
* `executeQuery()`;
* utilização de `ResultSet`;
* leitura dos dados retornados pelo banco;
* mapeamento de registros para objetos Java;
* conceito de DAO;
* criação da interface `ClienteDAO`;
* criação da implementação `ClienteDAOImpl`;
* utilização de interfaces;
* separação das responsabilidades de persistência.

## CRUD desenvolvido

| Operação         | SQL      | Método          |
| ---------------- | -------- | --------------- |
| Criar            | `INSERT` | `salvar()`      |
| Consultar por ID | `SELECT` | `buscarPorId()` |
| Listar           | `SELECT` | `listarTodos()` |
| Atualizar        | `UPDATE` | `atualizar()`   |
| Excluir          | `DELETE` | `excluir()`     |

## Fluxo de persistência

```text
Cliente
   ↓
ClienteDAO
   ↓
ClienteDAOImpl
   ↓
ConnectionFactory
   ↓
JDBC
   ↓
PostgreSQL
```

Nas consultas, teremos também o caminho inverso:

```text
PostgreSQL
    ↓
ResultSet
    ↓
Cliente
```

A interface define **o que pode ser feito**.

```java
public interface ClienteDAO {

    void salvar(Cliente cliente);

    Cliente buscarPorId(Integer id);

    List<Cliente> listarTodos();

    void atualizar(Cliente cliente);

    void excluir(Integer id);
}
```

A implementação `ClienteDAOImpl` define **como essas operações serão realizadas utilizando JDBC**.

---

# Parte 3 — Service e Regras de Negócio

## Objetivo

Separar as regras de negócio da lógica de persistência, criando uma nova camada de responsabilidade na aplicação.

## Conteúdo

* conceito de Service;
* criação da `ClienteService`;
* diferença entre persistência e regra de negócio;
* validações de dados;
* comunicação entre Service e DAO;
* dependência através da interface `ClienteDAO`;
* injeção de dependência pelo construtor;
* montagem das dependências na `Main`;
* fluxo completo da aplicação;
* atividade prática para completar as demais operações.

## Separação das responsabilidades

```text
Cliente
↓
Representa os dados

ClienteDAO / ClienteDAOImpl
↓
Persistência

ClienteService
↓
Regras de negócio

ConnectionFactory
↓
Criação das conexões
```

O fluxo completo da aplicação passa a ser:

```text
Main
  ↓
ClienteService
  ↓
ClienteDAO
  ↓
ClienteDAOImpl
  ↓
ConnectionFactory
  ↓
JDBC
  ↓
PostgreSQL
```

A regra principal será:

> **Regra de negócio não deve ficar dentro do DAO.**

O DAO deve se preocupar com acesso aos dados.

A Service deve se preocupar com validações, decisões e regras da aplicação.

---

# 🗂️ Estrutura final do projeto

Ao final do curso teremos aproximadamente a seguinte estrutura:

```text
jdbc-clientes
│
├── pom.xml
│
└── src
    └── main
        └── java
            └── com
                └── devforce
                    └── jdbc
                        │
                        ├── Main.java
                        │
                        ├── model
                        │   └── Cliente.java
                        │
                        ├── dao
                        │   ├── ClienteDAO.java
                        │   └── ClienteDAOImpl.java
                        │
                        ├── service
                        │   └── ClienteService.java
                        │
                        └── config
                            └── ConnectionFactory.java
```

Cada pacote terá uma responsabilidade:

```text
model
↓
Representação dos dados

dao
↓
Acesso e persistência dos dados

service
↓
Regras de negócio

config
↓
Configuração e criação das conexões
```

---

# ⚙️ Configuração do Maven

## `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.devforce</groupId>
    <artifactId>jdbc-clientes</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>26</maven.compiler.source>
        <maven.compiler.target>26</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.7</version>
        </dependency>

    </dependencies>

</project>
```

---

# 🐘 Banco de dados

Durante o curso utilizaremos **PostgreSQL**.

## Criando o banco

```sql
CREATE DATABASE devforce_jdbc;
```

Após criar o banco, conecte-se ao banco:

```text
devforce_jdbc
```

---

# 📋 Tabela `cliente`

A aplicação trabalhará com apenas uma tabela.

```sql
CREATE TABLE cliente (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    data_nascimento DATE NOT NULL,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Estrutura da tabela

| Campo             | Tipo           | Descrição                   |
| ----------------- | -------------- | --------------------------- |
| `id`              | `SERIAL`       | Identificador do cliente    |
| `nome`            | `VARCHAR(100)` | Nome do cliente             |
| `data_nascimento` | `DATE`         | Data de nascimento          |
| `criado_em`       | `TIMESTAMP`    | Data de criação do registro |

---

# 🌱 Dados iniciais

Para começar os testes, podemos inserir alguns clientes manualmente.

```sql
INSERT INTO cliente (nome, data_nascimento)
VALUES
    ('Ana Silva', '1995-05-10'),
    ('Carlos Souza', '1988-11-23'),
    ('Mariana Santos', '2001-02-15');
```

---

# 🔎 Consulta inicial

Para verificar os registros:

```sql
SELECT *
FROM cliente;
```

Resultado esperado:

```text
id | nome             | data_nascimento | criado_em
---+------------------+-----------------+---------------------
1  | Ana Silva        | 1995-05-10      | ...
2  | Carlos Souza     | 1988-11-23      | ...
3  | Mariana Santos   | 2001-02-15      | ...
```

---

# 🧩 Modelo do projeto

## Cliente

A tabela `cliente` será representada na aplicação através da classe `Cliente`.

```text
Cliente
├── id
├── nome
├── dataNascimento
└── criadoEm
```

Esses campos posteriormente serão preenchidos através dos dados retornados pelo `ResultSet`.

A correspondência será aproximadamente:

```text
Banco de dados       Java

id                →  id
nome              →  nome
data_nascimento   →  dataNascimento
criado_em         →  criadoEm
```

---

# 🔄 CRUD que será desenvolvido

Durante o curso implementaremos as quatro operações fundamentais de persistência:

```text
CRUD

C → Create
R → Read
U → Update
D → Delete
```

Em nosso projeto:

| Operação  | SQL      | Java                              |
| --------- | -------- | --------------------------------- |
| Criar     | `INSERT` | `salvar()`                        |
| Consultar | `SELECT` | `buscarPorId()` / `listarTodos()` |
| Atualizar | `UPDATE` | `atualizar()`                     |
| Excluir   | `DELETE` | `excluir()`                       |

---

# 🏁 Resultado final

Ao final do curso teremos uma aplicação com responsabilidades separadas:

```text
Main
 │
 ▼
ClienteService
 │
 ▼
ClienteDAO
 │
 ▼
ClienteDAOImpl
 │
 ▼
ConnectionFactory
 │
 ▼
JDBC
 │
 ▼
PostgreSQL
```

Cada camada terá um propósito específico:

```text
Main
↓
Inicia e utiliza a aplicação

Service
↓
Regras de negócio

DAO
↓
Persistência

ConnectionFactory
↓
Criação das conexões

JDBC
↓
Comunicação com o banco

PostgreSQL
↓
Armazenamento dos dados
```

O projeto será simples propositalmente.

O objetivo não é criar uma arquitetura complexa, mas entender **como uma aplicação Java conversa diretamente com um banco de dados utilizando JDBC** e como podemos evoluir esse código aplicando **separação de responsabilidades, DAO, interfaces, Factory, Singleton e Service**.

Ao final, o aluno terá uma base que poderá reutilizar em outros pequenos projetos e estará mais preparado para entender como frameworks como **Spring** organizam aplicações Java.
