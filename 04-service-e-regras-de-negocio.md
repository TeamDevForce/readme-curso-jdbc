# Parte 04 — Service e Regras de Negócio

## Objetivo

Nesta etapa vamos criar uma nova camada responsável pelas **regras de negócio** da aplicação.

Até aqui, nosso `ClienteDAOImpl` ficou responsável pelo acesso ao banco de dados.

Agora vamos separar mais uma responsabilidade:

```text
DAO
↓
Acesso aos dados

Service
↓
Regras de negócio
```

A ideia é evitar que validações, decisões e regras da aplicação fiquem misturadas com código de banco de dados.

---

# Por que criar uma Service?

Imagine que, antes de cadastrar um cliente, precisamos validar algumas informações.

Por exemplo:

* nome não pode ser `null`;
* nome não pode estar vazio;
* outras regras poderão ser adicionadas futuramente.

Poderíamos colocar essas validações dentro do DAO, mas isso misturaria responsabilidades.

O DAO deve se preocupar com:

```text
SQL
Connection
PreparedStatement
ResultSet
Persistência
```

Já a Service deve se preocupar com:

```text
Validações
Regras de negócio
Fluxo da aplicação
Decisões
```

---

# Separando responsabilidades

Nosso fluxo passa a ser:

```text
Main
  ↓
Service
  ↓
ClienteDAO
  ↓
ClienteDAOImpl
  ↓
JDBC
  ↓
PostgreSQL
```

Ou, olhando apenas para as camadas principais:

```text
Service
   ↓
interface ClienteDAO
   ↓
ClienteDAOImpl
   ↓
JDBC
```

A Service conhece a interface `ClienteDAO`, e não precisa conhecer detalhes internos do JDBC.

---

# Criando ClienteService

```java
package com.devforce.jdbc.service;

import com.devforce.jdbc.dao.ClienteDAO;
import com.devforce.jdbc.model.Cliente;

public class ClienteService {

    private final ClienteDAO clienteDAO;

    public ClienteService(ClienteDAO clienteDAO) {
        this.clienteDAO = clienteDAO;
    }

    public void cadastrar(Cliente cliente) {

        if (cliente.getNome() == null ||
            cliente.getNome().isBlank()) {

            throw new IllegalArgumentException(
                    "Nome é obrigatório"
            );
        }

        clienteDAO.salvar(cliente);
    }
}
```

---

# Entendendo a Service

Observe que a Service recebe um `ClienteDAO` pelo construtor:

```java
private final ClienteDAO clienteDAO;

public ClienteService(ClienteDAO clienteDAO) {
    this.clienteDAO = clienteDAO;
}
```

Isso significa que a Service depende da **interface**:

```text
ClienteDAO
```

e não diretamente de:

```text
ClienteDAOImpl
```

O fluxo fica assim:

```text
ClienteService
      ↓
ClienteDAO
      ↑
ClienteDAOImpl
```

A implementação concreta é fornecida de fora.

---

# Injeção de dependência

Quando fazemos:

```java
new ClienteService(clienteDAO);
```

estamos fornecendo para a Service uma dependência que ela precisa para funcionar.

Esse conceito é chamado de:

**Injeção de Dependência**

Em vez de a própria Service criar o DAO:

```java
ClienteDAO clienteDAO =
        new ClienteDAOImpl();
```

ela recebe esse objeto através do construtor.

Isso deixa as classes menos acopladas.

---

# Regra de negócio

Dentro do método `cadastrar()`, temos uma regra:

```java
if (cliente.getNome() == null ||
    cliente.getNome().isBlank()) {

    throw new IllegalArgumentException(
            "Nome é obrigatório"
    );
}
```

Essa validação pertence à aplicação e não ao acesso ao banco.

Por isso ela fica na Service.

Depois que a regra é validada, a Service chama:

```java
clienteDAO.salvar(cliente);
```

O DAO então executa a operação de persistência.

---

# Fluxo do cadastro

O fluxo completo fica assim:

```text
Main
  ↓
ClienteService.cadastrar()
  ↓
Valida regra de negócio
  ↓
ClienteDAO.salvar()
  ↓
ClienteDAOImpl
  ↓
PreparedStatement
  ↓
PostgreSQL
```

Se o nome estiver inválido:

```text
Cliente
   ↓
ClienteService
   ↓
Validação falha
   ↓
Exception
```

Se estiver válido:

```text
Cliente
   ↓
ClienteService
   ↓
ClienteDAO
   ↓
Banco de dados
```

---

# Regra de negócio não deve ficar no DAO

Uma ideia importante desta etapa é:

> O DAO deve lidar com persistência.
> A Service deve lidar com regras de negócio.

Por exemplo, não seria interessante colocar dentro do `ClienteDAOImpl`:

```java
if (cliente.getNome() == null ||
    cliente.getNome().isBlank()) {

    throw new IllegalArgumentException(
            "Nome é obrigatório"
    );
}
```

Porque o DAO começaria a assumir duas responsabilidades:

```text
Persistência
+
Regra de negócio
```

Ao utilizar uma Service, separamos essas responsabilidades.

---

# Main

Agora podemos montar nossa aplicação na classe `Main`.

```java
package com.devforce.jdbc;

import com.devforce.jdbc.dao.ClienteDAO;
import com.devforce.jdbc.dao.ClienteDAOImpl;
import com.devforce.jdbc.model.Cliente;
import com.devforce.jdbc.service.ClienteService;

import java.time.LocalDate;

public class Main {

    public static void main(String[] args) {

        ClienteDAO clienteDAO =
                new ClienteDAOImpl();

        ClienteService clienteService =
                new ClienteService(clienteDAO);

        Cliente cliente =
                new Cliente(
                        "Marcelo",
                        LocalDate.of(1992, 5, 20)
                );

        clienteService.cadastrar(cliente);
    }
}
```

---

# Entendendo a Main

Primeiro criamos a implementação do DAO:

```java
ClienteDAO clienteDAO =
        new ClienteDAOImpl();
```

Depois passamos essa dependência para a Service:

```java
ClienteService clienteService =
        new ClienteService(clienteDAO);
```

Criamos um cliente:

```java
Cliente cliente =
        new Cliente(
                "Marcelo",
                LocalDate.of(1992, 5, 20)
        );
```

E realizamos o cadastro:

```java
clienteService.cadastrar(cliente);
```

O fluxo completo agora é:

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

---

# Estrutura do projeto

Até aqui teremos algo parecido com:

```text
com.devforce.jdbc

├── Main.java
│
├── config
│   └── ConnectionFactory.java
│
├── model
│   └── Cliente.java
│
├── dao
│   ├── ClienteDAO.java
│   └── ClienteDAOImpl.java
│
└── service
    └── ClienteService.java
```

Cada pacote possui uma responsabilidade principal:

```text
model
↓
Representação dos dados

dao
↓
Persistência

service
↓
Regras de negócio

config
↓
Configuração e conexão
```

---

# Atividade prática

O exemplo acima implementa apenas o fluxo de cadastro.

Agora cada aluno deverá completar as demais operações utilizando a mesma estrutura.

Implementar na `ClienteService` e utilizar na `Main`:

```text
buscar por ID
listar todos
atualizar
excluir
```

A ideia é que o aluno pratique o fluxo completo:

```text
Main
↓
Service
↓
DAO
↓
JDBC
↓
Banco de dados
```

---

# Desafio

Depois de concluir o CRUD de clientes, utilize a mesma estrutura para criar outro pequeno projeto.

Algumas possibilidades:

```text
Produto
Funcionário
Aluno
Livro
Pedido
Curso
```

Procure manter a organização:

```text
model
dao
service
config
```

O objetivo é perceber que a mesma estrutura pode ser reutilizada em vários projetos Java.

---

# Resumo

Nesta etapa aprendemos:

* o papel da camada Service;
* separação de responsabilidades;
* diferença entre regra de negócio e persistência;
* injeção de dependência pelo construtor;
* dependência através de interface;
* comunicação entre Service e DAO;
* montagem das dependências na `Main`;
* fluxo completo da aplicação.

Agora nossa aplicação está dividida em responsabilidades mais claras:

```text
Main
  ↓
Service
  ↓
DAO
  ↓
JDBC
  ↓
PostgreSQL
```

Essa organização prepara o caminho para arquiteturas maiores e também ajuda a entender melhor como frameworks como Spring organizam aplicações Java.
