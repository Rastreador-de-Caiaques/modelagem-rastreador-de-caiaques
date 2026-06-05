# Protocolo de Comunicação e Ingestão

Esta documentação descreve a especificação dos formatos de payload de telemetria, das conexões de rádio LoRa e das APIs REST/WebSocket que integram o ecossistema do **Rastreador de Caiaques**.

---

## 1. Comunicação via Rádio LoRa (Móvel $\rightarrow$ Base)

A transmissão sem fio utiliza modulação LoRa transparente ponto a ponto na banda ISM de **915 MHz**.

### Configuração do Rádio (SX1276 / E32):
*   **Frequência de Operação:** 915 MHz
*   **Potência de Transmissão:** 20 dBm (100 mW)
*   **Taxa de Transmissão Aérea (Air Data Rate):** 1.2 kbps (otimizada para longo alcance e alta robustez contra barreiras físicas)
*   **Modo de Transmissão:** Transparente (o rádio funciona como uma extensão serial sem fio)

---

## 2. Formato do JSON de Telemetria

O payload é serializado pelo transmissor no formato JSON puro.

### Caso A: Coordenadas Válidas (GPS Fix ativo)
Enviado quando o módulo móvel possui conexão com satélites de posicionamento.
```json
{
  "id": 1,
  "lat": -23.961234,
  "lng": -46.331234
}
```

*   `id` *(Integer)*: Identificador único do dispositivo físico (configurado no firmware).
*   `lat` *(Float)*: Latitude geográfica em graus decimais (precisão de 6 casas).
*   `lng` *(Float)*: Longitude geográfica em graus decimais (precisão de 6 casas).

### Caso B: Sem Fix de GPS (Ausência de Sinal)
Enviado a cada 5 segundos quando o caiaque está no mar mas não consegue calcular as coordenadas (ex: tempo nublado, obstáculos).
```json
{
  "id": 1,
  "lat": null,
  "lng": null
}
```

---

## 3. APIs REST do Backend (HTTPS)

O gateway receptor na terra decodifica o JSON de rádio e realiza chamadas HTTPS na API do NestJS.

### Ingestão de Coordenadas
*   **Endpoint:** `POST /api/tracking`
*   **Content-Type:** `application/json`
*   **Payload:** *(JSON de Telemetria detalhado na Seção 2)*
*   **Comportamento do Servidor:**
    1.  Recebe as coordenadas.
    2.  Procura o caiaque no banco de dados pelo seu `deviceId` (equivalente ao campo `id` do JSON).
    3.  Caso o caiaque não exista, realiza um auto-cadastro com o status inicial `IN_USE`.
    4.  Caso exista, atualiza o status para `IN_USE`.
    5.  Registra a nova coordenada histórica na tabela `TrackingPoint`.
    6.  Realiza o broadcast imediato dos dados via WebSocket.
*   **Resposta (HTTP 200 OK):**
    ```json
    {
      "ok": true
    }
    ```

### Ping de Estação Base (Heartbeat)
*   **Endpoint:** `GET /api/tracking/heartbeat`
*   **Comportamento do Servidor:** Recebe a requisição indicando que a estação base está operando normalmente e faz um broadcast via WebSocket informando que o receptor terrestre está online.
*   **Resposta (HTTP 200 OK):**
    ```json
    {
      "ok": true
    }
    ```

### Consulta Inicial do Painel
*   **Endpoint:** `GET /api/tracking`
*   **Comportamento do Servidor:** Retorna as últimas posições conhecidas de todos os caiaques para carga inicial da tela de monitoramento.

---

## 4. Broadcast de Tempo Real (WebSockets / WSS)

O backend expõe um servidor WebSocket na porta `/ws` para os navegadores dos operadores terrestres.

### Mensagens Transmitidas pelo Servidor:

#### 1. Confirmação de Conexão (Ao conectar ao /ws)
```json
{
  "tipo": "conectado"
}
```

#### 2. Status da Estação Base (Disparado pelo Heartbeat da base)
```json
{
  "tipo": "base"
}
```
*   *Nota:* O frontend interpreta esta mensagem para acender o led indicador de "Estação Base Online". Se o frontend passar mais de 120 segundos sem receber esse evento, define a base como "Offline".

#### 3. Posição de Telemetria do Caiaque (Disparado pela ingestão de dados)
```json
{
  "id": 1,
  "lat": -23.961234,
  "lng": -46.331234
}
```
*   *Nota:* Repassa os valores de latitude, longitude e ID recebidos do caiaque em tempo real para atualização imediata dos marcadores no mapa Leaflet.
