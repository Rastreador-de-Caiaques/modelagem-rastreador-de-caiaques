# Cadastro FlutterFlow + Mapa com Firestore + PWA Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** App de cadastro (clientes, caiaques, passeios) no FlutterFlow sobre Firestore, mapa Angular mostrando nome do caiaque e cliente do passeio, e mapa instalável como PWA.

**Architecture:** FlutterFlow é o único lugar de escrita do cadastro (Firestore). O Angular só lê o Firestore (autenticado) e junta com as posições do WebSocket pelo `deviceId`. Backend NestJS/Postgres não muda.

**Tech Stack:** FlutterFlow + Firebase (Auth, Firestore), Angular 20 + `@angular/fire`, `@angular/service-worker`, Jasmine/Karma.

**Spec:** `docs/superpowers/specs/2026-10-30-flutterflow-cadastro-design.md`

## Global Constraints

- O monorepo `rastreadorcaiaque` está arquivado: todo código novo e todo PR vai para os repositórios da organização `Rastreador-de-Caiaques`: `frontend-rastreador-de-caiaques` (Angular/PWA) e `modelagem-rastreador-de-caiaques` (docs e relatório de revisão).
- Um branch e um PR por task, a partir da `main` do repositório, mensagens de commit em português, sem push direto na `main`.
- Firestore: coleções `clientes`, `caiaques`, `passeios`; estados `disponivel | em_uso | manutencao` (caiaque) e `em_andamento | encerrado` (passeio); `caiaques.deviceId` numérico e único.
- Login por Firebase Auth (e-mail/senha), no app e no mapa.
- Mapa nunca quebra se o Firestore falhar: cai para "Caiaque N".
- Telas e textos em português. Sem alterar o backend nem o Postgres. Sem publicar nas lojas.
- `gh` não está instalado: PRs são abertos pelo navegador ou após o usuário autorizar instalar o `gh`.

## Review Focus

- Dois caiaques cadastrados com o mesmo `deviceId`: o cadastro deve recusar o segundo.
- Rádio envia `id` sem caiaque cadastrado: o mapa mostra "Caiaque N", sem erro.
- Iniciar passeio com caiaque que acabou de ficar `em_uso` por outro operador: deve falhar sem criar passeio duplicado.
- Firestore indisponível com usuário logado: o mapa mantém posições com nomes genéricos "Caiaque N".
- Cliente sem telefone ou nome vazio: o cadastro deve recusar nome vazio.

---

### Task 1: Organização dos repositórios e documentos

**Files:**
- Create: `docs/superpowers/specs/2026-10-30-flutterflow-cadastro-design.md` e `docs/superpowers/plans/2026-10-30-flutterflow-cadastro.md` no repositório `modelagem-rastreador-de-caiaques`
- Produces: clones locais em `C:\Users\guest1\Desktop\CAIAQUES_RASTREADOR\repos\<nome>`

- [ ] **Step 1:** Clonar `frontend-rastreador-de-caiaques` e `modelagem-rastreador-de-caiaques` em `repos\`; conferir `git remote -v` apontando para a organização.
- [ ] **Step 2:** No clone de modelagem, criar `docs/flutterflow-cadastro` e copiar spec e plano para `docs/superpowers/`.
- [ ] **Step 3:** Commit `docs: design e plano do cadastro no FlutterFlow`, push e abrir PR (descrição: objetivo, links do spec e plano).
- [ ] **Step 4:** Verificar: PR aberto, `git status` limpo. Expected: PR visível na organização.

### Task 2: Firebase — Auth, regras e índice

**Files:**
- Create: `firestore.rules` e `docs/firestore-modelo.md` no repositório de modelagem (cópia versionada das regras e do modelo)
- Produces: usuários operadores no Firebase Auth; regras publicadas no console

**Interfaces:**
- Produces: leitura e escrita em `clientes`, `caiaques`, `passeios` apenas com `request.auth != null`.

- [ ] **Step 1:** No console Firebase do projeto do FlutterFlow, habilitar Auth por e-mail/senha (o usuário cria os operadores; não criar contas por ele).
- [ ] **Step 2:** Escrever `firestore.rules` com `allow read, write: if request.auth != null` nas três coleções e negação para o resto; publicar.
- [ ] **Step 3:** Criar o índice de consulta `caiaques` por `deviceId` e `passeios` por `estado`.
- [ ] **Step 4:** Verificar no simulador de regras do console: leitura sem login negada, com login permitida.
- [ ] **Step 5:** Commit e PR em modelagem (`docs: regras do Firestore e modelo de dados`).

### Task 3: FlutterFlow — login e clientes

**Interfaces:**
- Produces: páginas `Login`, `ClientesLista`, `ClienteForm` (criar/editar); coleção `clientes` com `nome` (obrigatório), `telefone`, `documento`, `criadoEm`.

- [ ] **Step 1:** Criar coleção `clientes` em Firestore (schema acima) no FlutterFlow.
- [ ] **Step 2:** Criar página `Login` (Firebase Auth) e definir como inicial após `SplashScreen`; deslogado vai para `Login`.
- [ ] **Step 3:** Criar `ClientesLista` (busca por nome) e `ClienteForm` com nome obrigatório e botão de excluir com confirmação.
- [ ] **Step 4:** Rodar em Test Mode: criar, editar, excluir cliente; nome vazio é recusado. Expected: operações refletem no Firestore.

### Task 4: FlutterFlow — caiaques com `deviceId`

**Interfaces:**
- Consumes: coleção `caiaques` existente (nome, serial, estado, cor).
- Produces: campo `deviceId` (Integer) e estados `disponivel | em_uso | manutencao`; telas `KayakInventory`, `AddKayak`, `EditKayak` ligadas ao Firestore real.

- [ ] **Step 1:** Adicionar `deviceId` à coleção; migrar os valores de `estado` existentes para os três estados novos.
- [ ] **Step 2:** Trocar os dados fixos de `KayakInventory` (itens repetidos, "Total Units", "In Service") por consulta ao Firestore com contagens reais.
- [ ] **Step 3:** Em `AddKayak`/`EditKayak`, validar `deviceId` único antes de salvar (consulta por `deviceId`; recusar se já existe em outro documento).
- [ ] **Step 4:** Test Mode: cadastrar dois caiaques com o mesmo `deviceId`; o segundo é recusado com mensagem. Expected: apenas um documento.

### Task 5: FlutterFlow — passeios

**Interfaces:**
- Consumes: `clientes`, `caiaques` (Task 3, 4).
- Produces: coleção `passeios` (`clienteRef`, `caiaqueRef`, `inicio`, `fim`, `estado`); páginas `PasseioNovo`, `PasseiosAtivos`.

- [ ] **Step 1:** Criar coleção `passeios`.
- [ ] **Step 2:** `PasseioNovo`: escolher cliente e caiaque (apenas `estado == disponivel`). Ação personalizada que, numa transação do Firestore, reconfere o estado do caiaque, cria o passeio `em_andamento` e põe o caiaque `em_uso`; se não estiver mais `disponivel`, aborta com aviso.
- [ ] **Step 3:** `PasseiosAtivos`: lista de passeios `em_andamento` com botão "Encerrar", que numa transação preenche `fim`, estado `encerrado` e volta o caiaque para `disponivel`.
- [ ] **Step 4:** Test Mode: iniciar e encerrar um passeio; estados do caiaque e do passeio corretos. Tentar iniciar passeio com caiaque `em_uso`: bloqueado.

### Task 6: Angular — nomes do Firestore no mapa

**Files:**
- Modify: `src/app/services/caiaques.service.ts`, `src/app/app.config.ts`, `src/environments/environment*.ts`
- Create: `src/app/services/cadastro.service.ts`, `src/app/services/cadastro.service.spec.ts`, `src/app/login/login.ts` (+ html/scss)
- Produces: `CadastroService.rotulo$(id: number): Observable<string>`; ex.: `"Canoa Azul — João"`, ou `"Caiaque 7"` sem cadastro ou em erro.

**Interfaces:**
- Consumes: coleções `caiaques` (por `deviceId`) e `passeios` (`estado == em_andamento`) do Firestore.

- [ ] **Step 1:** Teste `cadastro.service.spec.ts` com Firestore simulado: `rotulo$(7)` com caiaque e passeio ativo emite `"Canoa Azul — João"`; sem passeio emite `"Canoa Azul"`; sem caiaque emite `"Caiaque 7"`; erro do Firestore emite `"Caiaque 7"`.
- [ ] **Step 2:** Rodar `ng test --watch=false`. Expected: falha, serviço inexistente.
- [ ] **Step 3:** Instalar `@angular/fire` e `firebase`; implementar `CadastroService` com a assinatura acima, apenas leitura, com `catchError` caindo para `"Caiaque N"`.
- [ ] **Step 4:** Rodar os testes. Expected: PASS.
- [ ] **Step 5:** Tela `Login` (e-mail/senha, Firebase Auth) e guarda de rota; o mapa inteiro exige login: deslogado, a guarda de rota leva para `Login`. Usar `CadastroService.rotulo$` nos rótulos de `mapa`, `painel-caiaque` e `painel-status`.
- [ ] **Step 6:** Configuração do Firebase só em `environment*.ts` com a config pública do app web; commit, push e PR no repositório do frontend.

### Task 7: Angular — PWA instalável

**Files:**
- Create: `public/manifest.webmanifest`, ícones 192 e 512 em `public/icons/`, `ngsw-config.json`
- Modify: `src/index.html`, `angular.json`, `src/app/app.config.ts`

- [ ] **Step 1:** Adicionar `@angular/service-worker` (`ng add @angular/pwa`), nome "Rastreador de Caiaques", `display: standalone`, `lang: pt-BR`, tema coerente com o app.
- [ ] **Step 2:** Registrar o service worker só em produção, sem cachear o WebSocket nem as chamadas ao Firestore.
- [ ] **Step 3:** `ng build` e servir `dist` com `npx http-server`; no Lighthouse, a categoria PWA/instalável passa. Expected: botão "Instalar" no Chrome.
- [ ] **Step 4:** Commit, push e PR; após o deploy no Vercel, testar a instalação no celular.

### Task 8: Revisão geral e sugestões

**Files:**
- Create: `docs/revisao-geral.md` no repositório de modelagem

- [ ] **Step 1:** Revisar backend, frontend, firmware e FlutterFlow; listar achados por severidade (por exemplo: backend sem login nem hash de senha, CORS aberto `*`, `/api/v1/kayaks` retorna 404 no Render, nomes fixos no mapa, `CONTEXT.md` desatualizado sobre Next.js, rota acumulada sem limite no serviço Angular).
- [ ] **Step 2:** Para cada achado, dar sugestão e esforço estimado; separar "corrigir já" de "ideia de interface".
- [ ] **Step 3:** Commit e PR; o usuário decide o que vira tarefa.
