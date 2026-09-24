# 🐳 Laboratório Docker – Fundamentos e Prática com Containers

> **Objetivo:** Compreender os conceitos fundamentais de **containers**, dominar os principais comandos do **Docker**, entender modos de execução, identificação de containers e implantar um **servidor web NGINX** customizado.

---

## 🧠 1. Conceitos Fundamentais sobre Containers

### O que são Containers?

Containers são **unidades padronizadas de software** que empacotam o código e todas as suas dependências (bibliotecas, configurações, binários), garantindo que uma aplicação seja executada de forma consistente em qualquer ambiente.

Ao contrário de uma **máquina virtual**, que virtualiza um sistema operacional completo com seu próprio kernel, um **container compartilha o kernel do host**, tornando-o mais leve, rápido e eficiente.

#### Containers vs. Máquinas Virtuais

| Característica            | Máquina Virtual (VM)                | Container (Docker)                     |
|---------------------------|-------------------------------------|----------------------------------------|
| Virtualização             | Hardware completo (via hypervisor)  | Nível de sistema operacional           |
| Tempo de inicialização    | Minutos                             | Segundos (ou milissegundos)            |
| Tamanho típico            | Vários GB (5-20 GB)                 | Centenas de MB ou menos                |
| Isolamento                | Total (SO próprio)                  | Isolamento de processo (namespaces + cgroups) |
| Eficiência de recursos    | Menor (overhead significativo)      | Maior (compartilha kernel)             |
| Densidade                 | Dezenas por host                    | Centenas por host                      |
| Portabilidade             | Limitada (formato específico)       | Alta (OCI padrão)                      |

### Por que usar Containers?

- **Portabilidade:** O mesmo container roda em qualquer ambiente com Docker (dev, teste, produção)
- **Rapidez:** Inicialização quase instantânea comparada a VMs
- **Isolamento:** Cada aplicação roda em seu próprio ambiente isolado
- **Escalabilidade:** Fácil de replicar e distribuir horizontalmente
- **Reprodutibilidade:** Elimina o clássico "funciona na minha máquina"
- **Eficiência:** Melhor aproveitamento de recursos computacionais
- **Versionamento:** Imagens podem ser versionadas e controladas
- **Microsserviços:** Arquitetura ideal para aplicações distribuídas

### Arquitetura do Docker

```
┌─────────────────────────────────────────────┐
│         Cliente Docker (CLI)                │
│      docker run, docker build, etc.         │
└──────────────────┬──────────────────────────┘
                   │ API REST
┌──────────────────▼──────────────────────────┐
│          Docker Daemon (dockerd)            │
│  ┌────────────┐  ┌────────────┐             │
│  │ Containers │  │  Imagens   │             │
│  └────────────┘  └────────────┘             │
│  ┌────────────┐  ┌────────────┐             │
│  │   Volumes  │  │   Redes    │             │
│  └────────────┘  └────────────┘             │
└─────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│          Kernel do Sistema Operacional      │
└─────────────────────────────────────────────┘
```

> **Legenda:** *Imagens* são modelos (read-only); *containers* são instâncias em execução de uma imagem.

---

## ⚙️ 2. Pré-requisitos

Você pode realizar este laboratório de duas formas: a opção online oficial (Codespaces) ou uma instalação local do Docker:

### ☁️ Opção 1 – GitHub Codespaces (online, recomendada)

> **Caminho oficial sem instalação local.** Este repositório inclui uma configuração de dev container com **Docker-in-Docker**, permitindo executar o roteiro diretamente no navegador.

1. Tenha uma conta no **GitHub**.
2. Publique este material em um repositório GitHub ou utilize o repositório institucional já disponibilizado.
3. No GitHub, abra o repositório e acesse **Code → Codespaces → Create codespace on main**.
4. Aguarde a criação do ambiente.
5. No terminal do Codespaces, valide o ambiente:
   ```bash
   docker version
   docker info
   ```

**Observações importantes:**
- O Codespaces executa um ambiente Linux hospedado na nuvem.
- As portas `8080` e `8081` já ficam configuradas para encaminhamento automático neste repositório.
- Se ao abrir a URL encaminhada aparecer tela de login do GitHub, a porta está **privada**. Na aba **PORTS**, clique com o botão direito na porta → **Port Visibility → Public** para liberar o acesso (necessário, por exemplo, para tirar screenshots em aba anônima).
- Se o Docker não estiver disponível logo após abrir o ambiente, execute **Codespaces: Rebuild Container**.
- Contas pessoais possuem franquia mensal; após o limite, o uso pode exigir forma de pagamento ou orçamento da organização.

### 💻 Opção 2 – Docker local (Docker Desktop ou Docker Engine)
1. **Windows/Mac:** Baixe o **Docker Desktop**:  
   [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
   
2. **Linux (Ubuntu/Debian):**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   # Reinicie a sessão para aplicar permissões,
   # ou rode `newgrp docker` para aplicar sem logout.
   ```

3. **Verifique a instalação:**
   ```bash
   docker version
   docker info
   ```

---

## 🎯 3. Conceitos Essenciais: Identificação e Modos de Execução

> **⚠️ Atenção:** os comandos desta seção são **ilustrativos** — a prática começa na **Seção 4**. Se você executá-los para testar, remova os containers criados antes de iniciar o laboratório, senão o Passo 2 falhará com `Conflict. The container name "/webserver" is already in use` ou `port is already allocated`:
>
> ```bash
> docker rm -f meu-nginx webserver ubuntu-server
> docker container prune -f
> ```

### 🔑 Identificação de Containers

Todo container Docker pode ser identificado de **três formas diferentes**:

#### 1. Container ID (Hash Completo)
- Hash SHA256 de 64 caracteres
- Exemplo: `a3f5b8c7e9d1f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2`
- Gerado aleatoriamente pelo Docker a cada novo container
- Raramente usado na prática (muito longo)

#### 2. Short ID (Hash Curto)
- Primeiros 12 caracteres do hash completo
- Exemplo: `a3f5b8c7e9d1`
- **Mais comum** para identificação rápida
- Obtido com: `docker ps`

#### 3. Nome do Container
- Nome legível atribuído automaticamente ou manualmente
- Exemplo automático: `loving_tesla`, `gallant_turing`
- Exemplo manual: `meu-webserver`, `api-backend`
- Atribuído com: `--name nome_do_container`

**Exemplo prático:**
```bash
# Criar container com nome customizado
docker run -d --name meu-nginx nginx

# Criar container sem nome (nome automático será gerado)
docker run -d nginx

# Listar e ver as diferentes formas de identificação
docker ps
# CONTAINER ID   IMAGE   COMMAND                  NAMES
# a3f5b8c7e9d1   nginx   "nginx -g 'daemon of…"   meu-nginx
# b7e2c4f6a8d0   nginx   "nginx -g 'daemon of…"   romantic_fermi
```

**Importante:** Você pode usar qualquer uma das três formas para gerenciar containers:
```bash
docker stop a3f5b8c7e9d1        # Usando Short ID
docker stop meu-nginx           # Usando nome
docker logs romantic_fermi      # Usando nome automático
```

---

### 🔄 Modos de Execução: Interativo vs. Detached

O Docker oferece diferentes modos de executar containers, cada um adequado para cenários específicos:

#### 🖥️ Modo Interativo (`-it`)

O modo interativo mantém uma **sessão ativa** com o container, conectando seu terminal ao STDIN/STDOUT do processo.

**Flags:**
- `-i` (--interactive): Mantém STDIN aberto mesmo sem conexão
- `-t` (--tty): Aloca um pseudo-TTY (terminal)
- Geralmente usados juntos: `-it`

**Quando usar:**
- Executar comandos dentro do container em tempo real
- Debugging e exploração
- Aplicações que requerem input do usuário
- Shells interativos (bash, sh, etc.)

**Exemplo:**
```bash
# Executar um container Ubuntu com shell interativo
docker run -it ubuntu bash

# Você estará DENTRO do container
root@a3f5b8c7e9d1:/# ls
root@a3f5b8c7e9d1:/# pwd
root@a3f5b8c7e9d1:/# exit  # Sai e para o container
```

**Características:**
- ✅ Terminal fica "preso" ao container
- ✅ Você vê output em tempo real
- ✅ Pode interagir com o processo
- ❌ Se fechar o terminal, o container para
- ❌ Não ideal para serviços de longa duração

---

#### 🔌 Modo Detached (`-d`)

O modo *detached* executa o container em **segundo plano**, liberando o terminal.

**Flag:**
- `-d` (--detach): Executa container em background

**Quando usar:**
- Serviços web (NGINX, Apache)
- Bancos de dados (MySQL, PostgreSQL)
- APIs e microsserviços
- Qualquer aplicação que deve rodar continuamente
- Ambientes de produção

**Exemplo:**
```bash
# Executar NGINX em background
docker run -d --name webserver -p 8080:80 nginx

# O terminal retorna imediatamente com o Container ID (64 caracteres)
a3f5b8c7e9d1f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2

# Container continua rodando em background
docker ps
# CONTAINER ID   STATUS    PORTS                  NAMES
# a3f5b8c7e9d1   Up 5s     0.0.0.0:8080->80/tcp   webserver
```

**Características:**
- ✅ Terminal fica livre para outros comandos
- ✅ Container continua rodando mesmo se fechar o terminal
- ✅ Ideal para serviços de longa duração
- ✅ Pode gerenciar múltiplos containers simultaneamente
- ❌ Não vê output direto (usar `docker logs`)

---

#### 🔀 Combinando Modos: Detached + Interativo

Você pode iniciar um container em detached e depois conectar interativamente:

```bash
# 1. Iniciar em detached
docker run -d --name ubuntu-server ubuntu sleep infinity

# 2. Conectar interativamente depois
docker exec -it ubuntu-server bash

# Agora você está dentro, mas ao sair (exit),
# o container continua rodando!
```

---

#### 📊 Comparação Prática

| Aspecto                  | Modo Interativo (`-it`)     | Modo Detached (`-d`)        |
|--------------------------|-----------------------------|-----------------------------|
| Terminal                 | Conectado ao container      | Livre para outros comandos  |
| Visibilidade de logs     | Imediata no terminal        | Via `docker logs`           |
| Ao fechar terminal       | Container para              | Container continua          |
| Uso típico               | Debug, testes, shells       | Serviços, produção          |
| Exemplo                  | `docker run -it ubuntu`     | `docker run -d nginx`       |
| Input do usuário         | ✅ Possível                  | ❌ Não disponível            |
| Múltiplos containers     | Difícil (1 terminal/cont.)  | Fácil (todos em background) |

---

## 🧩 4. Laboratório Prático

**⚠️ Importante:** Durante a execução, registre cada comando e observação em um arquivo `comandos.txt`.

**Modelo sugerido para `comandos.txt`:**

```text
# Passo 1 – hello-world
docker run hello-world
# Observação: Docker baixou a imagem e exibiu a mensagem de confirmação.

# Passo 2 – NGINX detached
docker run -d --name webserver -p 8080:80 nginx:1.27
# Observação: container iniciado em background, porta 8080 mapeada.
```

---

### 🔹 Passo 1 – Executar o container "Hello World"

```bash
docker run hello-world
```

**O que acontece:**
1. Docker procura a imagem localmente
2. Não encontra, faz download do Docker Hub
3. Cria e executa o container
4. Exibe mensagem de confirmação
5. Container finaliza automaticamente

**Análise:**
```bash
# Verificar que o container foi criado mas está parado
docker ps -a

# Você verá algo como:
# CONTAINER ID   STATUS                     NAMES
# abc123def456   Exited (0) 2 minutes ago   reverent_morse
```

---

### 🔹 Passo 2 – Criar um servidor web NGINX (modo detached)

```bash
docker run -d --name webserver -p 8080:80 nginx:1.27
```

**Detalhamento das flags:**
- `-d`: Modo **detached** (background)
- `--name webserver`: Nome customizado para facilitar gerenciamento
- `-p 8080:80`: **Port mapping** (host:container)
  - `8080`: Porta no seu computador (host)
  - `80`: Porta dentro do container
- `nginx:1.27`: Nome da imagem e **tag de versão** fixada (boa prática: evite `latest` em produção)

**Acessar o servidor:**
- **GitHub Codespaces:** Abra a aba **PORTS** e clique na porta `8080`, ou use o link automático exibido no terminal para `localhost:8080`
- **Docker Desktop:** Acesse [http://localhost:8080](http://localhost:8080)

**Você deve ver:** A página padrão "Welcome to nginx!"

> 📸 **Tire agora o `screenshot-passo2.png`** (o container `webserver` será modificado no Passo 4 e removido no Passo 6).

---

### 🔹 Passo 3 – Listar containers (entendendo os estados)

```bash
# Containers em execução (running)
docker ps

# TODOS os containers (running + stopped)
docker ps -a

# Formato customizado e mais legível
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# Apenas os IDs dos containers em execução
docker ps -q

# Listar apenas containers finalizados (usando filtro)
docker ps -a -f status=exited

# Último container criado
docker ps -l
```

**Estados possíveis:**
- `Up`: Container rodando
- `Exited`: Container parado
- `Created`: Container criado mas nunca iniciado
- `Restarting`: Container reiniciando
- `Paused`: Container pausado

---

### 🔹 Passo 4 – Modo interativo: Acessar e modificar o container

```bash
# Conectar ao container em modo interativo
docker exec -it webserver bash

# Se estiver usando uma imagem baseada em Alpine (ex.: nginx:alpine),
# substitua `bash` por `sh`, pois Alpine não traz bash por padrão:
#   docker exec -it webserver sh

# Agora você está DENTRO do container!
# O prompt mudará para algo como: root@a3f5b8c7e9d1:/#

# Ver o conteúdo padrão do NGINX
cat /usr/share/nginx/html/index.html

# Modificar a página principal
# (a tag <meta charset> evita que o emoji apareça como "ðŸ³" no navegador,
#  pois o NGINX padrão não informa o charset no cabeçalho HTTP)
echo '<meta charset="utf-8"><h1>🐳 Sei tudo sobre DOCKER!</h1><p>Modificado em tempo real!</p>' > /usr/share/nginx/html/index.html

# Sair do container (ele continua rodando)
exit
```

**Recarregue o navegador** → Você verá a página modificada!

> 📸 **Tire agora o `screenshot-passo4.png`** com a página modificada.


---

### 🔹 Passo 5 – Gerenciar ciclo de vida do container

```bash
# Parar o container (graceful shutdown)
docker stop webserver

# Verificar status
docker ps -a
# STATUS: Exited (0) X seconds ago

# Iniciar novamente
docker start webserver

# Verificar status
docker ps
# STATUS: Up X seconds

# Reiniciar (stop + start)
docker restart webserver

# Pausar (congela o processo)
docker pause webserver

# Despausar
docker unpause webserver

# Forçar parada imediata (não recomendado)
docker kill webserver
```

**Diferença entre stop e kill:**
- `stop`: Envia SIGTERM, aguarda 10s, depois SIGKILL (graceful)
- `kill`: Envia SIGKILL imediatamente (forçado)

> Após `docker kill`, o container fica em estado `Exited`. Antes de seguir para o Passo 6, reinicie-o para garantir um estado consistente:
>
> ```bash
> docker start webserver
> ```

---

### 🔹 Passo 6 – Remover containers (limpeza parcial)

> A limpeza de **imagens** foi movida para o final do roteiro (após a Atividade Final), porque a imagem `nginx` ainda será reutilizada.

```bash
# Remover o container (já para automaticamente se estiver rodando)
docker rm -f webserver

# Remover TODOS os containers parados (ex.: o do hello-world)
# O comando pede confirmação: responda "y". Se responder "N", o
# `docker rmi hello-world` da limpeza final falhará (imagem em uso).
docker container prune

# Listar imagens baixadas (apenas para inspecionar)
docker images
```

---

## 🌐 5. Atividade Final – Servidor Web Personalizado

### 1. Clonar o repositório de exemplo

O repositório [`fatec-cd/pratica-docker`](https://github.com/fatec-cd/pratica-docker) contém um site estático (`index.html` + `site.css`) preparado para esta atividade. O objetivo é servi-lo com o NGINX **a partir de um container** e acessá-lo por uma **URL pública**.

```bash
# Clonar repositório com conteúdo web
git clone https://github.com/fatec-cd/pratica-docker.git
cd pratica-docker
```

**📝 Nota para Windows:**
- Se não tiver Git instalado, baixe em: [https://git-scm.com/download/win](https://git-scm.com/download/win)
- Ou baixe o ZIP do repositório diretamente no GitHub (**Code → Download ZIP**), extraia e entre na pasta extraída (ela se chamará `pratica-docker-main`)
- Confira que você está na pasta certa: `ls` (ou `dir`) deve listar `index.html`, `site.css` e `README.md`

### 2. Executar NGINX com bind mount

**Linux/macOS/GitHub Codespaces:**
```bash
docker run -d --name meuweb \
  -p 8081:80 \
  -v "$(pwd)":/usr/share/nginx/html:ro \
  nginx:1.27
```

**Windows (PowerShell):**
```powershell
docker run -d --name meuweb -p 8081:80 -v "${PWD}:/usr/share/nginx/html:ro" nginx:1.27
```

**Windows (CMD):**
```bat
docker run -d --name meuweb -p 8081:80 -v "%cd%":/usr/share/nginx/html:ro nginx:1.27
```

### 3. Entendendo o comando

**Detalhamento das flags:**
- `-d`: Modo detached (background)
- `--name meuweb`: Nome do container
- `-p 8081:80`: Mapeia porta 8081 (host) → 80 (container) (diferente da porta 8080 usada anteriormente, para evitar conflitos de portas!)
- `-v`: **Bind mount** - conecta diretório do host ao container
  - `$(pwd)` ou `${PWD}`: Diretório atual (onde está o repositório). As **aspas** são necessárias quando o caminho contém espaços (ex.: `C:\Users\João Silva\...` ou pastas do OneDrive)
  - `/usr/share/nginx/html`: Diretório padrão do NGINX no container
  - `:ro`: **Read-only** - container só pode ler, não modificar
- `nginx:1.27`: Imagem oficial do NGINX (versão fixada) baseada em Debian

### 4. Acessar a aplicação

#### GitHub Codespaces – URL pública (caminho oficial da entrega)

1. Abra a aba **PORTS** e localize a porta `8081`.
2. Clique com o botão direito na porta → **Port Visibility → Public**.
3. Copie o endereço da coluna **Forwarded Address** (formato `https://<nome-do-codespace>-8081.app.github.dev`). Se preferir, gere o endereço no terminal:
   ```bash
   echo "https://${CODESPACE_NAME}-8081.${GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}"
   ```
4. Abra essa URL em uma **aba anônima**. A página deve abrir **sem** pedir login no GitHub. Se pedir, a porta ainda está privada (volte ao passo 2).

> A URL continua ativa apenas enquanto o Codespace e o container estiverem em execução.

#### Docker Desktop (local)

Acesse [http://localhost:8081](http://localhost:8081). Esse endereço é **local** e só abre no seu computador. Para ter uma URL pública, faça a atividade no Codespaces.

**✅ Resultado esperado:** A página **"🐳 Parabéns! Seu container está no ar."**, com layout estilizado (se aparecer sem cores/estilo, o `site.css` não está na pasta montada). O quadro **Verificação do ambiente** mostra o endereço acessado e se o acesso é público.

> 📸 **Tire agora o `screenshot-final.png`** (antes da limpeza do item 5), mostrando a **barra de endereços com a URL pública** e o quadro *Verificação do ambiente* com **"Acesso público? Sim"**.

**🧪 Experimente (bind mount em ação):** com o container rodando, edite o `index.html` na pasta `pratica-docker` do host (por exemplo, troque o título) e recarregue o navegador. A alteração aparece na hora, sem recriar o container. Depois desfaça a mudança com `git checkout index.html`.

### 5. Limpar o ambiente

```bash
# Parar e remover o container
docker stop meuweb
docker rm meuweb

# Voltar um nível e conferir os dados originais intactos
cd ..
ls pratica-docker/

# Confirmar que não há mais containers do laboratório
docker ps -a
```

### 6. Limpeza final de imagens (opcional)

```bash
# Remover imagens específicas usadas no laboratório
# (se você executou os exemplos da Seção 3, remova também: nginx ubuntu)
docker rmi nginx:1.27 hello-world

# Relatar o espaço consumido por imagens e remover as não utilizadas
docker image prune
```

## 📤 6. Entrega da Atividade

Envie no **Microsoft Teams** 

1. **`comandos.txt`**: O arquivo com o histórico dos comandos executados durante o laboratório (conforme solicitado no início da **Seção 4**).

2. **Screenshots** (arquivos PNG, com os nomes sugeridos):
   - `screenshot-passo2.png` — Página padrão do NGINX rodando (porta 8080)
   - `screenshot-passo4.png` — Página modificada via `docker exec` (Passo 4)
   - `screenshot-final.png` — Atividade final em execução pela **URL pública** da porta 8081 (barra de endereços visível)

3. **`respostas.txt`** com as questões abaixo:

#### Questões obrigatórias:

**Q1.** Qual a diferença entre `docker ps` e `docker ps -a`? Qual comando mostra apenas containers finalizados?

**Q2.** Explique a diferença entre executar um container em modo interativo (`-it`) e em modo detached (`-d`). Cite dois exemplos de uso para cada modo.

**Q3.** Explique o que faz a flag `-p 8080:80` no `docker run`. O que aconteceria se você tentasse iniciar um segundo container usando também `-p 8080:80` na mesma máquina? Como poderia contornar?

**Q4.** Como um container pode ser identificado? Explique as três formas e quando usar cada uma.


---

## 🎓 7. Conceitos Extras para Estudo

### 🔹 Boas Práticas

1. **Use nomes descritivos:** `--name api-backend` melhor que `--name test1`
2. **Sempre especifique versões de imagens:** `nginx:1.27` melhor que `nginx:latest`
3. **Use volumes para dados persistentes:** Nunca confie em dados dentro do container
4. **Minimize camadas em Dockerfiles:** Combine comandos quando possível
5. **Use .dockerignore:** Evite copiar arquivos desnecessários
6. **Rode containers como non-root:** Aumente a segurança
7. **Limpeza regular:** Use `docker system prune` periodicamente

### 🔹 Troubleshooting Comum

| Problema | Solução |
|----------|---------|
| "Conflict. The container name ... is already in use" | Já existe um container (mesmo parado) com esse nome. Remova-o com `docker rm -f <nome>` ou use outro `--name` |
| "Port already allocated" | Outra aplicação usando a porta. Descubra o container conflitante com `docker ps --filter "publish=8080"` e pare-o com `docker stop`, ou use outra porta no host (ex.: `-p 8090:80`). Atenção: o `docker run` que falhou deixa um container no estado `Created` com o nome escolhido — remova-o (`docker rm <nome>`) antes de tentar de novo |
| "the input device is not a TTY" (Git Bash no Windows) | Use PowerShell/CMD, ou prefixe com `winpty`: `winpty docker exec -it webserver bash` |
| "Cannot connect to Docker daemon" | No Docker Desktop, verifique se o serviço está rodando. No GitHub Codespaces, reconstrua o dev container se o Docker não tiver iniciado corretamente |
| Porta `8080` não abre no Codespaces | Abra a aba **PORTS**, confirme se a porta foi encaminhada e clique no link gerado para visualização |
| Container para imediatamente | Processo principal terminou. Use `docker logs` para investigar |
| "No such file or directory" em volumes | Caminho incorreto. Use caminho absoluto ou `$(pwd)` |
| Imagem não encontrada | Verifique o nome. Use `docker pull` primeiro |

---

## 🧩 8. Referências e Recursos

### 📚 Documentação Oficial
- [Docker Documentation](https://docs.docker.com/) - Documentação completa e atualizada
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/) - Todos os comandos
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/) - Sintaxe do Dockerfile
- [Docker Hub](https://hub.docker.com/) - Repositório oficial de imagens
- [GitHub Codespaces](https://docs.github.com/en/codespaces) - Ambiente online oficial recomendado para este roteiro

### 🎓 Tutoriais e Cursos
- [GitHub Codespaces Overview](https://docs.github.com/en/codespaces/overview) - Visão geral do serviço
- [Port Forwarding in Codespaces](https://docs.github.com/en/codespaces/developing-in-a-codespace/forwarding-ports-in-your-codespace) - Como acessar aplicações web em execução no ambiente online
- [Docker Labs](https://github.com/docker/labs) - Repositório oficial de laboratórios
- [Docker Curriculum](https://docker-curriculum.com/) - Guia para iniciantes

### 🧰 Ambiente do Repositório
- Este repositório inclui `.devcontainer/devcontainer.json` para provisionar automaticamente o ambiente no GitHub Codespaces
- A configuração usa o feature oficial `docker-in-docker` da especificação Dev Containers

### 📖 Livros e Guias
- "Docker Deep Dive" - Nigel Poulton
- "Docker in Action" - Jeff Nickoloff
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/) - Guia oficial

### 🛠️ Ferramentas Úteis
- [Docker Desktop](https://www.docker.com/products/docker-desktop) - Interface gráfica oficial
- [Portainer](https://www.portainer.io/) - Interface web para gerenciar Docker
- [Dive](https://github.com/wagoodman/dive) - Analisar camadas de imagens
- [Hadolint](https://github.com/hadolint/hadolint) - Linter para Dockerfiles

### 🌐 Comunidade
- [Docker Community Forums](https://forums.docker.com/)
- [Stack Overflow - Docker Tag](https://stackoverflow.com/questions/tagged/docker)
- [Reddit r/docker](https://www.reddit.com/r/docker/)
- [Docker Community Slack](https://www.docker.com/docker-community)

### 📺 Vídeos (em português)
- [Canal Full Cycle](https://www.youtube.com/@FullCycle) - Desenvolvimento com Docker
- [Canal Código Fonte TV](https://www.youtube.com/@codigofontetv) - Conceitos e práticas

---

## 📋 9. Checklist de Conclusão

Antes de finalizar, verifique se você:

- [ ] Executou todos os comandos do laboratório e anotou no `comandos.txt`
- [ ] Entendeu a diferença entre modo interativo e detached
- [ ] Compreendeu as formas de identificar containers (ID, short ID, nome) e listar os finalizados com filtros
- [ ] Conseguiu acessar o NGINX no navegador nas portas `8080` e `8081`
- [ ] Modificou o conteúdo da página usando `docker exec`
- [ ] Criou o servidor web personalizado (atividade final com diretório montado) e o acessou pela URL pública
- [ ] Removeu os containers ao final do laboratório (e, opcionalmente, as imagens)
- [ ] Capturou as screenshots necessárias (com os nomes `screenshot-passo2.png`, `screenshot-passo4.png`, `screenshot-final.png`)
- [ ] Respondeu às questões em `respostas.txt`

---

🧭 **Boa prática e bons estudos!**

> *"Containers não são o futuro, são o presente. Dominar Docker é essencial para qualquer profissional de tecnologia moderno."*

---


