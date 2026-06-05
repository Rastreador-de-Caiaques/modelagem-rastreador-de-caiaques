# Arquitetura de Software

Esta documentação descreve a organização arquitetural do software do **Sistema de Rastreamento de Caiaques**.

O sistema é dividido em dois blocos principais de software: **Backend** (API NestJS) e **Frontend** (Angular Single Page Application).

---

## 1. Backend (API NestJS)

O backend é construído sobre o ecossistema **Node.js** (versão 18+) utilizando o framework **NestJS 10** e a linguagem **TypeScript**. A arquitetura segue os princípios da **Clean Architecture** (Arquitetura Limpa), dividida em 4 camadas bem delimitadas:

```
[ Presentation Layer (Controllers / WebSockets) ]
                       ↓
   [ Application Layer (Use Cases / DTOs) ]
                       ↓
      [ Domain Layer (Entities / Interfaces) ]
                       ↑
[ Infrastructure Layer (Prisma Repositories / DB) ]
```

### Detalhamento das Camadas:

#### A. Camada de Domínio (Domain Layer)
*   **Papel:** Centraliza as regras de negócio puras, entidades estruturais e contratos de interfaces de repositórios. Ela não possui qualquer dependência externa (não conhece frameworks, ORMs ou bibliotecas de terceiros).
*   **Arquivos Principais:**
    *   `src/domain/entities/kayak.entity.ts`: Entidade contendo o modelo de dados de um Caiaque e o enum `KayakStatus` (`AVAILABLE`, `IN_USE`, `MAINTENANCE`, `ALERT`).
    *   `src/domain/entities/tracking-point.entity.ts`: Entidade contendo a estrutura de coordenadas (`latitude`, `longitude`, `recordedAt`, etc.).
    *   `src/domain/repositories/kayak.repository.ts`: Interface (`IKayakRepository`) contendo assinaturas de busca e persistência de caiaques.

#### B. Camada de Aplicação (Application Layer)
*   **Papel:** Orquestra os casos de uso do sistema, implementando a lógica dos fluxos operacionais específicos e manipulando DTOs (Data Transfer Objects).
*   **Casos de Uso Principais:**
    *   `RegisterKayakUseCase`: Registro de novas embarcações.
    *   `GetTrackingUseCase`: Consulta de rotas e coordenadas históricas.

#### C. Camada de Infraestrutura (Infrastructure Layer)
*   **Papel:** Responsável pelo acesso a recursos externos como o banco de dados. Implementa as interfaces definidas na Camada de Domínio usando o **Prisma ORM** para ler e salvar registros no banco relacional PostgreSQL.
*   **Componentes:**
    *   `UserRepositoryImpl` / `KayakRepositoryImpl`: Classes concretas injetadas pelo NestJS para persistência real.
    *   `schema.prisma`: Definição declarativa das tabelas e relacionamentos do banco relacional.

#### D. Camada de Apresentação (Presentation Layer)
*   **Papel:** Expõe os adaptadores para o mundo exterior. Recebe requisições da rede e as direciona aos casos de uso.
*   **Componentes:**
    *   `TrackingController`: Expõe endpoints HTTP REST (`POST /api/tracking` para o gateway e `GET /api/tracking` para o frontend).
    *   `TrackingGateway`: Servidor WebSocket (`/ws`) utilizando o adaptador `ws` do NestJS para transmitir eventos em tempo real.

---

## 2. Frontend (Angular SPA)

O frontend é desenvolvido em **Angular 19** com TypeScript e estilização em SCSS, hospedado na rede de distribuição estática da Vercel.

### Componentes de UI:
*   **`MapaComponent`:** Incorpora o mapa interativo através da biblioteca **Leaflet**. Desenha marcadores personalizados para cada caiaque ativo no mar e desenha caminhos dinâmicos (`Polyline`) representando o rastro da rota percorrida no dia.
*   **`PainelCaiaque`:** Exibe dados numéricos da embarcação selecionada (ID, velocidade média, última atualização, tempo de sessão no mar).
*   **`PainelStatus`:** Painel indicador que mostra o status do backend e do heartbeat da estação base em terra.

### Serviços e Lógica de Fluxo:
*   **`CaiaqueService`:** Centraliza a inteligência de conexão. Utiliza `webSocket` do RxJS para escutar atualizações assíncronas do backend.
*   **Filtro de Ruído (Fórmula de Haversine):** O frontend calcula a distância matemática em metros entre o novo ponto recebido e o ponto anterior do histórico. Caso o deslocamento seja menor do que **5 metros**, o ponto é ignorado para filtrar ruídos estáticos de GPS estacionário.
*   **Persistência Local (Session Cache):** Salva o estado acumulado das sessões no `localStorage` do navegador, mantendo o histórico de rota renderizado na tela do operador mesmo se a aba do navegador for atualizada.
