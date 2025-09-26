# GUIA DE INSTALAÇÃO - pH Controller v6.0

## CHECKLIST PRÉ-INSTALAÇÃO

### ✅ BACKUP DE SEGURANÇA
**IMPORTANTE:** Antes de instalar a nova versão, anote os valores atuais:

1. **pH Ideal atual:** ___________
2. **Última calibração:** ___________
3. **Configurações especiais:** ___________

### ✅ VERIFICAÇÃO DE HARDWARE
- [ ] Arduino Nano funcional
- [ ] Display LCD 20x4 respondendo
- [ ] Sensor DS18B20 conectado (pino 2)
- [ ] ADS1115 conectado via I2C
- [ ] Botões funcionais (pinos 3, 4, 5)
- [ ] Bomba conectada (pino 6)
- [ ] LED conectado (pino 7)

---

## INSTALAÇÃO PASSO A PASSO

### PASSO 1: PREPARAÇÃO DO AMBIENTE
```
1. Abra Arduino IDE
2. Verifique se as bibliotecas estão instaladas:
   - Wire (nativa)
   - LiquidCrystal_I2C
   - OneWire
   - DallasTemperature
   - Adafruit_ADS1X15
3. Selecione: Tools > Board > Arduino Nano
4. Configure: Tools > Processor > ATmega328P (Old Bootloader)
```

### PASSO 2: UPLOAD DO FIRMWARE
```
1. Conecte o Arduino Nano ao computador
2. Abra o arquivo: arduino_ph_controller_optimized.ino
3. Clique em Upload (Ctrl+U)
4. Aguarde confirmação: "Done uploading"
```

### PASSO 3: PRIMEIRA INICIALIZAÇÃO
```
1. Desconecte e reconecte a alimentação
2. Observe a tela de inicialização:
   "pH Controller v6.0"
   "Inicializando..."
   [Barra de progresso]
3. Aguarde 10-15 segundos para estabilização
```

---

## VERIFICAÇÃO PÓS-INSTALAÇÃO

### ✅ TESTE BÁSICO DE FUNCIONAMENTO

**1. TELA PRINCIPAL**
```
STATUS ATUAL |
PH Medio: X.XX
PH Inst.: X.XX
T:XX.X°C Ideal:X.X
```

**2. NAVEGAÇÃO DE MENU**
```
- Toque curto no MENU → Menu Principal
- UP/DOWN → Navegar opções
- MENU → Selecionar
- Toque longo no MENU (2s) → Menu Técnico
```

**3. TESTE DE SENSORES**
```
- Temperatura deve mostrar valor coerente (15-35°C ambiente)
- pH deve mostrar valor estável (mesmo incorreto antes da calibração)
- Valores não devem mostrar "nan" ou números absurdos
```

### ✅ RECUPERAÇÃO DE CONFIGURAÇÕES

**CENÁRIO 1: Configurações Preservadas**
```
✅ pH Ideal mantido
✅ Calibração preservada
✅ Sistema operacional
→ Prossiga para calibração de verificação
```

**CENÁRIO 2: Configurações Padrão Carregadas**
```
ℹ️ Sistema detectou dados inválidos
ℹ️ Carregou valores padrão seguros
ℹ️ Mensagem no LCD: "CONFIG. INVALIDA / CARREGANDO PADROES"
→ Prossiga para nova calibração
```

---

## CALIBRAÇÃO INICIAL/VERIFICAÇÃO

### CALIBRAÇÃO DE 2 PONTOS (RECOMENDADA)

**PREPARAÇÃO:**
- [ ] Soluções buffer pH 4.01 e 7.01
- [ ] Água destilada para lavagem
- [ ] Papel absorvente limpo

**PROCEDIMENTO:**
```
1. Menu Principal → Calibrar Sonda → Calibrar: 2 Pontos

2. PONTO 1 (pH 7.01):
   - Selecione "Solução 7.01"
   - Pressione MENU
   - Lave a sonda, mergulhe em pH 7.01
   - Pressione MENU quando estável
   - Aguarde leitura de 30 segundos

3. PONTO 2 (pH 4.01):
   - Sistema solicita segundo ponto
   - Selecione "Solução 4.01"  
   - Pressione MENU
   - Lave a sonda, mergulhe em pH 4.01
   - Pressione MENU quando estável
   - Aguarde leitura de 30 segundos

4. FINALIZAÇÃO:
   - Sistema calcula slope
   - Se slope normal: "Calibração Concluída!"
   - Se slope anormal: Tela de confirmação
```

### VALIDAÇÃO PÓS-CALIBRAÇÃO
```
1. Teste em pH 7.01: Deve ler 7.01 ±0.05
2. Teste em pH 4.01: Deve ler 4.01 ±0.05
3. Ajuste pH Ideal para valor desejado
4. Teste operação da bomba em modo automático
```

---

## RECURSOS NOVOS DISPONÍVEIS

### 🆕 SISTEMA DE RECUPERAÇÃO AUTOMÁTICA
```
- Detecção automática de travamentos
- Recuperação de configurações corrompidas
- Reinicialização inteligente em caso de erro
- Não requer intervenção manual
```

### 🆕 MENU TÉCNICO APRIMORADO
```
Acesso: Toque longo (2s) no botão MENU na tela principal

Opções disponíveis:
- Diagnóstico da Sonda: Mede tensões brutas
- Limites de Slope: Configura faixas aceitáveis
- Estatísticas: Tempo de operação, calibrações, erros
```

### 🆕 INDICADORES VISUAIS MELHORADOS
```
- Indicador giratório de atividade (|/-\)
- Alertas mais claros para pH baixo
- Barras de progresso em calibrações
- Códigos de erro informativos
```

---

## SOLUÇÃO DE PROBLEMAS

### ❌ PROBLEMA: "Erro ADS1115!"
```
CAUSA: Módulo ADC não detectado
SOLUÇÃO:
1. Verifique conexões I2C (SDA/SCL)
2. Teste continuidade dos fios
3. Verifique alimentação do módulo
4. Confirme endereço I2C (0x48 padrão)
```

### ❌ PROBLEMA: "ERRO DE SENSOR!"
```
CAUSA: Sensor de temperatura desconectado
SOLUÇÃO:
1. Verifique conexão do DS18B20 no pino 2
2. Confirme alimentação 3.3V/5V
3. Teste resistor pull-up de 4.7kΩ
4. Reinicie o sistema
```

### ❌ PROBLEMA: pH sempre 7.00
```
CAUSA: Calibração não realizada ou inválida
SOLUÇÃO:
1. Execute calibração de 2 pontos
2. Verifique qualidade das soluções buffer
3. Confirme limpeza da sonda
4. Use Menu Técnico → Diagnóstico da Sonda
```

### ❌ PROBLEMA: Sistema reinicia sozinho
```
CAUSA: Fonte de alimentação instável
SOLUÇÃO:
1. Adicione capacitor 100-1000µF entre VCC/GND
2. Verifique corrente da fonte (mín. 500mA)
3. Use fonte estabilizada
4. Evite cabos muito longos
```

---

## MANUTENÇÃO PREVENTIVA

### DIÁRIA
- [ ] Verificar leituras de pH e temperatura
- [ ] Confirmar funcionamento da bomba
- [ ] Observar alertas no display

### SEMANAL
- [ ] Limpeza da sonda de pH
- [ ] Verificação das soluções buffer
- [ ] Teste de calibração (opcional)

### MENSAL
- [ ] Calibração completa de 2 pontos
- [ ] Limpeza geral do sistema
- [ ] Verificação de conexões
- [ ] Backup manual das configurações

---

## SUPORTE TÉCNICO

### INFORMAÇÕES PARA SUPORTE
Ao solicitar suporte, forneça:
```
- Versão do firmware: v6.0
- Modelo do hardware: Arduino Nano
- Descrição do problema
- Código de erro (se houver)
- Última calibração realizada
- Modificações no hardware
```

### LOGS DE DEBUG
Para análise técnica, conecte via Serial (9600 baud):
```
Arduino IDE → Tools → Serial Monitor
Velocidade: 9600 baud
Copie as mensagens exibidas durante o problema
```

---

## CONCLUSÃO

A instalação da versão 6.0 oferece melhorias significativas em confiabilidade e robustez, mantendo total compatibilidade com o hardware existente. O sistema agora é mais resistente a falhas e oferece recursos avançados de diagnóstico e recuperação.

**Em caso de dúvidas ou problemas, documente detalhadamente o comportamento observado e as mensagens exibidas para facilitar o suporte técnico.**