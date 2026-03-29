# Controle de Saída Digital com PWM no ESP32

![ESP-IDF](https://img.shields.io/badge/ESP--IDF-v5.0+-E03C31?style=for-the-badge&logo=espressif)
![C](https://img.shields.io/badge/Linguagem-C-00599C?style=for-the-badge&logo=c&logoColor=white)
![Wokwi](https://img.shields.io/badge/Simulador-Wokwi-1E88E5?style=for-the-badge)

Este repositório contém o código-fonte referente à Atividade 3: Controle de Saída Digital com PWM no ESP32. O projeto foi desenvolvido utilizando o framework ESP-IDF e estruturado para validação e execução no simulador Wokwi.

**Acesse a simulação do circuito:** [Visualizar projeto no Wokwi](https://wokwi.com/projects/459463206313116673)

## Objetivo do Projeto

Demonstrar o controle de atuadores digitais por meio da manipulação de Modulação por Largura de Pulso (PWM). O sistema controla a intensidade luminosa de quatro LEDs e a frequência de um buzzer piezoelétrico de forma independente, utilizando o driver LEDC do microcontrolador ESP32-S3.

## Arquitetura de Hardware

O circuito projetado utiliza os seguintes componentes:
- 1x Microcontrolador ESP32-S3-DevKitC-1
- 4x LEDs (com respectivos resistores limitadores de corrente)
- 1x Buzzer Piezoelétrico

### Mapeamento de Pinos (Pinout)

| Componente | Pino (GPIO) | Canal LEDC | Frequência Base |
| :--- | :--- | :--- | :--- |
| LED 1 | GPIO 4 | Canal 0 | 1 kHz |
| LED 2 | GPIO 5 | Canal 1 | 1 kHz |
| LED 3 | GPIO 6 | Canal 2 | 1 kHz |
| LED 4 | GPIO 7 | Canal 3 | 1 kHz |
| Buzzer | GPIO 15 | Canal 4 | Variável (500 Hz a 2000 Hz) |

## Estrutura Lógica e Funcionamento

A inicialização do firmware garante o estado de repouso (duty cycle em 0%) para todos os atuadores. O laço de repetição principal divide a operação nas seguintes fases:

1. **Fading Sincronizado:** Alteração gradual e simultânea do duty cycle de todos os LEDs (de 0% até a resolução máxima de 13 bits, retornando a 0% em seguida).
2. **Fading Sequencial:** Execução do efeito de transição de brilho de forma isolada, percorrendo iterativamente do LED 1 ao LED 4.
3. **Teste Sonoro de Varredura:** Ativação do buzzer com incrementos graduais de frequência de 500 Hz até 2000 Hz, seguidos por decrementos simétricos até a frequência inicial.

## Como Executar

### Via Simulador Wokwi
1. Acesse o link do projeto no Wokwi fornecido no início deste documento.
2. Inicie a simulação para visualizar o comportamento do código em tempo real diretamente no navegador.

### Via Ambiente Local (ESP-IDF)
1. Clone este repositório para o seu ambiente local de desenvolvimento.
2. Certifique-se de possuir o ESP-IDF configurado e atualizado.
3. Navegue até o diretório raiz do projeto e defina o microcontrolador alvo como ESP32-S3:
   ```bash
   idf.py set-target esp32s3
   ```
   
    Em seguida, execute os comandos padrão de compilação e gravação:
    ```bash
    idf.py build flash monitor
    ```
