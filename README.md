# Sistema Distribuído com RabbitMQ RPC

Um sistema distribuído implementado em Python utilizando **RabbitMQ** com padrão **RPC (Remote Procedure Call)**, demonstrando comunicação assíncrona, balanceamento de tarefas e arquitetura de microsserviços.

## 📋 Visão Geral

Este projeto implementa um sistema de computação distribuída onde um cliente central envia requisições para múltiplos serviços remotos executando em processos separados. A comunicação ocorre através de filas do RabbitMQ, permitindo escalabilidade e desacoplamento entre componentes.

### Arquitetura

```
┌─────────────┐
│   Cliente   │  (rpc_client.py)
│    RPC      │
└──────┬──────┘
       │
       ├─────────────────────────────────────────┐
       │                                         │
   ┌───▼────┐  ┌──────────┐  ┌──────────┐  ┌───▼────┐
   │ Serviço │  │ Serviço  │  │ Serviço  │  │Serviço │
   │  Soma   │  │  Média   │  │  Busca   │  │ Info   │
   └────────┘  └──────────┘  └──────────┘  └────────┘
   (service_soma.py) (service_media.py) (service_busca.py) (service_info.py)
```

## 🎯 Objetivos da Atividade

O sistema demonstra os seguintes conceitos de computação distribuída:

| Conceito | Pontuação | Implementação |
|----------|-----------|---------------|
| **Comunicação Assíncrona** | 3 pontos | Filas do RabbitMQ com padrão request/reply |
| **Balanceamento de Tarefas** | 2 pontos | Distribuição de requisições entre serviços |
| **Mecanismo RPC via RabbitMQ** | 3 pontos | Padrão RPC com correlation_id e reply_to |
| **Documentação** | 2 pontos | README com instruções e exemplos |
| **Total** | **10 pontos** | ✅ |

## 📁 Estrutura do Projeto

```
rabbitmq-rpc-distribuido/
│
├── README.md                    # Documentação do projeto
├── requirements.txt             # Dependências Python
│
├── client/
│   └── rpc_client.py           # Cliente RPC com menu interativo
│
├── services/
│   ├── service_soma.py         # Serviço: soma de dois números
│   ├── service_media.py        # Serviço: cálculo de média
│   ├── service_busca.py        # Serviço: busca em banco de dados simulado
│   └── service_info.py         # Serviço: informações do sistema
│
└── common/
    └── rpc_utils.py            # Funções utilitárias (conexão com RabbitMQ)
```

## 🚀 Como Executar

### Pré-requisitos

- **Python 3.7+**
- **RabbitMQ** instalado e em execução
- **pip** para gerenciar dependências

### 1. Instalação do RabbitMQ

#### No Ubuntu/Debian:
```bash
sudo apt-get update
sudo apt-get install rabbitmq-server
sudo systemctl start rabbitmq-server
sudo systemctl enable rabbitmq-server
```

#### No macOS (com Homebrew):
```bash
brew install rabbitmq
brew services start rabbitmq
```

#### No Windows:
Baixe o instalador em [https://www.rabbitmq.com/install-windows.html](https://www.rabbitmq.com/install-windows.html)

### 2. Instalação das Dependências

```bash
pip install -r requirements.txt
```

**Arquivo `requirements.txt`:**
```
pika==1.3.2
```

### 3. Executar os Serviços

Abra **4 terminais separados** e execute cada comando:

#### Terminal 1 - Serviço de Soma
```bash
python services/service_soma.py
```

**Saída esperada:**
```
[x] Serviço SOMA pronto...
```

#### Terminal 2 - Serviço de Média
```bash
python services/service_media.py
```

**Saída esperada:**
```
[x] Serviço MEDIA pronto...
```

#### Terminal 3 - Serviço de Busca
```bash
python services/service_busca.py
```

**Saída esperada:**
```
[x] Serviço BUSCA pronto...
```

#### Terminal 4 - Serviço de Informações
```bash
python services/service_info.py
```

**Saída esperada:**
```
[x] Serviço INFO pronto...
```

### 4. Executar o Cliente

Em um **5º terminal**, execute:

```bash
python client/rpc_client.py
```

## 💻 Uso do Cliente

O cliente apresenta um menu interativo:

```
=== Cliente RPC ===
1 - Soma
2 - Média
3 - Busca
4 - Info do Servidor

Escolha o serviço:
```

### Exemplos de Uso

#### Exemplo 1: Serviço de Soma
```
Escolha o serviço: 1
A: 10
B: 5
Resultado: 15
```

#### Exemplo 2: Serviço de Média
```
Escolha o serviço: 2
Lista (ex: 1 2 3): 10 20 30 40
Resultado: 25.0
```

#### Exemplo 3: Serviço de Busca
```
Escolha o serviço: 3
Nome para buscar: joao
Resultado: {"idade": 22, "cidade": "São Paulo"}
```

#### Exemplo 4: Informações do Sistema
```
Escolha o serviço: 4
Resultado: {"so": "Linux", "versao": "5.15.0-..."}
```

## 🔧 Detalhes Técnicos

### Padrão RPC (Remote Procedure Call)

O sistema implementa o padrão RPC clássico do RabbitMQ:

1. **Cliente** envia mensagem para fila do serviço com:
   - `reply_to`: Nome da fila de resposta
   - `correlation_id`: ID único para correlacionar requisição/resposta
   - `body`: Dados da requisição em JSON

2. **Serviço** processa a mensagem e responde:
   - Publica resposta na fila especificada em `reply_to`
   - Inclui o mesmo `correlation_id` para identificação

3. **Cliente** aguarda resposta na fila de callback

### Comunicação Assíncrona

- **Filas**: Cada serviço possui sua própria fila nomeada
- **Processamento**: Serviços processam requisições de forma assíncrona
- **Escalabilidade**: Múltiplas instâncias de um serviço compartilham a mesma fila

### Balanceamento de Tarefas

O RabbitMQ distribui automaticamente as mensagens entre os consumidores usando **round-robin**:

```python
channel.basic_qos(prefetch_count=1)  # Processa uma mensagem por vez
channel.basic_consume(queue=queue_name, on_message_callback=on_request)
```

## 📚 Componentes Principais

### `common/rpc_utils.py`

Funções utilitárias para conexão com RabbitMQ:

```python
import pika

def get_connection():
    """Cria conexão com RabbitMQ no localhost"""
    return pika.BlockingConnection(
        pika.ConnectionParameters(host='localhost')
    )

def get_channel(connection):
    """Obtém canal da conexão"""
    return connection.channel()
```

### `client/rpc_client.py`

Cliente RPC que implementa o padrão request/reply:

```python
class RPCClient:
    def __init__(self):
        self.connection = get_connection()
        self.channel = self.connection.channel()
        self.callback_queue = self.channel.queue_declare(queue='', exclusive=True).method.queue
        self.channel.basic_consume(queue=self.callback_queue, on_message_callback=self.on_response)
        self.response = None
        self.corr_id = None

    def on_response(self, ch, method, props, body):
        if props.correlation_id == self.corr_id:
            self.response = body

    def call(self, service_name, message):
        self.response = None
        self.corr_id = str(uuid.uuid4())
        self.channel.basic_publish(
            exchange='',
            routing_key=service_name,
            properties=pika.BasicProperties(
                reply_to=self.callback_queue,
                correlation_id=self.corr_id
            ),
            body=json.dumps(message)
        )
        while self.response is None:
            self.connection.process_data_events()
        return self.response.decode()
```

### Serviços

Cada serviço segue o padrão:

```python
def on_request(ch, method, props, body):
    data = json.loads(body)
    result = processar(data)  # Lógica específica do serviço
    ch.basic_publish(
        exchange='',
        routing_key=props.reply_to,
        properties=pika.BasicProperties(
            correlation_id=props.correlation_id
        ),
        body=json.dumps(result)
    )
    ch.basic_ack(method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue=queue_name, on_message_callback=on_request)
channel.start_consuming()
```

## 🔍 Fluxo de Funcionamento

### Sequência de uma Requisição

```
1. Cliente cria fila exclusiva de callback
2. Cliente envia mensagem para fila do serviço com:
   - reply_to: nome da fila de callback
   - correlation_id: UUID único
3. Serviço recebe mensagem da fila
4. Serviço processa a requisição
5. Serviço publica resposta na fila de callback
6. Cliente recebe resposta e correlaciona pelo correlation_id
7. Cliente retorna resultado ao usuário
```

### Diagrama de Sequência

```
Cliente          RabbitMQ         Serviço
  │                 │                │
  ├─ Cria fila ────>│                │
  │                 │                │
  ├─ Envia req ────>│                │
  │                 ├─ Entrega ─────>│
  │                 │                │
  │                 │              (processa)
  │                 │                │
  │                 │<─ Publica resp─┤
  │<─ Recebe ───────┤                │
  │                 │                │
```

## 🛠️ Extensões Possíveis

### 1. Adicionar Novo Serviço

Crie `services/service_novo.py`:

```python
from common.rpc_utils import get_connection, get_channel
import pika
import json

connection = get_connection()
channel = get_channel(connection)
queue_name = "service_novo"
channel.queue_declare(queue=queue_name)

def processar(data):
    # Sua lógica aqui
    return resultado

def on_request(ch, method, props, body):
    data = json.loads(body)
    result = processar(data)
    ch.basic_publish(
        exchange='',
        routing_key=props.reply_to,
        properties=pika.BasicProperties(correlation_id=props.correlation_id),
        body=json.dumps(result)
    )
    ch.basic_ack(method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue=queue_name, on_message_callback=on_request)
channel.start_consuming()
```

### 2. Múltiplas Instâncias de um Serviço

Execute o mesmo serviço em múltiplos terminais:

```bash
# Terminal 1
python services/service_soma.py

# Terminal 2
python services/service_soma.py
```

As requisições serão distribuídas automaticamente entre as instâncias.

### 3. Monitoramento com RabbitMQ Management

Acesse a interface web em `http://localhost:15672`:
- **Usuário**: guest
- **Senha**: guest

Monitore filas, mensagens e consumidores em tempo real.

## ⚠️ Tratamento de Erros

### Conexão Recusada

Se receber erro de conexão:

```
pika.exceptions.AMQPConnectionError: Connection refused
```

**Solução**: Verifique se RabbitMQ está em execução:

```bash
# Linux
sudo systemctl status rabbitmq-server

# macOS
brew services list | grep rabbitmq

# Windows
# Verifique no Gerenciador de Tarefas
```

### Fila Não Encontrada

Se um serviço não responder, verifique:

1. Se o serviço está em execução
2. Se o nome da fila está correto no cliente
3. Se RabbitMQ está acessível

## 📊 Conceitos Demonstrados

| Conceito | Descrição | Implementação |
|----------|-----------|---------------|
| **Filas de Mensagens** | Armazenamento temporário de mensagens | RabbitMQ queues |
| **RPC** | Chamada de procedimento remoto | Padrão request/reply com correlation_id |
| **Assincronismo** | Não-bloqueante | `process_data_events()` no cliente |
| **Desacoplamento** | Independência entre componentes | Comunicação via filas |
| **Escalabilidade** | Múltiplas instâncias | Round-robin automático |
| **Confiabilidade** | Garantia de entrega | Acknowledgment de mensagens |

## 🔐 Considerações de Produção

Para usar em produção, considere:

1. **Autenticação RabbitMQ**: Configure credenciais
2. **Persistência**: Ative durabilidade de filas
3. **Timeouts**: Implemente timeouts para requisições
4. **Logging**: Adicione logs estruturados
5. **Tratamento de Erros**: Implemente retry logic
6. **Monitoramento**: Configure alertas para filas cheias
7. **SSL/TLS**: Criptografe a comunicação

## 📖 Referências

- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [Pika Python Client](https://pika.readthedocs.io/)
- [RabbitMQ RPC Tutorial](https://www.rabbitmq.com/tutorials/tutorial-six-python.html)
- [Padrões de Mensageria](https://www.enterpriseintegrationpatterns.com/)

## 👤 Autor

Projeto desenvolvido como atividade de Computação Distribuída.

## 📝 Licença

Este projeto é fornecido como material educacional.

---

**Última atualização**: Dezembro 2024
