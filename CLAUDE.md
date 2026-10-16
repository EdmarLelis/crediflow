# CLAUDE.md — CrediFlow (plataforma de operações de crédito, projeto de portfólio)

## Contexto

Sou desenvolvedor full stack júnior com experiência profissional em TypeScript, Node,
NestJS, React e PostgreSQL. Tenho bastante estudo em Python e noção de Django (MVT), mas
nunca usei FastAPI, SQLAlchemy nem Svelte.

Este projeto tem **dois objetivos simultâneos**:

1. **Portfólio para uma vaga específica** — dev júnior numa gestora de crédito que pede
   Python/FastAPI, PostgreSQL, Svelte, AWS e desenvolvimento assistido por agentes. Precisa
   ficar no ar, com README bom e código que eu consiga defender linha a linha em entrevista.
2. **Aprender FastAPI, SQLAlchemy e Svelte de verdade.** O aprendizado importa tanto quanto
   o resultado.

Por causa do objetivo 2, **a forma como você me ajuda importa mais que a velocidade**.
A seção "Como trabalhar comigo" é a parte mais importante deste arquivo.

## Como trabalhar comigo (regra principal)

**Não escreva código de feature por mim.** Se você implementar, eu não aprendo.

Ciclo de cada etapa:

1. **Conceito** — explique o que vamos fazer e _por quê_, com um exemplo curto e isolado
   (não o código do meu projeto). Máximo ~20 linhas. Sempre que fizer sentido, compare
   com o que eu já conheço: Django (ORM, serializers, views) para o backend, NestJS para
   injeção de dependência e camadas, React para o Svelte.
2. **Estudo** — me diga o que pesquisar para aprofundar: termos de busca exatos, seção da
   documentação oficial.
    **Nunca invente URLs.** Sem certeza de que o link existe, dê o termo de busca.
3. **Tarefa** — o que implementar, com critério de aceite claro ("`POST /operacoes`
   retorna 422 quando o valor estoura o limite disponível da empresa").
4. **Eu escrevo o código.**
5. **Revisão** — aponte _o problema_ primeiro e me deixe tentar corrigir; só mostre a
   solução se eu pedir ou errar duas vezes.
6. **Commit** — sugira a mensagem e seguimos.

### Pode escrever direto, sem me fazer digitar

Configuração e boilerplate (`pyproject.toml`, config do Ruff, `docker-compose.yml`,
`Dockerfile`, `alembic.ini` e o `env.py` do Alembic, `svelte.config.js`, config do
Tailwind, GitHub Actions), comandos de terminal, script de seed com dados fake, e
correção de erro de ambiente sem valor didático.

### NÃO escreva sem eu pedir

Models SQLAlchemy, schemas Pydantic, routers, services, repositories, regras de negócio,
SQL da view materializada, componentes e rotas Svelte, testes qualquer coisa com regra
de negócio ou que eu precise explicar em entrevista.

### Estilo

Explique o **porquê**, não só o como. Havendo mais de uma abordagem, mostre o trade-off e
recomende uma. Me questione quando eu tomar uma decisão que vai doer depois. Se eu
escrever algo que funciona mas é ruim, fale. Não valide por funcionar.

## Versões (não sugira sintaxe antiga)

A maior parte dos tutoriais online usa versões antigas destas bibliotecas.

- **SQLAlchemy 2.0** (async): use `Mapped[...]`, `mapped_column()`, `select()` +
  `session.execute()`/`scalars()`, `AsyncSession` com `asyncpg`. Nada de
  `session.query()` nem `Column()` no estilo 1.x.
- **Pydantic v2**: `model_config = ConfigDict(...)`, `field_validator`,
  `model_validate`, `model_dump`. Nada de `class Config`, `@validator` ou `.dict()`.
- **Svelte 5** (runes): `$state`, `$derived`, `$effect`, `$props`. Nada de `export let`
  nem `$:` reativo do Svelte 4.

Na dúvida sobre sintaxe de alguma dessas versões, consulte a documentação oficial em vez de
responder de memória, e me avise que está incerto.

## Ambiente

Windows 11 · Python 3.12 · Docker Desktop para o Postgres local (precisa estar aberto antes
do `docker compose up -d`) · Node 24 + pnpm 11 (via corepack) para o front.

**Smart App Control do Windows está ligado** e já bloqueou executável nativo antes (pnpm 12).
Por isso o ambiente Python é `venv` + `pip`. Se for sugerir `uv` ou outra ferramenta com
binário nativo, avise que pode ser bloqueada. Se aparecer "Uma política de Controle de
Aplicativo bloqueou este arquivo", a causa é essa.

## Fluxo Git

**GitHub Flow.** `main` sempre pronto para produção, protegido por ruleset (PR obrigatório,
histórico linear, merge só por squash, branch apagada após merge).

- Branch curta por tarefa a partir do `main` atualizado: `feat/...`, `fix/...`, `chore/...`
- **Título do PR em Conventional Commits** (vira a mensagem do commit no `main`)
- Depois do merge: `git switch main && git pull` e `git branch -D <branch>`
- Migration nova no PR → aplicar no banco de produção junto do merge
- CI no PR: Ruff + pytest (back), lint + build (front)

## Estrutura

```
crediflow/
├── api/   → FastAPI (app/routers, app/services, app/repositories, app/models,
│            app/schemas, app/core), alembic/, tests/
├── web/   → SvelteKit + TypeScript
└── docker-compose.yml
```

## Stack (fechada: não sugerir substituições sem eu perguntar)

**Back:** Python 3.12 · FastAPI · SQLAlchemy 2.0 async + asyncpg · Alembic · Pydantic v2 ·
JWT (PyJWT) + hash de senha com bcrypt/passlib · Ruff

**Front:** SvelteKit · TypeScript · Tailwind · Chart.js

**Banco:** PostgreSQL 16 (Docker no dev, RDS em produção)

**Testes:** pytest + pytest-asyncio + httpx (AsyncClient), banco de teste separado

**Deploy:** AWS API em EC2 t3.micro com Docker Compose, banco no RDS PostgreSQL free tier,
front estático no S3 + CloudFront. **Alerta de orçamento (AWS Budgets) configurado antes de
criar qualquer recurso.**

## Domínio e regras de negócio

- **Empresa (cedente):** CNPJ (único, validado), razão social, setor, rating interno A–E,
  limite de crédito.
- **Operação:** empresa, valor, taxa ao mês, prazo em meses, data de início, status
  (`em_analise`, `aprovada`, `ativa`, `liquidada`, `inadimplente`).
- **Parcela:** número, vencimento, valor, status (`aberta`, `paga`, `atrasada`).
- **Usuário:** perfis `analista` e `gestor`.

Regras:

- Não aprovar operação que estoure o limite disponível (limite − saldo devedor das
  operações ativas da empresa).
- Só `gestor` aprova. Ao aprovar, gerar o cronograma pela **tabela Price**, numa transação.
- Pagamento de parcela; operação vira `liquidada` quando todas estiverem pagas.
- Rotina que marca parcelas vencidas como `atrasadas` e a operação como `inadimplente`
  após 90 dias de atraso.
- Valores monetários em `Numeric`/`Decimal`, **nunca float**.

## Decisões já tomadas (não questionar, só implementar)

- **Arquitetura em camadas:** router → service → repository, com `Depends` do FastAPI para
  injeção. Regra de negócio só no service.
- **Dinheiro em `Decimal`** do schema ao banco.
- **View materializada `carteira_resumo`** (saldo devedor por empresa e por rating,
  % de inadimplência) alimentando o dashboard, com refresh após aprovação/pagamento.
  No README: benchmark com e sem a view sobre um seed de ~50k parcelas.
- **Sem Redis, sem Celery.** A rotina de inadimplência é um endpoint protegido (só gestor),
  disparado manualmente ou por cron externo.

## Roadmap

Trabalhamos **uma etapa por vez**. Não avance sozinho.

- [ ] **1. Setup** — venv, estrutura de pastas, Docker Compose com Postgres, FastAPI com
      `GET /health` (200 com banco ok, 503 com banco fora), Alembic configurado
- [ ] **2. Models + migrations + seed** — Empresa, Operacao, Parcela, Usuario
- [ ] **3. Auth** — registro, login, JWT, dependency de usuário atual e de perfil
- [ ] **4. CRUD de empresas e operações** — paginação, filtros, erros padronizados
- [ ] **5. Regras de negócio** — aprovação com limite, tabela Price, pagamento,
      inadimplência
- [ ] **6. Testes** — foco em tabela Price, limite e aprovação; CRUD puro não precisa
- [ ] **7. View materializada + dashboard + benchmark**
- [ ] **8. Front SvelteKit** — login, dashboard, lista de operações, detalhe com cronograma
- [ ] **9. Deploy AWS + CI + README**

**Estratégia:** subir um esqueleto no ar logo após a etapa 1 (`/health` no EC2 + RDS), não
na etapa 9.

## README (seção obrigatória)

"**Como este projeto foi desenvolvido**": explicar que usei um agente como mentor, não como
gerador de código, e como funcionou o ciclo conceito → estudo → tarefa → eu implemento →
revisão. Citar exemplos reais de revisão que me fizeram mudar algo. Este `CLAUDE.md` fica
versionado no repo como evidência.

## Status atual

Projeto ainda não iniciado. **Próxima etapa: 1. Setup.**

## Fora de escopo

Upload de documentos, score automático de crédito, notificações, multiempresa/multi-tenant,
relatórios em PDF, refresh token, Terraform, tempo real.

Se eu sugerir qualquer um deles antes da etapa 9 fechar, **me lembre de que combinei de não
fazer**. Projeto de portfólio não terminado não vale nada.
