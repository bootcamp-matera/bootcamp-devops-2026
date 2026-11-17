# Cola do Docker e do Compose 🐳

Uma página para imprimir ou deixar aberta ao lado. Tudo aqui é o Docker do bootcamp (Aulas 07 e 08), com as mensagens de erro como elas aparecem de verdade.

## 🧠 O mapa mental: da receita à festa

```text
 📝 Dockerfile  ──docker build──▶  🎂 imagem  ──docker run──▶  🍰 container
   (a receita)                    (o bolo congelado)           (a fatia servida)

 📋 compose.yaml  ──docker compose up -d──▶  🍰🍰 vários containers + 🔌 uma rede entre eles
   (a encomenda da festa)
```

| Peça | Na padaria | O que é | Para olhar |
|------|------------|---------|------------|
| **Dockerfile** | A receita | Texto com o passo a passo para montar a imagem | `cat Dockerfile` |
| **Imagem** | O bolo congelado | Pacote pronto e imutável | `docker images` |
| **Container** | A fatia servida | Uma imagem em execução — pode haver várias da mesma imagem | `docker ps` |
| **compose.yaml** | A encomenda da festa | Diz **o que** deve estar na mesa; o Compose prepara tudo | `docker compose config` |
| **Docker Hub** | A padaria pública | Imagens prontas: `nginx`, `redis`, `alpine`... | hub.docker.com |

> **Imperativo** é ditar a receita passo a passo (`docker run`, `docker run`, `docker run`...). **Declarativo** é entregar a encomenda — "quero um site e um Redis" — e deixar o Compose descobrir os passos.

## 🗺️ O sistema da Aula 08, desenhado

```text
  SEU COMPUTADOR                     VM bootcamp
  navegador ──localhost:8080──────▶  porta 8080
  (regra WEB do VirtualBox)              │  ports: "8080:80"
  ┌─ rede interna, criada pelo Compose ──┼───────────────┐
  │                                      ▼               │
  │  ┌───────────────────┐       ┌────────────────────┐  │
  │  │ visitas           │◀──────│ site               │  │
  │  │ redis, porta 6379 │ pelo  │ nginx, porta 80    │  │
  │  │ sem ports:        │ nome  │ build: .           │  │
  │  └───────────────────┘       └────────────────────┘  │
  └──────────────────────────────────────────────────────┘
```

- O navegador chega na **porta 8080 da VM** (a regra `WEB` do setup), e o `ports: "8080:80"` leva até a porta 80 do `site`.
- Dentro da rede, os serviços se chamam **pelo nome**: o `site` acha o `visitas` pelo hostname `visitas` — o DNS da Aula 03, de graça.
- O `visitas` **não tem `ports:`**: só quem está na rede interna fala com ele. 🔐 Publique só o que precisa ser visto de fora.

## 📋 O compose.yaml da prática, linha a linha

```yaml
services:               # a lista de serviços (cada serviço = um container)
  site:                 # nome do serviço — vira hostname na rede interna
    build: .            # construa a imagem com o Dockerfile desta pasta (Aula 07)
    ports:
      - "8080:80"       # porta da VM : porta do container (o -p do docker run)
    depends_on:
      - visitas         # suba o visitas antes do site

  visitas:              # segundo serviço, alinhado ao site
    image: redis:alpine # imagem pronta do Docker Hub, sem Dockerfile
```

Os comentários (`#`) são só explicação — o arquivo da prática não precisa deles. E o `depends_on` garante a **ordem de subida**, não que o Redis já esteja pronto para atender; para o bootcamp, basta.

**YAML em 5 regras** — o erro mais comum da noite é de indentação:

| Regra | Certo | Errado (e a mensagem que aparece) |
|-------|-------|-----------------------------------|
| **2 espaços** por nível, nunca Tab | `  site:` | Tab no começo da linha → `found character that cannot start any token` |
| Mesmo nível, mesma coluna | `site:` e `visitas:` alinhados | `visitas:` colado na margem → `additional properties 'visitas' not allowed` |
| **Espaço depois dos dois-pontos** | `image: redis:alpine` | `image:redis:alpine` → `services.visitas must be a mapping` |
| Item de lista: hífen, espaço, linha própria | `- "8080:80"` | `-"8080:80"` → `services.site.ports must be a array` |
| Portas entre aspas | `"8080:80"` | Sem aspas costuma funcionar, mas com aspas não há dúvida: é texto `VM:container` |

> 🔍 **Antes de subir, confira:** `docker compose config` lê o arquivo e mostra como o Compose o entendeu — ou aponta a linha do erro. No `vi`, `:set list` mostra cada Tab como `^I` (`:set nolist` volta ao normal). Colou e virou escada? `Esc`, `:set paste`, Enter, `i` e cole de novo.

## 🐳 Docker: um container de cada vez (Aula 07)

| Comando | Faz |
|---------|-----|
| `docker run hello-world` | Baixa a imagem (se preciso) e roda um container |
| `docker run -d --name web -p 8080:80 nginx:alpine` | Em segundo plano (`-d`), com nome, porta 8080 da VM → 80 do container |
| `docker run --rm -it alpine sh` | Terminal **dentro** de um container novo; `--rm` o apaga ao sair (`exit`) |
| `docker ps` · `docker ps -a` | Containers rodando · todos, inclusive os parados |
| `docker logs web` · `docker logs -f web` | Saída do container · acompanha ao vivo (`Ctrl + C` sai) |
| `docker exec -it web sh` | Entra num container que **já está rodando** (`exit` sai) |
| `docker stop web` · `docker rm web` | Para · remove (parar não apaga!) |
| `docker build -t meu-projeto:1.0 .` | Constrói a imagem com o Dockerfile da pasta (o `.` é a pasta) |
| `docker images` · `docker rmi meu-projeto:1.0` | Lista as imagens · apaga uma imagem |

## 🧩 Compose: o sistema inteiro (Aula 08)

Rode sempre **de dentro da pasta do `compose.yaml`** (`cd ~/meu-projeto-devops`) e chame cada parte pelo **nome do serviço** (`site`, `visitas`).

| Comando | Faz |
|---------|-----|
| `docker compose config` | Confere o arquivo: mostra como o Compose o entendeu, ou a linha do erro |
| `docker compose up -d` | Sobe tudo em segundo plano; só cria ou recria o que falta ou mudou no arquivo |
| `docker compose up -d --build` | Idem, **reconstruindo** as imagens — use depois de mudar o `app/` ou o `Dockerfile` |
| `docker compose ps` | Situação de cada serviço |
| `docker compose logs visitas` · `logs -f site` | Log de um serviço · acompanha ao vivo (`Ctrl + C` sai; os containers seguem rodando) |
| `docker compose exec site sh` | Abre um terminal dentro do serviço (`exit` sai) |
| `docker compose exec visitas redis-cli GET visitas_do_site` | Roda um comando dentro do serviço |
| `docker compose restart site` | Reinicia um serviço |
| `docker compose stop` · `docker compose start` | Pausa tudo · religa |
| `docker compose down` | Derruba: remove os containers e a rede |
| `docker compose config --services` | Lista os nomes dos serviços do arquivo |

**Pausa, demolição e terra arrasada:**

| Comando | Containers | Rede | Dados dentro do container | Volumes |
|---------|------------|------|---------------------------|---------|
| `docker compose stop` | Parados, mas existem | Fica | ✅ Ficam | ✅ Ficam |
| `docker compose down` | **Removidos** | Removida | ❌ Somem — o `(nil)` do Passo 6 | ✅ Ficam |
| `docker compose down -v` | Removidos | Removida | ❌ Somem | ❌ **Apagados** |

**Os nomes que o Compose inventa** — para reconhecer o seu sistema no `docker ps`:

| Você declarou | O Compose criou | Onde ver |
|---------------|-----------------|----------|
| A pasta `meu-projeto-devops` | O projeto `meu-projeto-devops` | `docker compose ls` |
| O serviço `site`, com `build: .` | A imagem `meu-projeto-devops-site` | `docker images` |
| O serviço `site` | O container `meu-projeto-devops-site-1` | `docker ps` |
| Nada! | A rede `meu-projeto-devops_default` | `docker network ls` |

## ⚖️ Aula 07 × Aula 08: a mesma coisa, dita de dois jeitos

| Com `docker` (imperativo) | No `compose.yaml` (declarativo) |
|---------------------------|---------------------------------|
| `docker build -t meu-projeto:1.0 .` | `build: .` |
| `docker run ... nginx:alpine` | `image: nginx:alpine` |
| `--name meu-projeto` | O nome do serviço: `site:` |
| `-p 8080:80` | `ports:` + `- "8080:80"` |
| Um `docker run` por container, na ordem certa | `docker compose up -d` (a ordem vem do `depends_on`) |
| `docker stop` + `docker rm` de cada um | `docker compose down` |
| IPs anotados e redes criadas na mão | Rede interna automática; nomes no lugar de IPs |

## 💾 Para ir além: o contador que sobrevive

O `(nil)` do Passo 6 tem cura: um **volume** guarda os dados **fora** do container. No `compose.yaml`, acrescente ao `visitas` e, no final do arquivo, declare o volume (colado na margem, no mesmo nível de `services:`):

```yaml
  visitas:
    image: redis:alpine
    volumes:
      - dados:/data     # o volume "dados" na pasta onde o Redis grava

volumes:
  dados:
```

Teste: `docker compose up -d`, rode o `INCR` algumas vezes, `docker compose down`, `docker compose up -d` e o `GET` — o número voltou! Para apagar o volume de verdade: `docker compose down -v`.

## 🔐 Hábitos seguros (o compose.yaml vai para o GitHub na Aula 09)

| Hábito | Por quê | Como |
|--------|---------|------|
| Publique só o que precisa | Cada `ports:` é uma porta aberta na máquina | Banco e cache sem `ports:` — o site fala com eles pela rede interna |
| Senha **nunca** no `compose.yaml` | O arquivo vai para o Git, e o Git não esquece | Variável `${SENHA_BANCO}` + arquivo `.env` fora do Git (abaixo) |
| Fixe a versão da imagem | `redis:alpine` muda sem aviso a cada lançamento | Em projeto real: `redis:8-alpine` (versão maior) ou `redis:8.10-alpine` |
| Prefira imagens oficiais | Uma imagem é código de terceiros rodando na sua máquina | Selo *Docker Official Image* no Docker Hub |

Quando um serviço precisar de senha (um banco de verdade, por exemplo), o valor fica num arquivo `.env`, na pasta do `compose.yaml`, que o Compose lê sozinho:

```yaml
  banco:
    image: postgres:17-alpine
    environment:
      POSTGRES_PASSWORD: ${SENHA_BANCO}   # o valor vem do arquivo .env
```

```bash
vi .env                    # uma linha: SENHA_BANCO=uma-senha-forte
echo ".env" >> .gitignore  # o .env nunca vai para o Git
git status                 # o .env NAO pode aparecer na lista
```

## 🆘 Socorro

| Mensagem ou situação | O que fazer |
|----------------------|-------------|
| `found character that cannot start any token` | Tem **Tab** no arquivo: troque por espaços (no `vi`, `:set list` mostra o Tab como `^I`) |
| `mapping values are not allowed in this context` | Indentação torta: confira a linha indicada e a de cima |
| `additional properties 'xxx' not allowed` | Palavra fora do lugar: serviço fora do `services:`, ou nome errado (`service:` sem o *s*) |
| `services.xxx must be a mapping` | Faltou o espaço depois dos dois-pontos, ou o conteúdo do serviço não está indentado dentro dele |
| `services.site.ports must be a array` | Cada porta é um item de lista: `- "8080:80"`, em linha própria |
| `no configuration file provided: not found` | Você não está na pasta do projeto: `cd ~/meu-projeto-devops` |
| `compose build requires buildx 0.17.0 or later` | Falta o construtor de imagens: `sudo apt install -y docker-buildx` |
| `'compose' is not a docker command` | Plugin não instalado: `sudo apt install -y docker-compose-v2` |
| `Bind for 0.0.0.0:8080 failed: port is already allocated` | Sobrou um container da Aula 07 na 8080: `docker ps`, depois `docker stop` + `docker rm` |
| `pull access denied for ...` | Nome da imagem digitado errado (`redis:alpine`) |
| `no such service: xxx` · `service "xxx" is not running` | Nome do serviço errado (`docker compose config --services` lista os certos) ou serviço parado (`docker compose up -d`) |
| `permission denied ... docker.sock` | Sessão sem o grupo `docker`: `exit` e conecte de novo pelo SSH |
| `failed to connect to the docker API ... no such file or directory` | O serviço do Docker está parado: `sudo systemctl start docker` |
| `The "SENHA_BANCO" variable is not set` | Falta o `.env` (ou a variável dentro dele) na pasta do `compose.yaml` |
| `WARNING Memory overcommit must be enabled!` no log do Redis | Inofensivo no bootcamp — siga em frente |
| `curl: (7) Failed to connect to localhost port 8080` | O `site` não subiu: `docker compose ps` e `docker compose logs site` |
| Mudei o `app/index.html` e a página continua igual | A imagem é o bolo congelado: `docker compose up -d --build` |
| O `GET` devolve `(nil)` depois do `down` | Esperado: os dados morreram com o container. Quer guardar? Volume (acima) |
| `logs -f` não devolve o prompt | Está acompanhando ao vivo: `Ctrl + C` (os containers continuam rodando) |

## 🏁 A Aula 08 inteira, em 8 linhas

```bash
cd ~/meu-projeto-devops                                      # 1. a pasta do projeto
vi compose.yaml                                              # 2. declara o sistema
docker compose config                                        # 3. confere o arquivo
docker compose up -d                                         # 4. sobe tudo
docker compose ps                                            # 5. confere os serviços
docker compose exec visitas redis-cli INCR visitas_do_site   # 6. conversa com um serviço
docker compose down                                          # 7. derruba (e o 4 recria)
git add compose.yaml && git commit -m "adiciona compose.yaml com site e redis"   # 8. versiona
```

Veja também: [cola do terminal](cheat-sheet-terminal.md) · [cola do Git](cheat-sheet-git.md) · [cola do GitHub](cheat-sheet-github.md) · [glossário](../glossario.md).
