# Parte 03 — CRUD básico: DAO e Interface

## Objetivo

Nesta etapa vamos construir as operações básicas de persistência utilizando JDBC.

Também começaremos a organizar nosso código utilizando o padrão **DAO — Data Access Object**.

Ao final desta etapa teremos:

* classe `Cliente`;
* interface `ClienteDAO`;
* implementação `ClienteDAOImpl`;
* operação de cadastro;
* consulta por ID;
* listagem;
* atualização;
* exclusão.

O fluxo ficará aproximadamente assim:

```text
Cliente
   ↓
ClienteDAO
   ↓
ClienteDAOImpl
   ↓
ConnectionFactory
   ↓
PostgreSQL
```

---

# 1. Criando a classe Cliente

Primeiro vamos criar a classe `Cliente`, responsável por representar em Java os dados armazenados na tabela `cliente`.

Nossa tabela possui:

```text
cliente

id
nome
data_nascimento
criado_em
```

No Java teremos:

```text
Cliente

id
nome
dataNascimento
criadoEm
```

## Classe Cliente

```java
package com.devforce.jdbc.model;

import java.time.LocalDate;
import java.time.LocalDateTime;

public class Cliente {

    private Integer id;
    private String nome;
    private LocalDate dataNascimento;
    private LocalDateTime criadoEm;

    public Cliente() {
    }

    public Cliente(
            String nome,
            LocalDate dataNascimento
    ) {
        this.nome = nome;
        this.dataNascimento = dataNascimento;
    }

    public Cliente(
            Integer id,
            String nome,
            LocalDate dataNascimento,
            LocalDateTime criadoEm
    ) {
        this.id = id;
        this.nome = nome;
        this.dataNascimento = dataNascimento;
        this.criadoEm = criadoEm;
    }

    // getters e setters
}
```

Alguns tipos Java possuem correspondência natural com os tipos utilizados no PostgreSQL.

```text
PostgreSQL          Java

INTEGER       →     Integer
VARCHAR       →     String
DATE          →     LocalDate
TIMESTAMP     →     LocalDateTime
```

---

# 2. Criando o DAO

DAO significa:

**Data Access Object**

É um padrão utilizado para separar o código responsável pelo acesso ao banco de dados das outras partes da aplicação.

Em vez de espalhar comandos SQL pelo projeto, podemos concentrá-los em uma classe específica.

Nosso DAO será responsável pelas operações:

```text
salvar
buscarPorId
listarTodos
atualizar
excluir
```

---

# Interface ClienteDAO

Primeiro vamos definir quais operações nosso DAO deve possuir.

```java
package com.devforce.jdbc.dao;

import com.devforce.jdbc.model.Cliente;

import java.util.List;

public interface ClienteDAO {

    void salvar(Cliente cliente);

    Cliente buscarPorId(Integer id);

    List<Cliente> listarTodos();

    void atualizar(Cliente cliente);

    void excluir(Integer id);
}
```

A interface define **o que pode ser feito**, mas não define ainda **como será feito**.

```text
ClienteDAO

salvar()
buscarPorId()
listarTodos()
atualizar()
excluir()
```

A implementação dessas operações ficará em:

```text
ClienteDAOImpl
```

---

# 3. Criando ClienteDAOImpl

Agora vamos criar a classe responsável pela implementação das operações definidas em `ClienteDAO`.

```java
public class ClienteDAOImpl implements ClienteDAO {

    private final ConnectionFactory connectionFactory;

    public ClienteDAOImpl() {
        this.connectionFactory =
                ConnectionFactory.getInstance();
    }

}
```

A classe implementa nossa interface:

```text
ClienteDAO
    ↑
    │ implements
    │
ClienteDAOImpl
```

Também utilizamos a `ConnectionFactory` criada anteriormente para obter conexões com o banco.

---

# 4. Implementando o CRUD

CRUD representa quatro operações fundamentais:

```text
C → Create
R → Read
U → Update
D → Delete
```

Em SQL:

```text
CREATE → INSERT
READ   → SELECT
UPDATE → UPDATE
DELETE → DELETE
```

---

# 4.1 Salvar

Vamos começar inserindo um cliente no banco de dados.

## SQL

```sql
INSERT INTO cliente
    (nome, data_nascimento)
VALUES
    (?, ?)
```

Os caracteres `?` representam valores que serão informados posteriormente pelo Java.

## Implementação

```java
@Override
public void salvar(Cliente cliente) {

    String sql = """
            INSERT INTO cliente
                (nome, data_nascimento)
            VALUES
                (?, ?)
            """;

    try (
            Connection connection =
                    connectionFactory.getConnection();

            PreparedStatement statement =
                    connection.prepareStatement(sql)
    ) {

        statement.setString(
                1,
                cliente.getNome()
        );

        statement.setObject(
                2,
                cliente.getDataNascimento()
        );

        statement.executeUpdate();

    } catch (SQLException e) {

        throw new RuntimeException(e);
    }
}
```

---

# Entendendo o código

## Connection

```java
Connection connection =
        connectionFactory.getConnection();
```

`Connection` representa uma conexão ativa entre nossa aplicação Java e o banco de dados.

```text
Java
 ↓
Connection
 ↓
PostgreSQL
```

---

## PreparedStatement

```java
PreparedStatement statement =
        connection.prepareStatement(sql);
```

O `PreparedStatement` representa um comando SQL preparado para ser enviado ao banco.

Ele permite trabalhar com parâmetros utilizando `?`.

Por exemplo:

```sql
INSERT INTO cliente
    (nome, data_nascimento)
VALUES
    (?, ?)
```

Depois substituímos esses parâmetros através do Java.

---

## setString

```java
statement.setString(
        1,
        cliente.getNome()
);
```

O primeiro parâmetro indica a posição do `?`.

O segundo contém o valor que será enviado.

```text
VALUES (?, ?)
        ↑
        1
```

Portanto:

```java
setString(1, cliente.getNome());
```

preenche o primeiro parâmetro.

---

## setObject

```java
statement.setObject(
        2,
        cliente.getDataNascimento()
);
```

Aqui estamos preenchendo o segundo parâmetro.

Como `LocalDate` possui suporte pelo driver JDBC moderno, podemos utilizar `setObject()`.

```text
VALUES (?, ?)
           ↑
           2
```

---

## executeUpdate

```java
statement.executeUpdate();
```

`executeUpdate()` é utilizado normalmente para comandos que alteram dados.

Exemplos:

```text
INSERT
UPDATE
DELETE
```

Ele executa o comando SQL no banco de dados.

---

# try-with-resources

Observe que utilizamos:

```java
try (
        Connection connection = ...;

        PreparedStatement statement = ...
) {

}
```

Esse recurso do Java é chamado de:

**try-with-resources**

Ele fecha automaticamente os recursos utilizados quando o bloco termina.

Ou seja:

```text
abre Connection
      ↓
abre PreparedStatement
      ↓
executa operação
      ↓
fecha PreparedStatement
      ↓
fecha Connection
```

Isso evita deixar conexões e outros recursos abertos desnecessariamente.

---

# 4.2 Listar todos

Agora vamos buscar todos os clientes cadastrados.

## SQL

```sql
SELECT
    id,
    nome,
    data_nascimento,
    criado_em
FROM cliente
ORDER BY id
```

## Implementação

```java
@Override
public List<Cliente> listarTodos() {

    String sql = """
            SELECT
                id,
                nome,
                data_nascimento,
                criado_em
            FROM cliente
            ORDER BY id
            """;

    List<Cliente> clientes =
            new ArrayList<>();

    try (
            Connection connection =
                    connectionFactory.getConnection();

            PreparedStatement statement =
                    connection.prepareStatement(sql);

            ResultSet resultSet =
                    statement.executeQuery()
    ) {

        while (resultSet.next()) {

            Cliente cliente =
                    new Cliente();

            cliente.setId(
                    resultSet.getInt("id")
            );

            cliente.setNome(
                    resultSet.getString("nome")
            );

            cliente.setDataNascimento(
                    resultSet.getObject(
                            "data_nascimento",
                            LocalDate.class
                    )
            );

            cliente.setCriadoEm(
                    resultSet.getObject(
                            "criado_em",
                            LocalDateTime.class
                    )
            );

            clientes.add(cliente);
        }

        return clientes;

    } catch (SQLException e) {

        throw new RuntimeException(e);
    }
}
```

---

# executeQuery

Diferentemente de `executeUpdate()`, quando fazemos uma consulta utilizamos:

```java
statement.executeQuery();
```

`executeQuery()` é utilizado normalmente com:

```sql
SELECT
```

Seu retorno é um:

```text
ResultSet
```

---

# ResultSet

O `ResultSet` representa o resultado retornado por uma consulta.

Imagine que o banco retornou:

```text
id | nome          | data_nascimento
--------------------------------------
1  | Ana Silva     | 1995-05-10
2  | Carlos Souza  | 1988-11-23
3  | Mariana Santos| 2001-02-15
```

O `ResultSet` permite percorrer essas linhas.

```java
while (resultSet.next()) {
}
```

Cada chamada de:

```java
resultSet.next()
```

avança para a próxima linha.

---

# Mapeando banco para objeto

Dentro do `while`, pegamos os valores da linha atual:

```java
resultSet.getInt("id");

resultSet.getString("nome");
```

Depois criamos um objeto `Cliente`.

O processo é:

```text
Linha do banco
      ↓
ResultSet
      ↓
Cliente
      ↓
List<Cliente>
```

Chamamos esse processo de **mapeamento**.

Estamos transformando dados relacionais em objetos Java.

---

# 4.3 Buscar por ID

Agora vamos buscar apenas um cliente.

## SQL

```sql
SELECT
    id,
    nome,
    data_nascimento,
    criado_em
FROM cliente
WHERE id = ?
```

## Implementação

```java
@Override
public Cliente buscarPorId(Integer id) {

    String sql = """
            SELECT
                id,
                nome,
                data_nascimento,
                criado_em
            FROM cliente
            WHERE id = ?
            """;

    try (
            Connection connection =
                    connectionFactory.getConnection();

            PreparedStatement statement =
                    connection.prepareStatement(sql)
    ) {

        statement.setInt(
                1,
                id
        );

        try (
                ResultSet resultSet =
                        statement.executeQuery()
        ) {

            if (resultSet.next()) {

                Cliente cliente =
                        new Cliente();

                cliente.setId(
                        resultSet.getInt("id")
                );

                cliente.setNome(
                        resultSet.getString("nome")
                );

                cliente.setDataNascimento(
                        resultSet.getObject(
                                "data_nascimento",
                                LocalDate.class
                        )
                );

                cliente.setCriadoEm(
                        resultSet.getObject(
                                "criado_em",
                                LocalDateTime.class
                        )
                );

                return cliente;
            }

            return null;
        }

    } catch (SQLException e) {

        throw new RuntimeException(e);
    }
}
```

Aqui não precisamos utilizar um `while`, pois esperamos no máximo um cliente.

Por isso utilizamos:

```java
if (resultSet.next()) {
}
```

---

# 4.4 Atualizar

Para alterar um cliente utilizaremos `UPDATE`.

## SQL

```sql
UPDATE cliente
SET
    nome = ?,
    data_nascimento = ?
WHERE id = ?
```

## Implementação

```java
@Override
public void atualizar(Cliente cliente) {

    String sql = """
            UPDATE cliente
            SET
                nome = ?,
                data_nascimento = ?
            WHERE id = ?
            """;

    try (
            Connection connection =
                    connectionFactory.getConnection();

            PreparedStatement statement =
                    connection.prepareStatement(sql)
    ) {

        statement.setString(
                1,
                cliente.getNome()
        );

        statement.setObject(
                2,
                cliente.getDataNascimento()
        );

        statement.setInt(
                3,
                cliente.getId()
        );

        statement.executeUpdate();

    } catch (SQLException e) {

        throw new RuntimeException(e);
    }
}
```

Observe a ordem dos parâmetros:

```sql
nome = ?,              -- 1
data_nascimento = ?    -- 2
WHERE id = ?           -- 3
```

Por isso usamos:

```java
statement.setString(1, cliente.getNome());

statement.setObject(
        2,
        cliente.getDataNascimento()
);

statement.setInt(
        3,
        cliente.getId()
);
```

---

# 4.5 Excluir

Por último, vamos remover um cliente.

## SQL

```sql
DELETE FROM cliente
WHERE id = ?
```

## Implementação

```java
@Override
public void excluir(Integer id) {

    String sql = """
            DELETE FROM cliente
            WHERE id = ?
            """;

    try (
            Connection connection =
                    connectionFactory.getConnection();

            PreparedStatement statement =
                    connection.prepareStatement(sql)
    ) {

        statement.setInt(
                1,
                id
        );

        statement.executeUpdate();

    } catch (SQLException e) {

        throw new RuntimeException(e);
    }
}
```

---

# Estrutura até aqui

Nossa aplicação agora possui:

```text
com.devforce.jdbc

├── config
│   └── ConnectionFactory.java
│
├── model
│   └── Cliente.java
│
└── dao
    ├── ClienteDAO.java
    └── ClienteDAOImpl.java
```

E o fluxo está assim:

```text
Cliente
    ↓
ClienteDAO
    ↓
ClienteDAOImpl
    ↓
ConnectionFactory
    ↓
Connection
    ↓
PreparedStatement
    ↓
PostgreSQL
```

Para consultas:

```text
PostgreSQL
    ↓
ResultSet
    ↓
Cliente
```

---

# O papel do DAO

Nesse momento é importante entender a responsabilidade de cada classe.

```text
Cliente
   ↓
representa os dados

ClienteDAO
   ↓
define as operações de persistência

ClienteDAOImpl
   ↓
implementa as operações utilizando JDBC

ConnectionFactory
   ↓
fornece conexões com o banco
```

O `ClienteDAOImpl` deve se preocupar principalmente com:

```text
SQL
Connection
PreparedStatement
ResultSet
mapeamento
```

Ele não deve concentrar as regras de negócio da aplicação.

Essas responsabilidades serão tratadas posteriormente em uma camada de **Service**.

---

# Resumo

Nesta etapa aprendemos:

* criação do modelo `Cliente`;
* padrão DAO;
* uso de interfaces;
* implementação de uma interface;
* `Connection`;
* `PreparedStatement`;
* parâmetros com `?`;
* `setString()`;
* `setInt()`;
* `setObject()`;
* `executeUpdate()`;
* `executeQuery()`;
* `ResultSet`;
* mapeamento de registros para objetos;
* `INSERT`;
* `SELECT`;
* `UPDATE`;
* `DELETE`;
* `try-with-resources`.

Agora temos uma camada responsável pela persistência dos dados.

Na próxima etapa poderemos adicionar uma camada de **Service**, separando ainda mais as responsabilidades da aplicação.
