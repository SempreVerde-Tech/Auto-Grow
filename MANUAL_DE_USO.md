# Manual de Uso e Operação - Controlador de pH Profissional

**Versão do Firmware:** 5.0

## 1. Visão Geral

Este documento descreve como operar o sistema de controle de pH. O sistema é projetado para ser robusto e preciso, com funcionalidades avançadas para garantir a confiabilidade em ambientes comerciais.

**Principais Funcionalidades:**
-   Monitoramento contínuo do pH e da temperatura.
-   Controle automático de uma bomba para dosagem de solução.
-   Calibração de 1 e 2 pontos com compensação de temperatura.
-   Menu Técnico para diagnóstico e configuração de sensores não-padrão.
-   Sistema de segurança com Watchdog Timer e validação de leituras.

---

## 2. Operação Básica

### 2.1. Tela Principal (STATUS ATUAL)

Esta é a tela principal do sistema. Ela exibe as informações mais importantes em tempo real:
-   **PH Medio:** A média estabilizada das últimas leituras de pH. É este valor que o sistema usa para tomar decisões.
-   **PH Inst.:** O valor de pH instantâneo. É normal que este valor flutue um pouco.
-   **T:** A temperatura atual da solução, já com o ajuste de calibração aplicado.
-   **Ideal:** O valor de pH que o sistema tentará manter.

Um indicador giratório (`| / - \\`) no canto superior direito mostra que o sistema está ativo e fazendo leituras.

### 2.2. Acessando o Menu

-   Para acessar o **Menu Principal**, dê um **toque curto** (menos de 2 segundos) no botão `MENU`.

---

## 3. Menu Principal

Navegue pelos menus usando os botões `UP` (Cima) e `DOWN` (Baixo). Selecione uma opção com um toque curto no botão `MENU`.

### 3.1. Ajustar PH Ideal

Permite definir o valor de pH alvo que o sistema deve manter.
-   Use os botões `UP` e `DOWN` para aumentar ou diminuir o valor.
-   Pressione `MENU` para salvar e voltar à tela principal.

### 3.2. Calibrar Sonda

Leva ao **Menu de Calibração**, onde você pode calibrar a sonda de pH.

### 3.3. Voltar

Retorna à tela principal.

---

## 4. Menu de Calibração

A calibração periódica é **essencial** para a precisão do sistema. Recomenda-se uma calibração de 2 pontos semanalmente.

### 4.1. Calibrar: 1 Ponto

-   **Uso:** Apenas para pequenos ajustes rápidos se você já tem uma calibração de 2 pontos bem-sucedida. Não é um substituto para a calibração completa.
-   **Procedimento:**
    1.  Selecione a solução buffer que deseja usar (4.01, 7.01 ou 10.01).
    2.  Mergulhe a sonda na solução e pressione `MENU`.
    3.  Aguarde a estabilização e a confirmação.

### 4.2. Calibrar: 2 Pontos (Recomendado)

-   **Uso:** O método mais preciso. Define tanto o ponto zero (offset) quanto a sensibilidade (slope) da sonda.
-   **Procedimento:**
    1.  **Primeiro Ponto (Recomendado: pH 7.01):**
        -   Selecione a solução de pH 7.01.
        -   Lave a sonda em água destilada, seque-a suavemente.
        -   Mergulhe na solução de pH 7.01 e pressione `MENU`.
        -   Aguarde a leitura e a confirmação.
    2.  **Segundo Ponto (Recomendado: pH 4.01):**
        -   Lave a sonda novamente.
        -   O sistema irá pedir para você selecionar o segundo ponto. Escolha a solução de pH 4.01.
        -   Mergulhe na solução de pH 4.01 e pressione `MENU`.
        -   Aguarde a leitura.

-   **Slope Inválido (Tela de Confirmação):** Se o sistema detectar que a sensibilidade (slope) da sua sonda é muito diferente do padrão, ele exibirá o valor medido e pedirá sua confirmação.
    -   `Aceitar? > Nao / Sim`
    -   **Nao:** Cancela a calibração. Verifique sua sonda e soluções.
    -   **Sim:** (Para técnicos) Força o sistema a aceitar os valores da sonda não-padrão.

---

## 5. Menu Técnico (Acesso Avançado)

Para acessar o Menu Técnico, na **tela principal**, **pressione e segure o botão `MENU` por mais de 2 segundos.**

Este menu é para configuração avançada e diagnóstico do hardware.

### 5.1. Diagnostico Sonda

-   **Função:** Permite medir e exibir as tensões brutas da sonda nas soluções de pH 7 e 4, e calcula o slope resultante.
-   **Uso:** Ideal para verificar a saúde de uma sonda nova ou suspeita, e para determinar os limites de slope corretos para um lote de sensores não-padrão.
-   **Procedimento:** Siga as instruções na tela, que são similares a uma calibração de 2 pontos, mas apenas para diagnóstico (não salva os valores).

### 5.2. Limites de Slope

-   **Função:** Permite ajustar os limites mínimo e máximo de slope que o sistema considera válidos durante a calibração.
-   **Uso:** Use esta função se você trabalha com um lote de sensores que têm uma sensibilidade consistentemente fora do padrão industrial.
-   **Procedimento:**
    1.  Use `UP`/`DOWN` para selecionar o limite (Mínimo ou Máximo).
    2.  Pressione `MENU` para editar o valor.
    3.  Use `UP`/`DOWN` para ajustar o valor.
    4.  Pressione `MENU` para confirmar o novo valor.
    5.  Selecione "Salvar e Sair" para gravar as novas configurações na memória.

### 5.3. Voltar

Retorna à tela principal.

---

## 6. Solução de Problemas (FAQ)

-   **O sistema reinicia sozinho quando uso uma fonte externa.**
    -   **Causa:** Instabilidade ou ruído na fonte de alimentação.
    -   **Solução:** Adicione um capacitor (10µF a 1000µF) entre os pinos 5V e GND do Arduino para estabilizar a energia.

-   **A calibração sempre falha com "Slope Inválido".**
    -   **Causa:** A sua sonda é muito mais (ou menos) sensível que o padrão.
    -   **Solução:** Use o `Menu Técnico > Diagnostico Sonda` para descobrir o slope real. Depois, use `Menu Técnico > Limites de Slope` para ajustar os limites mínimo e máximo para que incluam o valor medido.

-   **As leituras de pH estão "travadas" ou erradas após uma calibração.**
    -   **Causa:** Dados de calibração corrompidos na memória.
    -   **Solução:** Use o código de limpeza de EEPROM (fornecido separadamente) para apagar a memória. Depois, recarregue o firmware principal. O sistema carregará os valores padrão e estará pronto para uma nova calibração.