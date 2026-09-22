# Passeios-App

Repositório de estudos de Angular, reunindo projetos feitos ao longo de um curso (baseado no Angular 19) e experimentos próprios. Cada pasta é um workspace Angular independente, com seu próprio `package.json`, `angular.json` e histórico de commits.

## Estrutura do repositório

```
Passeios-App/
├── conceitos-basicos/       # exercícios isolados de fundamentos do Angular
├── crud-angular-material/   # CRUD completo com Angular Material
└── passeio-app/             # aplicação maior: catálogo de passeios com auth Google
```

> **Atenção — submódulos incompletos.** As três pastas acima são rastreadas pelo Git como *gitlinks* (`git ls-files` mostra modo `160000` para elas), mas **não existe um arquivo `.gitmodules`** neste repositório. Na prática isso significa que um `git clone` normal deste repo trará as pastas **vazias** — o conteúdo só existe localmente, em cada subpasta que também é seu próprio repositório Git independente. Para trabalhar em qualquer um dos projetos, `cd` até a subpasta correspondente e use o Git de dentro dela normalmente. Se o objetivo for publicar os três projetos de fato integrados ao repositório principal, é preciso ou (a) configurar `.gitmodules` corretamente apontando para os remotes de cada subprojeto, ou (b) remover as referências de gitlink e versionar o conteúdo diretamente neste repo.

## Requisitos gerais

- **Node.js 20+** (os Dockerfiles usam `node:22-alpine`)
- **Angular CLI 19** (`npm i -g @angular/cli@19`) — todos os três projetos foram gerados com a versão 19.0.2
- Cada projeto tem suas próprias dependências; rode `npm install` dentro da pasta antes de qualquer comando `ng`

---

## `conceitos-basicos`

Projeto de exercícios soltos para fixar fundamentos do Angular — componentes, data binding, formulários simples. Não tem roteamento configurado (`app.routes.ts` está vazio); os componentes são exemplos isolados.

**Conteúdo:**
- `helloworld/` — primeiro componente, "olá mundo"
- `minhapagina/` — estrutura básica de componente (template, estilo, classe)
- `calculadora/` — formulário com operações e data binding
- `lista-compras/` — lista dinâmica (`*ngFor`), manipulação de array em memória

**Rodando:**
```bash
cd conceitos-basicos
npm install
npm start        # ng serve — http://localhost:4200
```

---

## `crud-angular-material`

CRUD de clientes com [Angular Material](https://material.angular.io/) e [`@angular/flex-layout`](https://github.com/angular/flex-layout) para o layout responsivo. Cadastro e consulta persistem em `localStorage` (sem backend).

**Funcionalidades:**
- Cadastro de cliente com máscara de CPF e data (`ngx-mask`)
- Seleção em cascata de UF → Município, consumindo a [BrasilAPI](https://brasilapi.com.br/)
- Consulta com busca por nome, edição e exclusão (com confirmação inline)
- Tabela Angular Material (`mat-table`)

**Stack:** Angular 19, Angular Material 19, RxJS, `uuid` (geração de IDs), `ngx-mask`.

**Rodando:**
```bash
cd crud-angular-material
npm install
npm start        # ng serve — http://localhost:4200
```

Rotas principais: `/cadastro` e `/consulta`.

---

## `passeio-app`

O projeto mais completo do repositório: um catálogo de passeios/lugares por categoria, com autenticação via Google (OAuth2/OIDC) e uma API fake servida por `json-server`. Estruturado em módulos Angular (`NgModule`), não em standalone components.

**Funcionalidades:**
- Landing page com login Google (`angular-oauth2-oidc`, fluxo implicit/OIDC contra `accounts.google.com`)
- Guard de rota (`auth.guard.ts`) protegendo áreas autenticadas
- CRUD de **Categorias** (`categorias/`) e **Lugares** (`lugares/`), cada lugar associado a uma categoria, com foto, localização e avaliação
- Galeria de lugares (`galeria/`)
- Layout/template compartilhado (`template/layout/`)
- Estilização com **Tailwind CSS**
- Dockerizado: build multi-stage (Angular → Nginx) para o front, e um container dedicado para a API fake

**Stack:** Angular 19, `angular-oauth2-oidc`, Tailwind CSS 3, `json-server` (API fake), Docker + Nginx.

**Rodando localmente:**
```bash
cd passeio-app
npm install

# em um terminal: API fake
npm run server              # json-server ./api/db.json — http://localhost:3000

# em outro terminal: front
npm start                   # ng serve — http://localhost:4200
```

**Rodando com Docker:**
```bash
# front (build Angular + serve via Nginx)
docker build --tag passeio-app .
docker run -p 8080:80 passeio-app

# API fake (dados de produção, api/db.production.json)
cd api
docker build --tag passeio-app-api .
docker run -p 4000:4000 passeio-app-api
```

> O `.dockerignore` do projeto exclui `node_modules`, `dist` e `.angular` do contexto de build — importante para não copiar binários nativos (ex.: `esbuild`) compilados para o seu SO por cima dos que o `npm install` instala dentro do container Linux.

**Observação sobre duplicidade de arquivos:** algumas pastas de `src/app/` (`categorias/`, `template/`) têm arquivos com nomes quase idênticos (ex.: `categorias-module.ts` **e** `categorias.module.ts`, `template-routing-module.ts` **e** `template-routing.module.ts`) — resultado de conteúdo do curso colado por cima de scaffolds gerados pela CLI. Vale revisar e remover os duplicados não utilizados antes de considerar o projeto "limpo".

---

## Contexto de aprendizado

Este repositório documenta a evolução de um estudo prático de Angular, migrando de uma base mais forte em back-end e React para o ecossistema Angular — incluindo `NgModule` vs. componentes standalone, ciclo de vida (`ngOnInit`), `HttpClient`, Observables/RxJS e injeção de dependência. Os três projetos representam estágios progressivos: fundamentos isolados → CRUD com biblioteca de UI → aplicação com autenticação e API real.
