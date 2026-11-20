# Blog Pessoal — API REST

API REST para um blog pessoal: usuários se cadastram, autenticam com JWT e publicam postagens organizadas por temas. O projeto inclui documentação interativa com Swagger, testes de integração com banco em memória e configuração de deploy com Docker e PostgreSQL.

O front-end que consome esta API está em [Blogpessoal-react](https://github.com/GuilhermeKaynam/Blogpessoal-react).

## Objetivo

Praticar a construção de um back-end completo em camadas — do modelo de dados à segurança e ao deploy — seguindo o padrão de projeto usado em aplicações Spring Boot de mercado.

## Funcionalidades

- **Autenticação stateless com JWT**: cadastro e login públicos; todas as outras rotas exigem `Authorization: Bearer <token>`.
- **Senhas criptografadas** com BCrypt antes de serem persistidas.
- **Usuários**: cadastro (sem permitir e-mail duplicado), login, atualização, listagem e busca por ID.
- **Temas**: CRUD completo e busca por descrição.
- **Postagens**: CRUD completo, busca por título e vínculo com um tema e um usuário; a data é atualizada automaticamente a cada alteração.
- **Validação de entrada** com Bean Validation (e-mail válido, tamanho mínimo de senha, limites de título e texto).
- **Documentação Swagger/OpenAPI** gerada a partir dos controllers.
- **Perfis de configuração**: `dev` (MySQL local) e `prod` (PostgreSQL via variáveis de ambiente).

## Tecnologias

- Java 17
- Spring Boot 3.3 (Web, Data JPA, Validation, Security)
- JJWT — geração e validação de tokens
- SpringDoc OpenAPI (Swagger UI)
- MySQL (desenvolvimento) · PostgreSQL (produção) · H2 (testes)
- JUnit 5 + `TestRestTemplate`
- Docker (build multi-stage)

## Arquitetura

O código segue a divisão em camadas do Spring:

| Camada | Responsabilidade |
|---|---|
| `controller` | Endpoints REST de usuários, temas e postagens |
| `service` | Regras de negócio de usuário: cadastro, atualização, criptografia de senha e autenticação |
| `repository` | Interfaces Spring Data JPA, incluindo consultas por título, descrição e e-mail |
| `model` | Entidades JPA (`Usuario`, `Tema`, `Postagens`) e o DTO `UsuarioLogin` |
| `security` | Filtro JWT, geração/validação de token, `UserDetailsService` e regras de autorização |
| `configuration` | Configuração do Swagger/OpenAPI |

**Relacionamentos:** um `Usuario` tem várias `Postagens`; um `Tema` tem várias `Postagens`; cada `Postagem` pertence a um usuário e a um tema.

## Estrutura de diretórios

```
Blog-Pessoal/
├── Dockerfile
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/generation/blogpessoal/
    │   │   ├── configuration/   # SwaggerConfig
    │   │   ├── controller/      # UsuarioController, TemaController, PostagemController
    │   │   ├── model/           # Usuario, Tema, Postagens, UsuarioLogin
    │   │   ├── repository/      # Repositórios JPA
    │   │   ├── security/        # JWT e Spring Security
    │   │   └── service/         # UsuarioService
    │   └── resources/
    │       ├── application.properties        # perfil ativo + Swagger
    │       ├── application-dev.properties    # MySQL local
    │       └── application-prod.properties   # PostgreSQL via variáveis de ambiente
    └── test/
        ├── java/.../controller/UsuarioControllerTest.java
        └── resources/application.properties  # H2 em memória
```

## Endpoints

| Método | Rota | Descrição | Acesso |
|---|---|---|---|
| `POST` | `/usuarios/cadastrar` | Cadastra um usuário | Público |
| `POST` | `/usuarios/logar` | Autentica e devolve o token JWT | Público |
| `GET` | `/usuarios/all` | Lista usuários | Token |
| `GET` | `/usuarios/{id}` | Busca usuário por ID | Token |
| `PUT` | `/usuarios/atualizar` | Atualiza um usuário | Token |
| `GET` | `/temas` | Lista temas | Token |
| `GET` | `/temas/{id}` | Busca tema por ID | Token |
| `GET` | `/temas/descricao/{descricao}` | Busca temas pela descrição | Token |
| `POST` / `PUT` | `/temas` | Cria / atualiza um tema | Token |
| `DELETE` | `/temas/{id}` | Remove um tema | Token |
| `GET` | `/postagens` | Lista postagens | Token |
| `GET` | `/postagens/{id}` | Busca postagem por ID | Token |
| `GET` | `/postagens/titulo/{titulo}` | Busca postagens pelo título | Token |
| `POST` / `PUT` | `/postagens` | Cria / atualiza uma postagem | Token |
| `DELETE` | `/postagens/{id}` | Remove uma postagem | Token |

A documentação interativa fica disponível na raiz da aplicação (Swagger UI) e o JSON da especificação em `/v3/api-docs`.

## Como executar localmente

### Pré-requisitos

- Java 17+
- MySQL em execução local (para o perfil `dev`)

### Passos

```bash
git clone https://github.com/GuilhermeKaynam/Blog-Pessoal.git
cd Blog-Pessoal
```

1. Em `src/main/resources/application.properties`, altere o perfil ativo para desenvolvimento:
   ```properties
   spring.profiles.active=dev
   ```
2. Em `application-dev.properties`, ajuste usuário e senha do seu MySQL. O banco é criado automaticamente na primeira execução.
3. Rode a aplicação:
   ```bash
   ./mvnw spring-boot:run
   ```

A API sobe em `http://localhost:8080`.

## Variáveis de ambiente (produção)

O perfil `prod` lê a conexão com o PostgreSQL de variáveis de ambiente — nenhuma credencial fica no código:

| Variável | Descrição |
|---|---|
| `POSTGRESHOST` / `POSTGRESPORT` / `POSTGRESDATABASE` | Endereço, porta e nome do banco |
| `POSTGRESUSER` / `POSTGRESPASSWORD` | Credenciais do banco |

### Docker

```bash
docker build -t blog-pessoal .
docker run -p 8080:8080 --env-file .env blog-pessoal
```

## Testes

```bash
./mvnw test
```

`UsuarioControllerTest` sobe a aplicação em porta aleatória com banco H2 em memória e valida: cadastro de usuário, bloqueio de e-mail duplicado, atualização e listagem.

## Melhorias futuras

- Mover a chave de assinatura do JWT para uma variável de ambiente.
- Ampliar a cobertura de testes para temas e postagens.
- Adicionar paginação às listagens.
