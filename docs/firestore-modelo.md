# Modelo de dados do Firestore (projeto Firebase `teste-f118a`)

Cadastro feito no app FlutterFlow "Kayak Manager"; o mapa Angular só lê.
Regras em [`firestore.rules`](../firestore.rules): só usuários autenticados (e-mail/senha).

## clientes
| campo | tipo | observação |
|---|---|---|
| nome | string | obrigatório |
| telefone | string | |
| documento | string | opcional |
| criadoEm | timestamp | |

## caiaques
| campo | tipo | observação |
|---|---|---|
| nome | string | |
| serial | string | |
| estado | string | `disponivel`, `em_uso` ou `manutencao` |
| cor | color | |
| deviceId | integer | igual ao `id` enviado pelo rádio; único |

## passeios
| campo | tipo | observação |
|---|---|---|
| clienteRef | reference (clientes) | |
| caiaqueRef | reference (caiaques) | |
| inicio | timestamp | |
| fim | timestamp | nulo enquanto em andamento |
| estado | string | `em_andamento` ou `encerrado` |

## Índices
Nenhum índice composto é necessário: as consultas (`deviceId ==`, `estado ==`) usam os índices automáticos de campo único.

## Notas
- O projeto também tem o app Android `br.caiaques.caiaques`, que usava `caiaques` com regras abertas até 15/11/2026; com as regras novas ele precisa estar autenticado.
- Operadores são criados no console do Firebase (Authentication > Usuários).
