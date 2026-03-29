#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/ledc.h"

#define LEDC_TIMER              LEDC_TIMER_0
#define LEDC_MODE               LEDC_LOW_SPEED_MODE
#define LEDC_DUTY_RES           LEDC_TIMER_13_BIT 
#define LEDC_FREQUENCY          (1000) 

#define PIN_LED1 4
#define PIN_LED2 5
#define PIN_LED3 6
#define PIN_LED4 7
#define PIN_BUZZER 15 

//  Mapeia a porcentagem (0-100%) para a resolução de 13 bits do duty cycle (0-8191).
#define PERCENT_TO_DUTY(p) ((p * 8191) / 100)
#define MIN 0
#define MAX 100

#define DELAY_MS 500
#define DELAY_BUZZER_MS 200

#define BUZZER_CHANNEL          LEDC_CHANNEL_4
#define BUZZER_TIMER            LEDC_TIMER_1

const int led_gpios[] = {PIN_LED1, PIN_LED2, PIN_LED3, PIN_LED4, PIN_BUZZER};
const ledc_channel_t led_channels[4] = {
    LEDC_CHANNEL_0,
    LEDC_CHANNEL_1,
    LEDC_CHANNEL_2,
    LEDC_CHANNEL_3
};

void init_pwm(){

    ledc_timer_config_t ledc_timer = {
        .speed_mode       = LEDC_MODE,
        .timer_num        = LEDC_TIMER,
        .duty_resolution  = LEDC_DUTY_RES,
        .freq_hz          = LEDC_FREQUENCY,
        .clk_cfg          = LEDC_AUTO_CLK
    };
    ledc_timer_config(&ledc_timer);

    ledc_timer_config_t buzzer_timer = {
        .speed_mode = LEDC_MODE,
        .timer_num = BUZZER_TIMER,
        .duty_resolution = LEDC_DUTY_RES,
        .freq_hz = 1000, 
        .clk_cfg = LEDC_AUTO_CLK
    };
    ledc_timer_config(&buzzer_timer);

    for (int i = 0; i < 4; i++) {
        ledc_channel_config_t ledc_channel = {
            .speed_mode     = LEDC_MODE,
            .channel        = led_channels[i],
            .timer_sel      = LEDC_TIMER,
            .intr_type      = LEDC_INTR_DISABLE,
            .gpio_num       = led_gpios[i],
            .duty           = 0,
            .hpoint         = 0
        };
        ledc_channel_config(&ledc_channel);
        ledc_update_duty(LEDC_MODE, led_channels[i]);
    }

    ledc_channel_config_t bz_config = {
        .speed_mode = LEDC_MODE,
        .channel = BUZZER_CHANNEL,
        .timer_sel = BUZZER_TIMER,
        .gpio_num = PIN_BUZZER,
        .duty = 0
    };
    ledc_channel_config(&bz_config);
}

void buzzer_tone(uint32_t freq_hz, uint32_t volume_percent) {
    if (freq_hz > 0) {
        ledc_set_freq(LEDC_MODE, BUZZER_TIMER, freq_hz);
        ledc_set_duty(LEDC_MODE, BUZZER_CHANNEL, PERCENT_TO_DUTY(volume_percent));
    } else {
        ledc_set_duty(LEDC_MODE, BUZZER_CHANNEL, 0);
    }
    ledc_update_duty(LEDC_MODE, BUZZER_CHANNEL);
}

void set_all_leds_duty(uint32_t duty_value) {
    for (int i = 0; i < 4; i++) {
        ledc_set_duty(LEDC_MODE, led_channels[i], PERCENT_TO_DUTY(duty_value));
        ledc_update_duty(LEDC_MODE, led_channels[i]);
    }
}

void set_led_duty(uint8_t channel, uint32_t duty_value){
    ledc_set_duty(LEDC_MODE, led_channels[channel], PERCENT_TO_DUTY(duty_value));
    ledc_update_duty(LEDC_MODE, led_channels[channel]);
}

void blink_led(uint8_t channel){
    for (uint16_t i = MIN; i <= MAX; i++){
        set_led_duty(channel, i);
        vTaskDelay(pdMS_TO_TICKS(DELAY_MS));
    }
    for (int16_t i = MAX; i >= MIN; i--){
        set_led_duty(channel, i);
        vTaskDelay(pdMS_TO_TICKS(DELAY_MS));
    }
}

void blink_sync_leds(){
    for (uint16_t i = MIN; i <= MAX; i++){
        set_all_leds_duty(i);
        vTaskDelay(pdMS_TO_TICKS(DELAY_MS));
    }
    for (int16_t i = MAX; i >= MIN; i--){
        set_all_leds_duty(i);
        vTaskDelay(pdMS_TO_TICKS(DELAY_MS));
    }
}

void vary_buzzer() {
    for (int f = 500; f <= 2000; f += 50) {
        buzzer_tone(f, 50); 
        vTaskDelay(pdMS_TO_TICKS(DELAY_BUZZER_MS));
    }
    for (int f = 2000; f >= 500; f -= 50) {
        buzzer_tone(f, 50);
        vTaskDelay(pdMS_TO_TICKS(DELAY_BUZZER_MS));
    }
    buzzer_tone(0, 0);
}

void app_main(void)
{
    init_pwm();
    vTaskDelay(pdMS_TO_TICKS(2000)); //garantindo que vai inicializar corretamente
    while(1){
        blink_sync_leds();
        for(int i = 0; i < 4; i++){          
            blink_led(led_channels[i]);
        }
        vary_buzzer();
    }
}
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
3. Navegue até o diretório raiz do projeto e execute os comandos padrão de compilação e gravação:
   ```bash
   idf.py build flash monitor