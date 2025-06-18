# Guntz Posts - Solution

Guntz Posts é uma solução de microsserviços para gestão e criação de posts via API REST construído com Spring Boot, Docker e Rabbitmq.

Post-Service, esse microsserviço exibe posts com recurso de páginação, além de pesquisa individual por Id (UUID),

Válida e armazena apenas posts que cumpram com todos requisitos inicias como as propriedades: title, body e author serem obrigatórias.

Em segundo plano envia os posts para uma fila do Rabbitmq (text-processor-service.post-processing.v1.q)

Esse microsserviço faz comunicação asíncrona com outro microsserviço (<b>Guntz Text Processor Service</b>) via RabbitMQ.

Guntz-Text-Processor-Service, é responsável pelo processamento dos posts que foram enviados anteriormente pelo outro microsserviço(Guntz Post Service).

Ele vai consumir a fila (text-processor-service.post-processing.v1.q), e na sequência iniciará a contagem de palavras no texto e o cálculo de um valor estimado com base na quantidade de palavras.

Uma vez processado, o resultado será enviado para a fila (post-service.post-processing-result.v1.q) que por sua vez será consumido pelo microsserviço Guntz Post Service.

## ✨ Funcionalidades

- 💾 **Armazenamento**: Persiste posts que cumprem com os requisitos.
- 🔍 **Consulta**: Busca por ID e listagem com páginação
- ⚡ **Comunicação Resiliente**: Tratamento de erros em diversas camadas com Spring Validation e também com tratamento de exceções. 
- ⚡ **Aplicação com Entrega Confiável**: Com uso de Mensageria via Rabbitmq, conseguimos garantindo maior confiabilidade na entrega das mensagens além de maior escabilidade

## 🛠️ Stack Tecnológica

- **Java 17** - Linguagem de programação
- **Spring Boot 3.x** - Framework principal
- **Spring Boot AMQP** - Stack de Mensageria via Rabbitmq, garantindo maior entrega e escabilidade dos serviços.
- **Spring Validation** - Tratamento de erros entre cliente e serviço
- **Spring Doc OpenAPI WebMvc UI** - Documentação da api de forma descomplicada
- **Spring Boot Devtools** - Para melhor produtividade durante o desenvolvimento
- **H2 Database** - Banco de dados em memória
- **Docker** - Para melhor automatização, implantação, dimensionamento e gerenciamento de aplicações em contêineres
- **Docker Compose** - Ferramenta que simplifica o desenvolvimento e gerenciamento de aplicações com múltiplos contêinere
- **Lombok** - Facilitador de escrita de código limpo
- **UUID Generator** - Gerenciador de Id com UUID
- **Maven** - Gerenciador de dependências
- **Log Slf4j** - Log com Slf4j 

## 🚀 Início Rápido

### Pré-requisitos

- ☕ JDK 17+
- 🐘 Maven
- 🔧 Git
- 🔧 Rabbitmq
- 🔧 Docker
- 🔧 Docker Compose

### Instalação e Execução

### Instalação e Execução

1. **Clone o repositório com submódulos**
   ```bash
   git clone --recurse-submodules https://github.com/ricardoguntzell/guntz-posts-meta.git guntz-posts
   ```
2. **Inicie o PostService** (novo terminal)
   ```bash
   cd post-service
   ./mvnw spring-boot:run
   ```
  > 🌐 Serviço disponível em: http://localhost:8080

3. **Inicie o TextProcessorService**
   ```bash
   cd text-processor-service
   ./mvnw spring-boot:run
   ```
  > 🌐 Serviço disponível em: http://localhost:8081

### Verificação Rápida

## 📖 Documentação da API
- http://localhost:8080/swagger-ui/index.html
- http://localhost:8081/swagger-ui/index.html

#### Criar Post
```http
POST /api/posts
Content-Type: application/json

{
    "title": "string",
    "body": "string",
    "author": "string"
}
```

**Respostas:**
- `201 Created` - post validado e criado
- `400 Bad Request` - Não preenchimento de algum campo

#### Buscar Post
```http
GET /api/posts/{id}
```

**Respostas:**
- `200 OK` - Post encontrado
- `404 Not Found` - Post não existe

#### Listar Post
```http
GET /api/posts?page=0&size=20
```

## 📖 Resumo Text Processor Service
- Este microsserviço consiste em consumir uma fila do rabbitmq, realizar o cálculo de algumas propriedades, produzir uma nova mensagem com esse resultado e gravar numa fila.

#### Mensagem Lida da fila Rabbitmq - Formato jSon
```
{
    "postId": "string", //UUID
    "postBody": "string"
}
```

#### Mensagem Produzida e postada na fila Rabbitmq - Formato jSon
```
{
    "postId": "string", //UUID
    "wordCount": 123,
    "calculatedValue": 12.3
}
```

## ⚙️ Configurações e Regras

### Validações
- **IDs**: Devem ser UUIDs válidos
- **Body**: Todos os campos são Obrigatórios
- **summary**: summary contém as 3 primeiras linhas do body.
- **Cacular o body**: A quantidade de palavras no corpo do texto
- **Cacular o valor estimado das palavras no body**: O valor estimado (palavras * $0,10).

### Tratamento de Erros

| Cenário | Código HTTP | Descrição |
|---------|-------------|-----------|
| Post não encontrado | `404` | ID não existe na base |
| Post com problema no body | `400` | Algum campo não foi informado |

### Fluxo de Dados

1. **Recepção**: PostService recebe requisição
2. **Validação**: Verifica se todas informações foram passadas/preenchidas
3. **Persistência**: Armazena os dados em DB
4. **Postagem em Exchange**: insere a mensagem na exchange do rabbitmq que posteiormente será lida pelo outro microserviço que por sua vez vai realizar o binding para a fila "text-processor-service.post-processing.v1.q"
5. **Escutar de Forma Assincrona o processamento do Text-Processor-Service**: Esse outro microserviço vai ler a mensagem através de binding, realizar um cálculo, determinar o valor de algumas propriedades e salvar numa exchange
6. **Leitura da Mensagem da Fila**: Faz o binding e consome a mensagem produzida pelo outro microsserviço da fila "post-service.post-processing-result.v1.q"
7. **Persistência e Atualização**: Atualiza os dados do Post em DB 