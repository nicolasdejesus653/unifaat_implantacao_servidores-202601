# TF008 — Docker Swarm

**Aluno:** José Henrique Teixeira Luiz
**RA:** 3225002
**Disciplina:** Implementação de servidor e nuvem (cloud)
**Aula:** 8 — Introdução à Orquestração (Docker Swarm)

---

## Questões Teóricas

### Questão 1: Conceito de Cluster

A diferença fundamental entre Docker Compose e Docker Swarm está no **escopo de execução**:

- **Docker Compose** gerencia múltiplos containers (um *Stack*) rodando em um **único host**. É voltado para desenvolvimento local e ambientes simples, onde todos os serviços convivem na mesma máquina.
- **Docker Swarm** gerencia o *Stack* distribuído em um **cluster de múltiplos hosts** (vários nós conectados). É voltado para produção, oferecendo alta disponibilidade, balanceamento de carga e tolerância a falhas — se um host cai, o Swarm redireciona os containers para outros nós saudáveis.

Em resumo: Compose = 1 máquina; Swarm = várias máquinas trabalhando como uma só.

---

### Questão 2: Funções dos Nós

- **Manager:** É o nó responsável pelo **controle e orquestração** do cluster. Mantém o estado desejado, agenda os *services*, distribui as tarefas para os workers, expõe a API do Swarm e toma todas as decisões administrativas.
- **Worker:** É o nó **executor**. Recebe as tarefas (tasks) atribuídas pelos Managers e executa os containers correspondentes. Não toma decisões de orquestração — apenas roda o que o Manager mandou.

Observação: um Manager também pode atuar como Worker (executar tasks), mas um Worker puro não pode gerenciar.

---

### Questão 3: Inicialização do Swarm

**a) Comando para inicializar um novo Cluster Swarm:**

```bash
docker swarm init
```

**b) Driver de Rede padrão para comunicação entre Services em diferentes Hosts:**

**Overlay** — É uma rede virtual que abstrai a comunicação entre containers em hosts diferentes do cluster, fazendo parecer que estão na mesma rede local.

---

### Questão 4: Criação de Service

**a) Comando para criar o service `web-escalavel` com 3 réplicas:**

```bash
docker service create --name web-escalavel --replicas 3 nginx:alpine
```

**b) Comando para visualizar o status em tempo real das 3 réplicas:**

```bash
docker service ps web-escalavel
```

---

### Questão 5: Atualização e Escalabilidade

**a) Comando para aumentar de 3 para 5 réplicas:**

```bash
docker service scale web-escalavel=5
```

**b) Termo que descreve a capacidade de realocar instâncias automaticamente quando um nó falha:**

**Self-healing** (auto-recuperação), também chamado de **reconciliação de estado**. O Swarm mantém continuamente o estado real igual ao estado desejado: se foram pedidas 5 réplicas e uma delas some por causa de um nó caído, o Swarm cria automaticamente outra réplica em um nó saudável.

---

## Tarefa Prática Integrada

### Passo 1 — Inicialização do Cluster

**Comandos executados:**

```bash
docker swarm leave --force
docker swarm init
```

**Saída:**

```
=== PASSO 1.1: Tentativa de limpeza ===
Error response from daemon: This node is not part of a swarm

=== PASSO 1.2: Inicialização do Swarm ===
Swarm initialized: current node (wwsb6cgxwyty1cju6l1bummta) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-41coedw0y1smnkkz3g1tisjimfku7skmuhe0v0ooyk1r1vat4g-1cjm5g7rq1yrjz9j5p02hm5xe 192.168.65.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

> Observação: o comando `docker swarm leave --force` retornou erro porque ainda não havia swarm ativo — comportamento esperado em um ambiente limpo.

---

### Passo 2 — Deploy do Serviço

**Comando executado:**

```bash
docker service create --name app-stack-tf9 --replicas 4 --publish 8001:80 nginx:alpine
```

**Saída:**

```
fog42wpi0y3tqfbd1963bv4ap
overall progress: 4 out of 4 tasks
verify: Service fog42wpi0y3tqfbd1963bv4ap converged
```

---

### Passo 3 — Validação e Evidências

#### Evidência 1 — Status das 4 réplicas

**Comando executado:**

```bash
docker service ls
docker service ps app-stack-tf9
```

**Saída:**

```
ID             NAME            MODE         REPLICAS   IMAGE          PORTS
fog42wpi0y3t   app-stack-tf9   replicated   4/4        nginx:alpine   *:8001->80/tcp

ID             NAME              IMAGE          NODE             DESIRED STATE   CURRENT STATE           ERROR     PORTS
uvl78vbk8gft   app-stack-tf9.1   nginx:alpine   docker-desktop   Running         Running 2 minutes ago
l2g6nth99tpa   app-stack-tf9.2   nginx:alpine   docker-desktop   Running         Running 2 minutes ago
9ho3ot9r4w4l   app-stack-tf9.3   nginx:alpine   docker-desktop   Running         Running 2 minutes ago
s84slq8tg0dc   app-stack-tf9.4   nginx:alpine   docker-desktop   Running         Running 2 minutes ago
```

✅ **As 4 réplicas estão `Running` e o service está com `4/4`.**

#### Evidência 2 — Acesso externo via curl

**Comando executado:**

```bash
curl localhost:8001
```

**Saída (HTML do nginx):**

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

**Headers HTTP da resposta:**

```
HTTP/1.1 200 OK
Server: nginx/1.29.8
Date: Mon, 27 Apr 2026 23:30:13 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Tue, 07 Apr 2026 12:09:53 GMT
Connection: keep-alive
ETag: "69d4f411-380"
Accept-Ranges: bytes
```

✅ **A aplicação respondeu com HTTP 200 OK servindo a página padrão do nginx.**

---

### Passo 4 — Escalabilidade (reduzir de 4 para 1)

**Comando executado:**

```bash
docker service scale app-stack-tf9=1
```

**Saída:**

```
app-stack-tf9 scaled to 1
overall progress: 1 out of 1 tasks
verify: Service app-stack-tf9 converged
```

**Status após escalar:**

```
ID             NAME            MODE         REPLICAS   IMAGE          PORTS
fog42wpi0y3t   app-stack-tf9   replicated   1/1        nginx:alpine   *:8001->80/tcp

ID             NAME              IMAGE          NODE             DESIRED STATE   CURRENT STATE           ERROR     PORTS
uvl78vbk8gft   app-stack-tf9.1   nginx:alpine   docker-desktop   Running         Running 2 minutes ago
```

✅ **O service foi reduzido para `1/1` réplica com sucesso.**

---

### Passo 5 — Limpeza Final

**Comandos executados:**

```bash
docker service rm app-stack-tf9
docker swarm leave --force
```

**Saída:**

```
=== PASSO 5.1: Remover service ===
app-stack-tf9

=== PASSO 5.2: Sair do swarm ===
Node left the swarm.

=== Confirmação: swarm inativo ===
 Swarm: inactive
```

✅ **Service removido e nó saiu do swarm. Ambiente limpo.**
