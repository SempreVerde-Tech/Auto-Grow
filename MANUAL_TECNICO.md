# Projeto ALEPH - Manual Técnico

**Versão do Documento:** 1.0
**Versão do Software:** 2.2

---

## 1. Visão Geral do Projeto

O ALEPH é um controlador de pH automatizado, projetado para monitorar e manter o nível de pH de soluções líquidas, primariamente em sistemas de cultivo hidropônico ou similares. O sistema mede continuamente o pH e a temperatura da solução e, caso o pH exceda o valor ideal configurado, aciona uma bomba peristáltica para dosar uma solução de "pH Down", corrigindo o nível.

O projeto foi desenvolvido utilizando a plataforma Arduino (especificamente o Arduino Nano) e é construído com componentes de baixo custo e fácil acesso. A arquitetura de software é baseada em uma máquina de estados finitos, tornando-o robusto e não-bloqueante, o que permite que a interface do usuário permaneça responsiva enquanto tarefas de monitoramento e controle ocorrem em segundo plano.

## 2. Componentes e Hardware

### 2.1. Lista de Componentes

| Componente                             | Função Principal                                       |
| -------------------------------------- | ------------------------------------------------------ |
| **Arduino NANO**                       | Cérebro do sistema, executa o código de controle.      |
| **Tela LCD I2C 20x4**                  | Interface do usuário (exibe status e menus).           |
| **Módulo ADC ADS1115**                 | Conversor Analógico-Digital de 16 bits para leitura precisa da sonda de pH. |
| **Módulo de pH com Sonda**             | Mede o potencial de hidrogênio (pH) da solução.        |
| **Módulo de Temp. DS18B20 com Sonda**  | Mede a temperatura para compensação da leitura de pH.    |
| **Módulo MOSFET IRF520 (ou similar)**  | Atua como um interruptor para ligar/desligar a bomba.    |
| **Bomba Peristáltica 5V**              | Dosa a solução de correção de pH.                      |
| **3x Botões Touch (ou Push-buttons)**  | Navegação na interface (MENU, UP, DOWN).               |
| **LED 3V**                             | Indicador visual de alertas.                           |
| **Fonte 5V 2A com Conector USB-C**     | Alimentação principal do sistema.                      |
| **Componentes de Proteção**            |                                                        |
| - Díodo TVS 5kpa                       | Proteção contra surtos de tensão (transientes).        |
| - Díodo Schottky 1N5822                | Proteção contra inversão de polaridade.                |
| - Polyfuse 3A                          | Fusível rearmável para proteção contra sobrecorrente.  |
| **Componentes Passivos**               | Capacitores (cerâmicos/eletrolíticos) e resistores para filtragem, pull-ups, etc. |

### 2.2. Diagrama de Pinos e Conexões

A seguir, a pinagem utilizada no Arduino Nano, conforme definido no código (`const int PIN_...`):

| Pino Arduino | Conectado a                               | Propósito                                       |
| :----------: | ----------------------------------------- | ----------------------------------------------- |
| **D2**       | Pino de Dados do Sensor DS18B20           | Comunicação One-Wire com o sensor de temperatura. |
| **D3**       | Botão UP (Cima)                           | Navegação nos menus (incrementar valor).        |
| **D4**       | Botão DOWN (Baixo)                        | Navegação nos menus (decrementar valor).        |
| **D5**       | Botão MENU                                | Entrar/sair de menus, confirmar seleção.        |
| **D6 (PWM)** | Pino de Sinal (Gate) do Módulo MOSFET     | Controle de velocidade e acionamento da bomba.  |
| **D7**       | Anodo (+) do LED de Alerta                | Indicador visual para pH baixo e erros.         |
| **A4 (SDA)** | Pino SDA do LCD I2C e do ADC ADS1115      | Linha de dados para comunicação I2C.            |
| **A5 (SCL)** | Pino SCL do LCD I2C e do ADC ADS1115      | Linha de clock para comunicação I2C.            |
| **GND**      | Comum (Terra) de todos os módulos.        | Referência de 0V para todo o circuito.          |
| **+5V**      | Alimentação de todos os módulos.          | Tensão de alimentação de 5V.                    |

**Conexões Adicionais:**
- **Sonda de pH:** Conectada à entrada do módulo amplificador de pH. A saída do módulo é conectada ao canal **A0** do ADC ADS1115.
- **ADDR do ADS1115:** Conectado ao GND para definir o endereço I2C como `0x48`.
- **Botões:** Conectados com resistores de pull-up internos do Arduino. O outro terminal do botão deve ser conectado ao GND.
- **LED:** O catodo (-) do LED deve ser conectado a um resistor (ex: 220Ω) e, em seguida, ao GND.

## 3. Arquitetura de Software

### 3.1. Máquina de Estados

O núcleo do software é uma máquina de estados finitos controlada pela `enum SystemState`. Isso evita o uso de `delay()` no `loop()` principal, garantindo que o sistema esteja sempre responsivo à leitura de sensores e à interação do usuário. O `loop()` principal chama a função de tratamento (`handle...()`) correspondente ao estado atual.

### 3.2. Controle de pH em Segundo Plano

A amostragem de pH (`readSensors()`) e a lógica de controle (`checkPhAndControl()`) são executadas continuamente dentro do `loop()` principal, sob a condição `if (currentState == STATE_MONITORING || currentState == STATE_SCREENSAVER)`. Isso garante que, mesmo com o protetor de tela ativo, o sistema continue a monitorar e a corrigir o pH da solução, uma característica essencial para a estabilidade do sistema.

### 3.3. Lógica de Alertas

O sistema possui três níveis de alerta visual:
1.  **LED Piscando:** Indica um erro crítico que exige intervenção imediata (`probeError` ou `solutionError`).
2.  **LED Aceso Contínuo:** Indica que o pH está abaixo do ideal (`currentPhStatus == PH_LOW`), uma condição de atenção, mas não um erro.
3.  **LED Apagado:** Condição normal de operação.

Essa lógica é centralizada no final do `loop()` para garantir uma hierarquia clara de prioridades.

## 4. Configurações e EEPROM

Todas as configurações personalizáveis são armazenadas na `struct SystemConfig` e salvas na memória EEPROM do Arduino para persistirem após uma reinicialização.

### 4.1. Estrutura `SystemConfig`

```c
struct SystemConfig {
  float phIdeal;
  float calibOffsetVoltage;
  float calibSlope;
  bool isCalibrated2Points;
  unsigned long pumpDurationMs;
  unsigned int pumpCooldownMinutes;
  byte pumpSpeedPercentage;
};
```

### 4.2. Parâmetros de Fábrica (Valores Padrão)

Se a EEPROM não tiver sido inicializada, o sistema carrega os seguintes valores padrão:

| Parâmetro               | Valor Padrão | Descrição                                        |
| ----------------------- | ------------ | -------------------------------------------------- |
| `phIdeal`               | 7.0          | pH alvo que o sistema tentará manter.            |
| `calibOffsetVoltage`    | 0.0 V        | Tensão de offset (ponto zero).                     |
| `calibSlope`            | -0.05916     | Slope teórico de uma sonda de pH a 25°C.           |
| `pumpDurationMs`        | 1000 ms (1s) | Duração padrão de cada dosagem da bomba.           |
| `pumpCooldownMinutes`   | 5 min        | Tempo de espera padrão entre as dosagens.          |
| `pumpSpeedPercentage`   | 25%          | Velocidade/vazão padrão da bomba.                  |

## 5. Detalhamento de Funções e Constantes

### 5.1. Constantes de Controle

- `PH_CONTROL_THRESHOLD` (0.3): Define a "zona morta" acima do pH ideal. A bomba só é acionada se `pH atual > (pH ideal + 0.3)`.
- `PH_STABILITY_THRESHOLD` (0.1): As leituras de pH são consideradas estáveis (e aptas para uma decisão de controle) somente se a variação entre as 3 últimas médias for menor que 0.1.
- `PH_CHANGE_TOLERANCE` (0.05): Uma dosagem só é considerada bem-sucedida se a queda de pH for de pelo menos 0.05. Evita que ruído do sensor seja interpretado como uma correção.

### 5.2. Funções Principais

- `loop()`: Ciclo principal. Orquestra a leitura de sensores, a máquina de estados e a lógica de alerta do LED.
- `readSensors()`: Coleta amostras de pH e temperatura em intervalos definidos.
- `processPhSamples()`: Calcula a média das amostras de pH (descartando os valores extremos) e executa as verificações de erro da sonda.
- `checkPhAndControl()`: Contém a lógica de decisão principal. Com base no pH médio estável, decide se o estado é `PH_NORMAL`, `PH_LOW` ou `PH_HIGH` e se a bomba deve ser acionada.
- `activatePump()` / `activatePumpSilently()`: Funções que acionam a bomba (com e sem feedback visual na tela, respectivamente), utilizando as configurações de duração e velocidade.
- `handle...()`: Cada estado da `enum SystemState` possui uma função `handle` correspondente que gerencia a lógica e a interface para aquele estado específico.
- `loadConfigFromEEPROM()` / `saveConfigToEEPROM()`: Funções responsáveis pela persistência dos dados de configuração na memória não volátil.

---
**Fim do Manual Técnico**