# Parte 1 — Setup e Conexão

## Objetivo

Entender como o Java se conecta ao PostgreSQL utilizando JDBC e aprender a centralizar a criação de conexões utilizando uma `ConnectionFactory`.

---

# Conexão direta

A forma mais simples de abrir uma conexão com o banco é utilizando diretamente o `DriverManager`.

```java
Connection connection = DriverManager.getConnection(
        "jdbc:postgresql://localhost:5432/devforce_jdbc",
        "postgres",
        "senha"
);
```

Nesse exemplo, informamos:

* URL do banco;
* usuário;
* senha.

O fluxo acontece assim:

```text
DriverManager
      ↓
Connection
      ↓
PostgreSQL
```

O `DriverManager` recebe as informações de acesso e devolve uma implementação de `Connection`.

A partir dessa conexão, conseguimos executar comandos SQL no banco.

---

# O problema da conexão direta

A conexão direta funciona, mas imagine repetir esse código em várias partes da aplicação:

```java
Connection connection = DriverManager.getConnection(
        "jdbc:postgresql://localhost:5432/devforce_jdbc",
        "postgres",
        "senha"
);
```

Teríamos alguns problemas:

* repetição de código;
* URL espalhada pelo projeto;
* usuário e senha duplicados;
* maior dificuldade para alterar configurações;
* classes diferentes assumindo a responsabilidade de criar conexões.

Por isso, podemos centralizar essa responsabilidade.

---

# ConnectionFactory

Uma `ConnectionFactory` é uma classe responsável por **criar conexões com o banco de dados**.

Em vez de chamar `DriverManager.getConnection()` diretamente em vários lugares, chamamos a Factory.

O fluxo passa a ser:

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

---

# ConnectionFactory — Versão 1

```java
package com.devforce.jdbc.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionFactory {

    private static final String URL =
            "jdbc:postgresql://localhost:5432/devforce_jdbc";

    private static final String USER = "postgres";

    private static final String PASSWORD = "postgres";

    public Connection getConnection() throws SQLException {

        return DriverManager.getConnection(
                URL,
                USER,
                PASSWORD
        );
    }
}
```

Agora as configurações ficam centralizadas dentro da Factory.

---

# Utilizando a Factory

Para obter uma conexão:

```java
ConnectionFactory factory =
        new ConnectionFactory();

Connection connection =
        factory.getConnection();
```

O código que precisa de uma conexão não precisa mais conhecer diretamente:

* URL;
* usuário;
* senha;
* detalhes do `DriverManager`.

---

# O que melhoramos?

Com a `ConnectionFactory`, conseguimos:

* centralizar a criação de conexões;
* evitar repetição de código;
* centralizar as configurações do banco;
* separar responsabilidades;
* facilitar futuras mudanças de configuração.

A responsabilidade da classe passa a ser clara:

```text
ConnectionFactory
        ↓
Criar Connection
```

---

# Por que Factory?

Factory significa **fábrica**.

Sua responsabilidade é criar objetos.

Nesse caso:

```text
ConnectionFactory
        ↓
     cria
        ↓
Connection
```

A classe que utiliza a conexão não precisa saber exatamente como ela foi criada.

Ela apenas solicita uma conexão.

---

# ConnectionFactory como Singleton

Existe ainda um detalhe.

Na versão atual, podemos criar várias instâncias da Factory:

```java
ConnectionFactory factory1 =
        new ConnectionFactory();

ConnectionFactory factory2 =
        new ConnectionFactory();
```

Mas a `ConnectionFactory` não possui estado e não existe uma necessidade real de criar várias instâncias dela.

Podemos então aplicar o padrão **Singleton**.

---

# O que é Singleton?

Singleton é um padrão que permite que uma classe possua apenas uma instância durante a execução da aplicação.

A ideia é:

```text
ConnectionFactory
        ↓
Uma única instância
```

---

# ConnectionFactory — Singleton

```java
package com.devforce.jdbc.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionFactory {

    private static final String URL =
            "jdbc:postgresql://localhost:5432/devforce_jdbc";

    private static final String USER = "postgres";

    private static final String PASSWORD = "postgres";

    private static final ConnectionFactory INSTANCE =
            new ConnectionFactory();

    private ConnectionFactory() {
    }

    public static ConnectionFactory getInstance() {
        return INSTANCE;
    }

    public Connection getConnection() throws SQLException {

        return DriverManager.getConnection(
                URL,
                USER,
                PASSWORD
        );
    }
}
```

---

# Entendendo o Singleton

Primeiro criamos uma única instância da Factory:

```java
private static final ConnectionFactory INSTANCE =
        new ConnectionFactory();
```

Depois impedimos que outras classes utilizem `new`:

```java
private ConnectionFactory() {
}
```

Por fim, fornecemos um método para acessar a instância existente:

```java
public static ConnectionFactory getInstance() {
    return INSTANCE;
}
```

O fluxo fica assim:

```text
ConnectionFactory.getInstance()
              ↓
     mesma instância da Factory
```

---

# Utilizando o Singleton

Agora não utilizamos mais:

```java
new ConnectionFactory();
```

Utilizamos:

```java
Connection connection =
        ConnectionFactory
                .getInstance()
                .getConnection();
```

---

# Atenção: Singleton da Factory != Singleton da Connection

Esse ponto é importante.

O Singleton é aplicado à:

```text
ConnectionFactory
```

e não à:

```text
Connection
```

A Factory possui apenas uma instância:

```text
ConnectionFactory
        ↓
    Singleton
```

Mas cada chamada de:

```java
getConnection()
```

pode criar uma nova conexão.

```text
ConnectionFactory
        │
        ├── getConnection() → Connection 1
        │
        ├── getConnection() → Connection 2
        │
        └── getConnection() → Connection 3
```

Não devemos manter uma única `Connection` aberta indefinidamente para toda a aplicação.

---

# Fluxo final

```text
Aplicação
    ↓
ConnectionFactory.getInstance()
    ↓
getConnection()
    ↓
DriverManager
    ↓
Connection
    ↓
PostgreSQL
```

---

# Testando a conexão

Podemos fazer um teste simples:

```java
import com.devforce.jdbc.config.ConnectionFactory;

import java.sql.Connection;
import java.sql.SQLException;

public class Main {

    public static void main(String[] args) {

        try (Connection connection =
                     ConnectionFactory
                             .getInstance()
                             .getConnection()) {

            System.out.println("Conexão realizada com sucesso!");

        } catch (SQLException e) {

            System.out.println("Erro ao conectar ao banco.");

            e.printStackTrace();
        }
    }
}
```

Aqui também utilizamos o `try-with-resources`.

Isso permite que a conexão seja fechada automaticamente após o uso.

---

# Resumo

Nesta etapa aprendemos:

* `DriverManager`;
* `Connection`;
* conexão direta com PostgreSQL;
* problema da repetição de código;
* `ConnectionFactory`;
* centralização das configurações;
* padrão Factory;
* padrão Singleton;
* diferença entre Singleton da Factory e da `Connection`;
* abertura e fechamento da conexão.

Na próxima etapa, utilizaremos essa conexão para executar as operações de **CRUD**.
