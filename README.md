## 👨‍💻 Autores

Leonardo Salazar - 557484

Alexsandro Macedo - 557068

## Apresentação

https://performance-kids-9jrdlwo.gamma.site/

# Performance Kids 🎮⚽

## 📌 Descrição do Projeto
O **Performance Kids** é um sistema em **Spring Boot 3 (Java 17)** para gerenciamento de **brinquedos esportivos** voltados a crianças até 12 anos.  
A aplicação expõe uma API REST com **CRUD completo** de **Brinquedos**, **Categorias** e **Funcionários**, usando **DTOs**, **validações**, **HATEOAS** e documentação via **Swagger OpenAPI**.

- **IDE utilizada:** IntelliJ IDEA (pode ser executado também no Eclipse/NetBeans).
- **Porta padrão:** `8081`

---

## 🏗️ Tecnologias & Dependências
- Spring Boot 3 (Web, Data JPA, Validation, HATEOAS)
- Lombok
- Oracle JDBC Driver
- Springdoc OpenAPI (Swagger UI)

<img width="799" height="447" alt="spring" src="https://github.com/user-attachments/assets/9f77a875-445a-4695-b3fe-13c6b9e97f85" />

---

## ⚙️ Configuração (`application.properties`)
```properties
server.port=8081

spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/XEPDB1
spring.datasource.username=PERFORMANCE_KIDS
spring.datasource.password=senha_aqui
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

> Use `ddl-auto=validate` em produção. O schema Oracle pode ser criado com um usuário dedicado (`PERFORMANCE_KIDS`).

---

## 📂 Estrutura de Pacotes
```
br.com.fiap.performancekids
 ┣ assembler        → ModelAssemblers (HATEOAS)
 ┣ controller       → REST Controllers (Brinquedo, Categoria, Funcionario)
 ┣ dto              → DTOs
 ┣ entity           → Entidades JPA
 ┣ repository       → Repositórios JPA
 ┣ service          → Regras de negócio
 ┗ PerformanceKidsApplication.java
```

---

## 🔑 Modelo de Dados (resumo)
### Brinquedo
- `id: Long`
- `nome: String`
- `categoria: Categoria (ManyToOne)`
- `classificacao: String` (`3-5`, `6-8`, `9-12`)
- `tamanho: String` (depende da categoria)
- `preco: BigDecimal`

### Categoria
- `id: Long`
- `nome: String` (`BOLA`, `TENIS`, `MEIA`, `ROUPA`, `RAQUETE`, `ACESSORIO`)
- `descricao: String` (opcional)

### Funcionario
- `id: Long`
- `nome: String`
- `email: String` (único)
- `senha: String`
- `cargo: String` (`ADMIN`, `OPERADOR`, ...)

---

## 🧠 Regras de Validação
- **Classificação** deve ser `3-5`, `6-8` ou `9-12`.
- **Categoria × Tamanho:**
  - `BOLA` → tamanho `3|4|5`
  - `ROUPA`/`MEIA` → `PP|P|M|G`
  - `TENIS` → `28..36`
- **Preço** `> 0`

---

## 🌐 Endpoints (CRUD) — com DTO & HATEOAS

### 1) Brinquedos
**POST `/brinquedos`** — cria
```json
{
  "nome": "Bola de Futebol Infantil",
  "categoriaId": 1,
  "classificacao": "6-8",
  "tamanho": "4",
  "preco": 79.90
}
```
**Resposta 201**
```json
{
  "id": 1,
  "nome": "Bola de Futebol Infantil",
  "categoriaId": 1,
  "categoriaNome": "BOLA",
  "classificacao": "6-8",
  "tamanho": "4",
  "preco": 79.90,
  "_links": {
    "self": { "href": "http://localhost:8081/brinquedos/1" },
    "lista": { "href": "http://localhost:8081/brinquedos" },
    "atualizar": { "href": "http://localhost:8081/brinquedos/1" },
    "deletar": { "href": "http://localhost:8081/brinquedos/1" }
  }
}
```

**GET `/brinquedos`** — lista (CollectionModel<EntityModel<DTO>>)

**GET `/brinquedos/{id}`** — busca por ID (EntityModel<DTO>)

**PUT `/brinquedos/{id}`** — atualiza
```json
{
  "nome": "Bola de Futebol Kids",
  "categoriaId": 1,
  "classificacao": "6-8",
  "tamanho": "4",
  "preco": 89.90
}
```

**DELETE `/brinquedos/{id}`** — remove (204 No Content)

---

### 2) Categorias
**POST `/categorias`**
```json
{
  "nome": "BOLA",
  "descricao": "Bolas esportivas para crianças"
}
```

**GET `/categorias`**, **GET `/categorias/{id}`**, **PUT `/categorias/{id}`**, **DELETE `/categorias/{id}`**

---

### 3) Funcionários
**POST `/funcionarios`**
```json
{
  "nome": "Ana Souza",
  "email": "ana@performancekids.com",
  "senha": "12345678",
  "cargo": "OPERADOR"
}
```

**GET `/funcionarios`**, **GET `/funcionarios/{id}`**, **PUT `/funcionarios/{id}`**, **DELETE `/funcionarios/{id}`**

---

## 🔗 HATEOAS — como usamos
- Controllers retornam **`EntityModel<DTO>`** (um recurso com `_links`)
- Listagens retornam **`CollectionModel<EntityModel<DTO>>`**
- Os links são montados por **Assemblers** (`BrinquedoModelAssembler`, etc.) com:
```java
linkTo(methodOn(BrinquedoController.class).buscarPorId(dto.getId())).withSelfRel();
```

---

## 🧪 Exemplos via cURL
**Criar Categoria**
```bash
curl -X POST http://localhost:8081/categorias \
 -H "Content-Type: application/json" \
 -d '{"nome":"BOLA","descricao":"Bolas esportivas"}'
```

**Criar Brinquedo**
```bash
curl -X POST http://localhost:8081/brinquedos \
 -H "Content-Type: application/json" \
 -d '{"nome":"Bola de Futebol Infantil","categoriaId":1,"classificacao":"6-8","tamanho":"4","preco":79.90}'
```

**Listar Brinquedos**
```bash
curl http://localhost:8081/brinquedos
```

**Atualizar Brinquedo**
```bash
curl -X PUT http://localhost:8081/brinquedos/1 \
 -H "Content-Type: application/json" \
 -d '{"id":1,"nome":"Bola Kids Pro","categoriaId":1,"classificacao":"6-8","tamanho":"4","preco":89.90}'
```

**Deletar Brinquedo**
```bash
curl -X DELETE http://localhost:8081/brinquedos/1
```

---

## 📖 Swagger (OpenAPI)
Acesse a documentação em:
```
http://localhost:8081/swagger-ui.html
```

---

## ▶️ Como executar
1. **Configurar** `application.properties` com seu Oracle.
2. **Rodar** pelo IntelliJ: `BrinquedosRevisaoApplication` (ou `PerformanceKidsApplication`), ou via Maven:
   ```bash
   mvn spring-boot:run
   ```
3. Testar no Postman/Insomnia ou pelo Swagger UI.

---

## 📝 Observações
- O campo **categoriaId** é obrigatório ao criar/atualizar Brinquedos.
- Regra de coerência **categoria × tamanho** e **classificação** já é validada no Service.
- Em produção, proteger endpoints e não expor senha em DTOs (já adotado no `FuncionarioDTO`).

---<img width="1159" height="691" alt="get_brinquedo" src="https://github.com/user-attachments/assets/95750518-9ba9-4e0d-a342-62a7f436bb2a" />
<img width="1159" height="318" alt="get_categorias" src="https://github.com/user-attachments/assets/ea9aef5a-88b5-4a6f-9185-e22b914d8e05" />
<img width="1161" height="256" alt="delete_funcionario" src="https://github.com/user-attachments/assets/89e598df-6be2-466c-b43d-509b20ed9ddb" />
<img width="1161" height="283" alt="update_funcionario" src="https://github.com/user-attachments/assets/cbed9bea-04c5-4ab8-b13d-0f14141af077" />
<img width="960" height="270" alt="categoria" src="https://github.com/user-attachments/assets/d6f0ba0b-5953-40ba-82e8-c87c024de59a" />
<img width="1159" height="540" alt="brinquedo" src="https://github.com/user-attachments/assets/eb25c694-5152-4fa8-81c1-5a955013446d" />
<img width="1159" height="318" alt="funcionario" src="https://github.com/user-attachments/assets/283fce3e-90c2-473a-b439-ec9fdfaebdaf" />
<img width="1159" height="318" alt="get_funcionario" src="https://github.com/user-attachments/assets/e5f4931f-46db-48fe-8d57-3df1e4eae39b" />






