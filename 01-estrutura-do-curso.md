# JDBC com Java e PostgreSQL

Curso introdutório de **JDBC** utilizando **Java**, **Maven** e **PostgreSQL**.

Durante o curso, vamos construir um pequeno sistema de cadastro de clientes, aplicando conceitos de persistência, organização de código e alguns padrões de projeto.

---

## 🎯 Objetivo do curso

Ao final do curso, o aluno deverá ser capaz de:

* compreender o que é JDBC;
* conectar uma aplicação Java ao PostgreSQL;
* executar operações de CRUD;
* utilizar `PreparedStatement` e `ResultSet`;
* mapear registros do banco para objetos Java;
* aplicar o padrão DAO;
* utilizar interfaces para desacoplar implementações;
* criar uma `ConnectionFactory`;
* compreender a aplicação de Factory e Singleton;
* organizar uma aplicação Java com acesso a banco de dados.

---

# 📚 Estrutura do curso

## Parte 1 — Setup e conexão

### Objetivo

Entender o que é JDBC e conseguir estabelecer uma conexão entre a aplicação Java e o banco de dados.

### Conteúdo

* Introdução ao JDBC;
* criação de um projeto Maven;
* configuração do `pom.xml`;
* adição do driver PostgreSQL;
* criação do banco de dados;
* criação da tabela `cliente`;
* conceito de `Connection`;
* conceito de `DriverManager`;
* criação de uma `ConnectionFactory`;
* teste da conexão com o banco.

---

## Parte 2 — CRUD básico

### Objetivo

Entender o fluxo básico de persistência utilizando JDBC.

### Conteúdo

* `INSERT`;
* `SELECT`;
* `UPDATE`;
* `DELETE`;
* utilização de `PreparedStatement`;
* utilização de `ResultSet`;
* parâmetros em comandos SQL;
* leitura dos dados retornados pelo banco;
* conversão de registros para objetos Java;
* criação da classe `Cliente`;
* mapeamento de `ResultSet` para `Cliente`.

### Operações estudadas

```text
Java
  ↓
Connection
  ↓
PreparedStatement
  ↓
PostgreSQL
  ↓
ResultSet
  ↓
Cliente
```

---

## Parte 3 — DAO + Interface

### Objetivo

Retirar os comandos SQL da classe principal e começar a organizar as responsabilidades da aplicação.

### Conteúdo

* conceito de DAO;
* responsabilidade de uma classe DAO;
* criação da interface `ClienteDAO`;
* criação da implementação `ClienteDAOImpl`;
* utilização de interfaces;
* separação entre persistência e regra de negócio;
* centralização das operações relacionadas ao banco.

### Estrutura esperada

```text
ClienteDAO
    ↑
    │ implements
    │
ClienteDAOImpl
```

A interface define **o que pode ser feito**.

A implementação define **como essas operações serão realizadas no banco de dados**.

Exemplo:

```java
public interface ClienteDAO {

    void salvar(Cliente cliente);

    List<Cliente> listar();

    void atualizar(Cliente cliente);

    void excluir(Long id);
}
```

---

## Parte 4 — Factory + Singleton + organização final

### Objetivo

Conhecer alguns padrões utilizados na organização do acesso ao banco de dados sem aumentar desnecessariamente a complexidade do projeto.

### Conteúdo

* revisão da `ConnectionFactory`;
* problema de espalhar `DriverManager.getConnection()` pelo projeto;
* centralização da criação de conexões;
* conceito de Factory;
* conceito de Singleton;
* Singleton aplicado à `ConnectionFactory`;
* integração entre DAO e Factory;
* fluxo completo do cadastro;
* revisão da arquitetura final do projeto.

### Fluxo final

```text
Main
  ↓
ClienteDAO
  ↓
ClienteDAOImpl
  ↓
ConnectionFactory
  ↓
Connection
  ↓
PostgreSQL
```

> A `ConnectionFactory` pode ser Singleton, mas cada chamada pode fornecer uma nova `Connection`.
>
> Dessa forma, não mantemos necessariamente uma única conexão com o banco durante toda a aplicação.

---

# 🗂️ Estrutura sugerida do projeto

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
                        └── config
                            └── ConnectionFactory.java
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

A aplicação trabalhará inicialmente com apenas uma tabela.

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
id | nome            | data_nascimento | criado_em
---+-----------------+-----------------+---------------------
1  | Ana Silva       | 1995-05-10      | ...
2  | Carlos Souza    | 1988-11-23      | ...
3  | Mariana Santos  | 2001-02-15      | ...
```

---

# 🧩 Entidades do projeto

## Cliente

A tabela `cliente` será representada na aplicação através da classe `Cliente`.

```text
Cliente
├── id
├── nome
├── dataNascimento
└── criadoEm
```

Posteriormente esses campos serão mapeados utilizando o `ResultSet`.

---

# 🔄 CRUD que será desenvolvido

Durante o curso implementaremos as quatro operações fundamentais:

| Operação  | SQL      | Java          |
| --------- | -------- | ------------- |
| Criar     | `INSERT` | `salvar()`    |
| Consultar | `SELECT` | `listar()`    |
| Atualizar | `UPDATE` | `atualizar()` |
| Excluir   | `DELETE` | `excluir()`   |

---

# 🏁 Resultado final

Ao final das quatro partes, teremos uma aplicação organizada aproximadamente desta forma:

```text
Main
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

O projeto será simples propositalmente.

O objetivo não é criar uma arquitetura complexa, mas entender **como uma aplicação Java conversa diretamente com um banco de dados utilizando JDBC** e como podemos organizar esse acesso de maneira mais limpa.
