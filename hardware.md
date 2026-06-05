# Hardware e Firmware Embarcado

Esta documentação descreve as especificações físicas dos dispositivos eletrônicos embarcados no caiaque (Transmissor) e instalados na estação em terra (Receptor Gateway).

---

## 1. Módulo Móvel: Caiaque (Transmissor)

Cada caiaque operacional possui uma caixa estanque embarcada contendo um microcontrolador, um módulo de geolocalização e um rádio transmissor alimentados por bateria recarregável.

```
[ Antena GPS ]      [ Antena LoRa ]
      ↓                    ↑
 [ GPS NEO-6M ]  →  [ ESP32/Nano ]  →  [ LoRa E32 ]
```

### Componentes de Hardware:
*   **Microcontrolador Principal:** ESP32 ou Arduino Nano (núcleo de processamento e gerenciamento de energia).
*   **Módulo GPS (NEO-6M):** Antena integrada de alta sensibilidade para aquisição de coordenadas via satélites GNSS.
*   **Módulo LoRa (E32-900T20D):** Transmissor de rádio baseado no chip Semtech SX1276 operando na faixa de 915 MHz com potência de 20dBm (100mW) para longo alcance.

### Mapeamento Físico de Pinos (UART):
*   **Conexão GPS $\leftrightarrow$ ESP32:**
    *   GPS `TX` ligado ao GPIO `22` (RX1 do ESP32)
    *   GPS `RX` ligado ao GPIO `23` (TX1 do ESP32)
    *   Baud Rate: `9600 bps` (padrão de fábrica do NEO-6M)
*   **Conexão LoRa $\leftrightarrow$ ESP32:**
    *   LoRa `TX` ligado ao GPIO `16` (RX2 do ESP32)
    *   LoRa `RX` ligado ao GPIO `17` (TX2 do ESP32)
    *   Baud Rate: `9600 bps`

### Lógica do Firmware (C++ / Arduino framework):
1.  **Inicialização:** Configura as seriais, inicializa o rádio LoRa e define a potência para `POWER_20` (20dBm) e a taxa de transmissão aérea para `AIR_DATA_RATE_001_12` (1.2 kbps).
2.  **Ciclo de Leitura:** Roda em loop contínuo lendo bytes do módulo GPS e passando para a biblioteca `TinyGPS++` para decodificação.
3.  **Transmissão (Intervalo de 5 segundos):**
    *   Se as coordenadas lidas forem válidas, monta um payload JSON: `{"id": 1, "lat": -23.961234, "lng": -46.331234}` e dispara via rádio.
    *   Se não houver conexão de satélites ativa (Sem Fix GPS), monta um payload alternativo indicando ausência de sinal: `{"id": 1, "lat": null, "lng": null}` para fins de diagnóstico.

---

## 2. Módulo Fixo: Estação Base (Receptor Gateway)

Instalada na central em terra, a estação base recebe as transmissões de rádio e as retransmite para a internet utilizando redes WiFi locais.

### Componentes de Hardware:
*   **Microcontrolador Integrado:** ESP32 SoC (WiFi de 2.4 GHz integrado de fábrica).
*   **Módulo LoRa (E32-900T20D):** Configurado na mesma frequência (915 MHz) e taxa (1.2 kbps) para receber a telemetria do caiaque.

### Mapeamento Físico de Pinos (UART):
*   **Conexão LoRa $\leftrightarrow$ ESP32:**
    *   LoRa `TX` ligado ao GPIO `16` (RX2 do ESP32)
    *   LoRa `RX` ligado ao GPIO `17` (TX2 do ESP32)
    *   Baud Rate: `9600 bps`

### Lógica do Firmware (C++ / Arduino framework):
1.  **Inicialização:** Configura o módulo LoRa nas mesmas definições da embarcação, inicializa a stack `WiFi.h` e se conecta à rede sem fio disponível.
2.  **Encaminhamento de Dados:** Ao receber um pacote LoRa válido, o firmware realiza uma chamada HTTP segura (**HTTPS POST**) enviando o JSON diretamente ao backend hospedado na nuvem.
3.  **Sinalizador de Atividade (Heartbeat):** A cada 10 segundos, caso a estação esteja ativa, envia um **HTTPS GET** ao endpoint `/api/tracking/heartbeat`. O recebimento deste ping avisa a equipe de operadores que a estação base está ligada e conectada.
