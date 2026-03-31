# Setup do Ambiente — Migracao Windows/WSL para Mac

Guia de configuracao do projeto [clone-tabnews](https://curso.dev) no macOS,
documentando o que foi ajustado ao migrar do Windows + WSL para Mac.

---

## Contexto

O projeto foi originalmente desenvolvido no Windows usando WSL (Windows Subsystem
for Linux) para rodar Docker. Ao migrar para Mac, o WSL nao e necessario — o
Docker Desktop roda nativamente no macOS via virtualizacao propria.

---

## Requisitos

| Ferramenta | Versao requerida | Como instalar |
|---|---|---|
| Node.js | 18.x (`lts/hydrogen`) | via nvm (ver abaixo) |
| nvm | qualquer | `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh \| bash` |
| Docker Desktop | qualquer recente | [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/) |
| npm | vem com o Node 18 | — |

---

## 1. Configurar a versao correta do Node

O projeto exige Node 18 (`lts/hydrogen`), definido no arquivo `.nvmrc`.

### Instalar e ativar via nvm

```bash
nvm install lts/hydrogen
nvm use lts/hydrogen
```

Confirmar:

```bash
node --version   # deve exibir v18.x.x
```

### Auto-switch ao entrar na pasta (opcional, mas recomendado)

Adicione ao seu `~/.zshrc` para trocar a versao automaticamente ao entrar
em qualquer projeto que tenha `.nvmrc`:

```zsh
autoload -U add-zsh-hook
load-nvmrc() {
  local nvmrc_path
  nvmrc_path="$(nvm_find_nvmrc)"
  if [ -n "$nvmrc_path" ]; then
    nvm use
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

Depois recarregue o shell:

```bash
source ~/.zshrc
```

---

## 2. Instalar dependencias do projeto

Com o Node 18 ativo, instale as dependencias:

```bash
npm install
```

---

## 3. Docker no Mac

No Mac com Docker Desktop **nao e necessario WSL**. O Docker Desktop sobe um
ambiente Linux interno automaticamente.

### Verificar se esta rodando

Abra o Docker Desktop e certifique-se de que o icone na barra de menu esta
ativo (whale icon verde/branco).

### Versoes disponiveis

```bash
docker --version          # ex: Docker version 29.x.x
docker compose version    # ex: Docker Compose version v5.x.x
docker-compose --version  # alias para o mesmo V2
```

> No Docker Desktop moderno, `docker-compose` (com hifen) e um alias para
> `docker compose` (V2). Ambos funcionam igual.

---

## 4. Subir o banco de dados (PostgreSQL)

O projeto usa um container PostgreSQL definido em `infra/compose.yaml`,
com as variaveis de ambiente em `.env.development`.

```bash
npm run services:up    # sobe o container em background
npm run services:stop  # para o container (mantem os dados)
npm run services:down  # para e remove o container
```

Para verificar se o container subiu:

```bash
docker ps
```

Voce deve ver uma linha com `postgres:16.0-alpine3.18` e status `Up`.

---

## 5. Rodar o projeto

```bash
npm run dev
```

Esse script sobe o banco (`services:up`) e inicia o Next.js na sequencia.
Acesse: [http://localhost:3000](http://localhost:3000)

### Endpoint de status

```bash
curl http://localhost:3000/api/v1/status
# {"status":"ok"}
```

---

## 6. Rodar os testes

Os testes sao de integracao e exigem que o servidor Next.js esteja rodando
(`npm run dev`) em outro terminal antes de executar:

```bash
npm test          # roda uma vez
npm run test:watch  # modo watch (reexecuta ao salvar)
```

---

## Correcoes feitas na migracao

### `package.json` — flag `-d` invalida em `down` e `stop`

**Problema:** os scripts `services:down` e `services:stop` usavam a flag `-d`
(detached), que so e valida para o comando `up`. Isso causava erro ao tentar
parar os servicos.

**Correcao:**

```diff
- "services:down": "docker-compose -f infra/compose.yaml down -d",
- "services:stop": "docker-compose -f infra/compose.yaml stop -d",
+ "services:down": "docker-compose -f infra/compose.yaml down",
+ "services:stop": "docker-compose -f infra/compose.yaml stop",
```

---

## Estrutura do projeto (resumo)

```
clone-tabnews/
├── .nvmrc                         # versao do Node (lts/hydrogen = 18.x)
├── .env.development               # variaveis de ambiente locais (nao commitar)
├── infra/
│   ├── compose.yaml               # definicao do container PostgreSQL
│   └── database.js                # client de conexao com o banco (pg)
├── pages/
│   ├── index.js                   # pagina inicial
│   └── api/v1/status/index.js     # endpoint GET /api/v1/status
└── tests/
    └── integration/api/v1/status/
        └── get.test.js            # teste de integracao do endpoint de status
```

---

## Variaveis de ambiente (`.env.development`)

```env
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=local_user
POSTGRES_DB=local_db
POSTGRES_PASSWORD=local_password
```

> Esse arquivo ja existe no repositorio pois e um ambiente local de
> desenvolvimento. Em producao as variaveis sao configuradas separadamente
> (ex: Vercel environment variables).

---

## Checklist para comecar as aulas

- [ ] `nvm install lts/hydrogen && nvm use lts/hydrogen`
- [ ] `node --version` confirma v18.x.x
- [ ] Docker Desktop esta aberto e rodando
- [ ] `npm install`
- [ ] `npm run dev` sobe sem erros
- [ ] `curl http://localhost:3000/api/v1/status` retorna `{"status":"ok"}`
