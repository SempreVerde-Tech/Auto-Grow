# Histórico de Versões do Projeto ALEPH

Este documento detalha as principais alterações, novas funcionalidades e correções de bugs introduzidas em cada versão do software do controlador de pH ALEPH.

---

### Versão 1.0: A Gênese
- **Data:** 2024-07-25
- **Descrição:** Versão inicial do sistema.
- **Funcionalidades:**
  - Estrutura básica com `setup()` e `loop()`.
  - Leitura de pH via ADC ADS1115 e de temperatura via DS18B20.
  - Arquitetura baseada em máquina de estados (`enum SystemState`).
  - Menus de interface para:
    - Definir pH Ideal.
    - Calibragem de 1 e 2 pontos.
    - Menu Técnico com visualização de valores brutos e teste de sensores.
  - Lógica inicial de controle: ativa a bomba de pH Down se `pH atual > pH ideal`.
  - Armazenamento de configurações (calibragem, pH ideal) na EEPROM.
- **Problemas Conhecidos:**
  - A tela de monitoramento sofre com cintilação (flicker) devido a `lcd.clear()` a cada ciclo.
  - A ativação da bomba utiliza a função `delay()`, bloqueando todo o sistema durante a dosagem.
  - A fórmula de conversão de voltagem para pH está com o sinal invertido.

---

### Versão 1.1: Correção de Hardware
- **Descrição:** Pequena atualização para corrigir a pinagem do hardware.
- **Mudanças:**
  - Invertidos os pinos dos botões MENU (`pino 5`) e UP (`pino 3`) para corresponder à placa física.

---

### Versão 1.2: Melhoria Visual
- **Descrição:** Foco em melhorar a experiência do usuário, eliminando a cintilação da tela.
- **Mudanças:**
  - Refatorada a lógica de exibição da tela de monitoramento para um modelo de "desenhar layout uma vez, atualizar valores depois".
  - Introduzida a flag `screenNeedsRedraw` para controlar quando a tela inteira deve ser redesenhada.

---

### Versão 1.3: Correção Crítica de Lógica
- **Descrição:** Correção do bug fundamental no cálculo do pH.
- **Mudanças:**
  - Corrigida a fórmula em `convertVoltageToPh` de `7.0 - ...` para `7.0 + ...`, alinhando-a com a equação de Nernst para sondas de pH.
  - Otimizada a tela de "Valores Brutos" para também não piscar.

---

### Versão 1.4 & 1.4.1: Refinamento da Interface
- **Descrição:** Melhorias na clareza e robustez da interface de monitoramento.
- **Mudanças:**
  - **(1.4)** Adicionado um espaço inicial nas mensagens de status (`" MONITORANDO "`) para corrigir bugs de alinhamento no display.
  - **(1.4.1)** Introduzida a `enum PhStatus` (`PH_NORMAL`, `PH_LOW`, `PH_HIGH`) para um controle de estado mais claro, substituindo a verificação direta de pinos.
  - A tela de monitoramento agora exibe mensagens específicas para cada estado (`PH ATUAL < IDEAL`, `AJUSTANDO PH`, etc.), melhorando o feedback ao usuário.

---

### Versão 1.5: O Protetor de Tela
- **Descrição:** Introdução de um protetor de tela para evitar burn-in no LCD.
- **Funcionalidades:**
  - Adicionado o estado `STATE_SCREENSAVER` e um timeout de inatividade.
  - Implementada uma animação de "Tetris reverso" como protetor de tela.
- **Problema Crítico Introduzido:** A lógica de monitoramento e controle de pH não era executada durante o screensaver.

---

### Versão 1.6: Multitarefa
- **Descrição:** Correção da falha crítica do screensaver e adição de funcionalidade.
- **Mudanças:**
  - **Correção Crítica:** A lógica de amostragem e controle de pH foi movida para o `loop()` principal, garantindo que o sistema continue funcionando em segundo plano, mesmo com o screensaver ativo.
  - **Nova Funcionalidade:** Adicionada a opção "Ativar Bomba" no Menu Técnico para testes manuais.

---

### Versão 1.6.1 a 1.6.3: Correções e Otimizações
- **Descrição:** Série de pequenas correções de bugs e melhorias de performance.
- **Mudanças:**
  - **(1.6.1)** Corrigido bug na lógica de ativação manual da bomba.
  - **(1.6.2)** **Regressão:** A lógica de ativação manual da bomba foi acidentalmente invertida.
  - **(1.6.3)** Corrigida a regressão da v1.6.2 e implementada uma otimização radical no screensaver, substituindo `lcd.clear()` por um sistema de buffer duplo (`lcdBuffer`, `prevLcdBuffer`) para eliminar totalmente a cintilação da animação.

---

### Versão 1.7 a 1.7.5: Mais Controle e Segurança
- **Descrição:** Adição de mais configurações para o usuário e melhorias na segurança.
- **Mudanças:**
  - **(1.7)** O tempo de acionamento da bomba (`pumpDurationMs`) tornou-se configurável pelo usuário no Menu Técnico e salvo na EEPROM.
  - **(1.7.1)** Corrigido bug no menu de ajuste do timer da bomba, que não carregava o valor salvo para edição.
  - **(1.7.2)** A animação do screensaver foi simplificada de "Tetris" para "Chuva", reduzindo o uso de memória e a complexidade do código.
  - **(1.7.3)** Corrigido bug de navegação no menu de confirmação de reinicialização.
  - **(1.7.4)** O sistema agora sai automaticamente do screensaver se a bomba for ativada, garantindo que o usuário veja a ação.
  - **(1.7.5)** A ativação da bomba foi refatorada para ser **não-bloqueante**, usando um novo estado `STATE_PUMPING` e a função `millis()`, eliminando o último grande `delay()` do código e evitando que a animação do screensaver congele.

---

### Versão 1.8 a 1.8.6: Robustez e Alertas Avançados
- **Descrição:** Foco em tornar o sistema mais robusto e inteligente, com novos alertas de segurança.
- **Mudanças:**
  - **(1.8)** Introduzido o alerta "VERIFICAR SOLUCAO PH", que é acionado se a bomba for ativada duas vezes consecutivas sem que o pH diminua.
  - **(1.8.1)** O screensaver agora também é interrompido pelo novo alerta de erro de solução.
  - **(1.8.2)** A lógica de detecção de falha da solução foi aprimorada para comparar com o pH no início do ciclo de correção, em vez da medição anterior.
  - **(1.8.4)** Adicionada uma tolerância (`PH_CHANGE_TOLERANCE`) para evitar que ruídos do sensor desativem o alerta de falha da solução indevidamente.
  - **(1.8.5)** A bomba não é mais bloqueada pelo alerta de falha da solução, permitindo que o sistema continue tentando corrigir o pH enquanto notifica o usuário.
  - **(1.8.6)** O LED de alerta agora tem comportamento diferenciado: pisca para erros críticos (solução/sonda) e fica aceso contínuo para pH baixo.

---

### Versão 1.9 a 1.9.3: Finalizando as Configurações
- **Descrição:** Adição das últimas funcionalidades de configuração e correções de bugs relacionadas.
- **Mudanças:**
  - **(1.9)** O tempo de "cooldown" da bomba tornou-se configurável (1 a 60 minutos) no Menu Técnico.
  - **(1.9.1)** Adicionado o alerta "CHECAR SONDA", que é acionado se as leituras de pH estiverem fora da faixa (0-14) ou se todas as amostras forem idênticas.
  - **(1.9.2)** Adicionada a configuração de **velocidade da bomba** (5-100%) no Menu Técnico.
  - **(1.9.3)** Corrigido bug que impedia o salvamento correto da velocidade da bomba na EEPROM.

---

### Versão 2.0: Refatoração da Velocidade da Bomba
- **Descrição:** Melhoria na forma como a velocidade da bomba é armazenada e utilizada.
- **Mudanças:**
  - A velocidade da bomba agora é salva na EEPROM como uma **porcentagem** (5-100), o mesmo valor que o usuário vê. A conversão para PWM (13-255) é feita apenas no momento de ativar a bomba, evitando perda de precisão e tornando o código mais intuitivo.

---

### Versão 2.1: Usabilidade dos Menus
- **Descrição:** Correção final na lógica de edição de valores nos menus.
- **Mudanças:**
  - Garantido que, ao entrar em qualquer tela de edição de valor (timer, cooldown, velocidade), o valor atual seja sempre carregado da configuração para uma variável temporária. A tela de edição passa a trabalhar apenas com a variável temporária, resolvendo inconsistências e tornando a experiência do usuário mais robusta.

---

### Versão 2.2 (Final): Documentação e Comentários
- **Descrição:** Versão final do código, com foco em limpeza, comentários e documentação interna.
- **Mudanças:**
  - Adicionados comentários detalhados em português em todo o código para explicar a função de cada bloco, constante e variável.
  - Revisão geral do código para garantir legibilidade e manutenibilidade.