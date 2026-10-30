# Revisão geral (2026-10-30)

Escopo e limites: li o backend NestJS (`main.ts`, `app.module.ts`, `schema.prisma`, controller de tracking, `CONTEXT.md`), o frontend Angular (serviço de caiaques, painel de status, ambientes) e o projeto FlutterFlow/Firebase. **Não revisei** o firmware, os diagramas, nem rodei o backend. Achados marcados "a confirmar" dependem de testar.

## Corrigir primeiro

| # | Achado | Evidência | Sugestão |
|---|---|---|---|
| 1 | Regras do Firestore estavam abertas (qualquer pessoa lia/gravava `caiaques` até 15/11/2026) | Console do Firebase, aba Regras | **Já corrigido**: login obrigatório (PR de regras). |
| 2 | Backend sem autenticação: qualquer um chama `POST /tracking` e a API | `main.ts` não tem guard; `CONTEXT.md` lista JWT como pendente | Chave simples (header) no `POST /tracking` da base ESP; JWT para o resto. |
| 3 | CORS aberto para qualquer origem | `main.ts`: `origin: '*'` | Restringir ao domínio do Vercel. |
| 4 | Senha de usuário sem hash | `CONTEXT.md`: "TODO: hash com bcrypt" | bcrypt antes de usar `User` de verdade. |
| 5 | `/api/v1/kayaks` responde 404 no Render | `GET https://rastreador-de-caiaques.onrender.com/api/v1/kayaks` → 404 | Conferir quais rotas existem de fato (a documentação cita `/api/v1`, o código usa prefixo `api`). |
| 6 | Código duplicado: pastas `presentation/`, `application/`, `domain/` (Clean Architecture) e módulos `kayak/`, `tracking/`, `user/` simples | `app.module.ts` importa só os módulos simples (a confirmar se `presentation/` é usado em algum lugar) | Escolher um e remover o outro. |
| 7 | Projeto Firebase "TESTE" no plano Blaze (cobrança por uso) | Console do Firebase | Criar alerta de orçamento. Nome sugere teste, mas é usado pelo FlutterFlow. |

## Melhorias de código (menor urgência)

- **Histórico sem limite no frontend:** `caiaques.service.ts` acrescenta pontos em `historico`/`rota` sem teto e salva no `localStorage`. Em passeios longos ou muitos caiaques isso cresce; limitar (ex.: últimos N pontos).
- **Documentação desatualizada:** `CONTEXT.md` fala em frontend Next.js, mas é Angular; `environment.prod.ts` comenta "Railway" e usa Render; `documentacao/arquitetura.md` do monorepo estava vazio.
- **Testes:** `app.spec.ts` estava quebrado (corrigido no PR dos nomes); `CONTEXT.md` cita testes unitários do backend com erro em `KayakStatus` (não rodei).
- **Render gratuito dorme:** o firmware já aumentou o timeout para 30 s por causa do cold start; considerar um ping periódico ou plano pago para a entrega ao cliente.

## Sugestões de interface (ideias, não verificadas com usuários)

- **App no celular:** PWA instalável (PR aberto). Testar no Android após o deploy.
- **Nomes reais no mapa** (PR aberto, opcional): "Canoa Azul — João" em vez de "Caiaque 7".
- **Alertas visuais:** caiaque sem sinal há X minutos e bateria baixa em destaque no painel de status (o WebSocket já traz `bat` e `vel`; não verifiquei se são exibidos).
- **Tempo de passeio:** mostrar tempo decorrido ao lado do nome; o painel já calcula "tempo no mar".
- **Cores/estados** do caiaque no mapa alinhados aos estados do cadastro (`disponivel`, `em_uso`, `manutencao`).
- **FlutterFlow:** a lista de caiaques ainda mostra dados de exemplo e as telas de passeio não foram criadas.

## Sobre o FlutterFlow
- O plano é Free: o painel avisa que o FlutterFlow Agent não está incluído; código personalizado pode exigir plano pago (não testado).
- Sem transação no Firestore pelo plano atual, "iniciar passeio" faz duas gravações (passeio e estado do caiaque). Dois operadores no mesmo caiaque ao mesmo tempo podem colidir (a confirmar).
