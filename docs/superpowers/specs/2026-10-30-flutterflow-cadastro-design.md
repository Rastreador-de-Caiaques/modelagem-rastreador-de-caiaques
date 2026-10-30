# Cadastro no FlutterFlow + monitoramento ligado ao Firebase — Design

Data: 2026-10-30

## Intenção
A Xtreme Caiaques precisa de um app fácil de usar no celular para cadastrar clientes e caiaques e registrar "o cliente X pegou o caiaque Y". O monitoramento em tempo real (mapa) continua como aplicação web, mas passa a mostrar nomes reais e o cliente do passeio em andamento, em vez de "Caiaque N".

## Objetivos
- App de cadastro no FlutterFlow (projeto existente `kayak-manager-zibit4`), dados no Firestore.
- Fluxo de passeio: escolher cliente + caiaque disponível, iniciar; encerrar devolve o caiaque.
- Monitoramento (Angular) lê nomes/cliente do Firestore (somente leitura), posição segue pelo WebSocket do backend.
- Monitoramento instalável no celular como PWA (ícone, tela cheia, offline básico do shell).
- Revisão geral do projeto com lista de melhorias de interface e de código.

## Não objetivos
- Não mover o cadastro para o Postgres nem para o backend NestJS.
- Não sincronizar Firestore com Postgres (Cloud Function descartada).
- Não publicar nas lojas (Play Store/App Store): fica a cargo do usuário.
- Sem pagamento, contrato ou histórico financeiro nesta fase.

## Anti-objetivos
- Dois lugares onde editar o mesmo cadastro.
- Regras de segurança abertas que deixem qualquer pessoa gravar no Firestore.
- Mapa que quebra quando o Firebase estiver fora do ar (deve cair para "Caiaque N").

## Restrições
- Backend publicado no Render; o caminho `/api/v1/kayaks` hoje responde 404, então não depender dele.
- WebSocket envia apenas `id` (número do dispositivo), `lat`, `lng`, `bat`, `vel`.
- Frontend Angular 20 no Vercel; backend, frontend e firmware em repositórios separados.
- O Firestore atual tem só a coleção `caiaques` (nome, serial, estado, cor); a lista do app mostra dados de exemplo.

## Abordagens consideradas
1. **Angular lê o Firestore direto, somente leitura (escolhida).** Simples, não toca no backend. Custo: o Angular passa a depender do Firebase e de boas regras de segurança.
2. Cloud Function copiando para o Postgres. Monitoramento independente do Firebase, mas muito mais trabalho e mais pontos de falha.
3. Sem ligação (mapa com "Caiaque N"). Sem trabalho, mas elimina o benefício do cadastro no monitoramento.

Escolha do usuário: opção 1.

## Modelo de dados (Firestore)
- `clientes`: nome, telefone, documento (opcional), criadoEm.
- `caiaques`: nome, serial, estado (disponivel | em_uso | manutencao), cor, `deviceId` (número, igual ao `id` do rádio, único).
- `passeios`: clienteRef, caiaqueRef, inicio, fim (nulo enquanto em andamento), estado (em_andamento | encerrado).

## Comportamento
- Iniciar passeio: cria `passeios` em andamento e põe o caiaque em `em_uso`. Só caiaques `disponivel` aparecem na escolha.
- Encerrar passeio: preenche `fim`, estado `encerrado`, caiaque volta a `disponivel`.
- Monitoramento: para cada `id` recebido, busca o caiaque com esse `deviceId`; se houver passeio em andamento, mostra "Nome — Cliente". Sem cadastro, mostra "Caiaque N".

## Decisões minhas (a confirmar)
- Login no app FlutterFlow com Firebase Auth (e-mail/senha) para operadores; Angular lê com leitura permitida só para usuários autenticados, ou, se você preferir mapa sem login, leitura pública limitada a nome do caiaque e nome do cliente.
- Início/fim do passeio gravados duas vezes (passeio e estado do caiaque) na mesma operação, para não ficar inconsistente.
- Estados e telas em português.

## Fica para o construtor
- Telas do FlutterFlow: lista/cadastro/edição de clientes, adaptação das telas de caiaques para o novo modelo, tela de passeios (iniciar/encerrar), ajustes visuais.
- Regras de segurança do Firestore e índice de `deviceId`.
- Módulo Angular de leitura do Firestore e PWA (manifesto, service worker, ícones).
- Lista de melhorias da revisão geral (interface, segurança do backend sem login, dados fixos nas telas).
