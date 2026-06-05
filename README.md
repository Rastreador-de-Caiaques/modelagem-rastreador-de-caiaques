# Modelagem UML — Sistema de Rastreamento de Caiaques (Xtreme Caiaques)

Este repositório contém toda a modelagem de arquitetura e infraestrutura para o **Sistema de Rastreamento de Caiaques em Tempo Real**, desenvolvido para a empresa **Xtreme Caiaques** (Santos-SP).

A modelagem é dividida em diagramas de software (Casos de Uso e Componentes) desenvolvidos no **Modelio**, e diagrama de infraestrutura física/lógica (Implantação) desenvolvido no **Draw.io**.

---

## 📂 Estrutura do Repositório

O repositório está organizado da seguinte forma:

*   **[`modelagem.zip`](modelagem.zip):** Arquivo compactado contendo o projeto/workspace do **Modelio** com os diagramas de Caso de Uso e de Componentes.
*   **[`diagrama_deployment.drawio`](diagrama_deployment.drawio):** Arquivo fonte XML editável do **Draw.io** contendo o Diagrama de Implantação 3D (UML 2.0).
*   **[`Diagrama de Caso de uso.png`](Diagrama%20de%20Caso%20de%20uso.png):** Exportação em imagem do Diagrama de Casos de Uso.
*   **[`Diagrama de Componentes.png`](Diagrama%20de%20Componentes.png):** Exportação em imagem do Diagrama de Componentes.
*   **[`Diagrama de Implantacao.png`](Diagrama%20de%20Implantacao.png):** Exportação em imagem do Diagrama de Implantação (Deployment).
*   **[`arquitetura.md`](arquitetura.md):** Detalhamento técnico da arquitetura de software (NestJS Clean Architecture + Angular SPA).
*   **[`hardware.md`](hardware.md):** Especificações físicas dos dispositivos embarcados (ESP32, GPS NEO-6M, LoRa E32).
*   **[`protocolo.md`](protocolo.md):** Definição dos pacotes de dados de telemetria (JSON), conexões de rádio LoRa e endpoints HTTP/WebSocket.

---

## 📊 Diagramas UML do Projeto

### 1. Diagrama de Casos de Uso (Use Case Diagram)

O diagrama descreve as interações entre os atores externos e os limites do sistema, mapeando as permissões e funcionalidades implementadas no escopo do projeto.

![Diagrama de Casos de Uso](Diagrama%20de%20Caso%20de%20uso.png)

#### Atores Modelados:
*   **Operador (Operator):** Ator humano que monitora a frota a partir da estação em terra (dashboard web).
*   **Administrador (Admin):** Ator com privilégios completos, herdando os casos de uso do Operador e possuindo ações exclusivas de gerenciamento.
*   **Caiaque (Transmissor):** Ator de hardware móvel que coleta e transmite dados via LoRa.
*   **Estação Base (Receptor Gateway):** Ator de hardware em terra que recebe os sinais LoRa e encaminha para a nuvem.

#### Casos de Uso Principais:
*   **Visualizar Mapa em Tempo Real:** Acompanhamento dinâmico das posições geográficas.
*   **Consultar Detalhes do Caiaque:** Leitura de coordenadas, tempo de uso e histórico de deslocamento.
*   **Ingerir Ponto de Telemetria:** Recebimento automático dos dados no backend, incluindo os relacionamentos `<<include>>` para **Auto-registrar Caiaque** (caso seja o primeiro sinal de um novo ID) e **Atualizar Status** para em uso.
*   **Sinalizar Disponibilidade (Heartbeat):** Verificação periódica do status de conexão da Estação Base.

---

### 2. Diagrama de Componentes (Component Diagram)

Este diagrama representa a estrutura modular do sistema de software, evidenciando as interfaces de comunicação e a arquitetura em camadas do backend.

![Diagrama de Componentes](Diagrama%20de%20Componentes.png)

#### Subsistemas e Camadas:
1.  **Firmware (C++):** Dividido entre o componente transmissor (móvel) e o receptor gateway (fixo), utilizando as bibliotecas `TinyGPS++` e `LoRa_E32` via UART, e dependendo da stack `WiFi.h` e `HTTPClient` na estação base.
2.  **Backend (NestJS / Clean Architecture):**
    *   **Presentation Layer:** Controllers REST expostos ao gateway e ao frontend.
    *   **Real-time Gateway:** Servidor WebSocket (`/ws`) para broadcast imediato.
    *   **Application Layer:** Casos de uso desacoplados (lógica de aplicação).
    *   **Domain Layer:** Entidades puras e contratos de repositório (desacoplamento total).
    *   **Infrastructure Layer:** Repositórios implementados via Prisma ORM e persistência no PostgreSQL.
3.  **Frontend (Angular):**
    *   **CaiaqueService:** Serviço que consome as APIs HTTP e a conexão WebSocket.
    *   **MapaComponent:** Componente que consome o service e utiliza a biblioteca **Leaflet** para plotar a interface visual.
    *   **Componentes de UI:** Painéis de status da base e detalhes da embarcação.

---

### 3. Diagrama de Implantação (Deployment Diagram)

Mapeia a topologia de rede física e lógica, mostrando os artefatos de software distribuídos em seus respectivos ambientes de execução físicos e virtuais.

![Diagrama de Implantação](Diagrama%20de%20Implantacao.png)

#### Nós e Artefatos:
*   **Caiaque (Device):** ESP32 rodando o artefato `firmware_transmissor.bin`, conectado via UART ao GPS NEO-6M e ao rádio LoRa E32.
*   **Estação Base (Device):** ESP32 rodando `firmware_receptor_web.bin`, conectado via UART ao receptor LoRa e à internet via WiFi integrado.
*   **Servidor Render (Execution Environment):** Hospeda o ambiente de execução Node.js contendo o executável compilado do backend NestJS (`backend_app`).
*   **Banco de Dados PostgreSQL (Neon/Render):** Instância de banco relacional contendo o schema ativo e as tabelas `Kayak`, `TrackingPoint` e `User`.
*   **Servidor Vercel (Web Server/CDN):** Armazena o bundle estático do frontend (`frontend_bundle` contendo HTML, CSS e JS compilados).
*   **Dispositivo do Operador (Device / Browser):** Executa o aplicativo Angular SPA na memória do navegador do cliente após o download.

#### Protocolos de Comunicação:
*   **LoRa (915 MHz):** Comunicação em rádio frequência sem fio de longo alcance entre o Caiaque e a Estação Base.
*   **HTTPS:** Requisições seguras da Estação Base para ingestão de telemetria e pings de heartbeat no Render.
*   **WSS (WebSocket Secure):** Canal persistente bidirecional seguro entre o navegador do operador e o backend.
*   **TCP/IP (Porta 5432):** Conexão segura de banco de dados utilizada pelo Prisma ORM.

---

## 🛠️ Como Utilizar os Arquivos Fonte

### Importar Projeto no Modelio:
1.  Baixe o arquivo [`modelagem.zip`](modelagem.zip).
2.  Abra o **Modelio** (versão 3.8 ou superior recomendada).
3.  Vá em **File** $\rightarrow$ **Import Project...** e selecione o arquivo `modelagem.zip`.
4.  O Modelio irá descompactar e carregar o workspace com os diagramas de Casos de Uso e Componentes editáveis.

### Editar Diagrama no Draw.io:
1.  Acesse o [Draw.io](https://app.diagrams.net/).
2.  Vá em **File** $\rightarrow$ **Open From** $\rightarrow$ **Device...** (ou arraste o arquivo [`diagrama_deployment.drawio`](diagrama_deployment.drawio) para a tela).
3.  O diagrama de implantação 3D será carregado com suporte a todas as propriedades da biblioteca UML 2.0 integrada.
