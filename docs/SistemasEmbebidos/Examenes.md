# Examenes Del Curso

---

## Primer Parcial: Simón Dice

---

En este parcial, nuestro reto consistió en hacer utilizando todos los conocimientos previos un juego de simón dice en raspberry Pi Pico 2, con las siguientes normas:

1.- La secuencia crece +1 por ronda, de 1 hasta 15.

2.- La persona jugadora debe repetir la secuencia con 4 botones dentro de un tiempo límite por ronda (En nuestro caso se anuló esta regla porque Sebastián quedó en podio del Kahoot).

3.- Tiempo límite por ronda (fase de entrada): TL = longitud + 5 segundos (p. ej., Ronda 7 → 12 s). (No lo aplicamos).

4.- Puntaje (0–15): mostrar la máxima ronda alcanzada en un display de 7 segmentos en hex (0–9, A, b, C, d, E, F).

5.- Aleatoriedad obligatoria: la secuencia debe ser impredecible en cada ejecución.

### Reglas del juego (obligatorias):

1.- Encendido/Reset: el 7 segmentos muestra “0” y queda en espera de Start (cualquier botón permite iniciar).

2.- Reproducción: mostrar la secuencia actual (LEDs uno por uno con separación clara).

3.- Entrada: al terminar la reproducción, la persona debe repetir la secuencia completa dentro de TL.

4.- Fallo (Game Over): botón incorrecto, falta/extra de entradas o exceder TL.

5.- Progresión: si acierta, puntaje = número de ronda, agrega 1 color aleatorio y avanza.

6.- Fin: al fallar o completar la Ronda 15. Mostrar puntaje final en 7 segmentos (hex).

---

### Programa 

---

```bash

#include "pico/stdlib.h"
#include "hardware/adc.h"
#include <stdlib.h>
#include <time.h>

#define Rondas 15
#define Parpadeo 400
#define Pausa 250
#define Debounce 50

#define LED0 0
#define LED1 1
#define LED2 3
#define LED3 4

#define BTN0 27
#define BTN1 28
#define BTN2 14
#define BTN3 15

#define SegmentoA  16
#define SegmentoB  17
#define SegmentoC  18
#define SegmentoD 26
#define SegmentoDp  20
#define SegmentoE  21
#define SegmentoF  22
#define SegmentoG  2

// arrays de pines para recorrer
const uint LEDS[4] = {LED0, LED1, LED2, LED3};
const uint Botones[4] = {BTN0, BTN1, BTN2, BTN3};
const uint Segmentos[8] = {SegmentoA, SegmentoB, SegmentoC, SegmentoDp, SegmentoE, SegmentoF, SegmentoG, SegmentoD};

//Ánodo común, 0=1, 1=0
const bool MapaDisplay[16][8] = {
    {0,0,0,0,0,0,1,1}, // 0
    {1,0,0,1,1,1,1,1}, // 1
    {0,0,1,0,0,1,0,1}, // 2
    {0,0,0,0,1,1,0,1}, // 3
    {1,0,0,1,1,0,0,1}, // 4
    {0,1,0,0,1,0,0,1}, // 5
    {0,1,0,0,0,0,0,1}, // 6
    {0,0,0,1,1,1,1,1}, // 7
    {0,0,0,0,0,0,0,1}, // 8
    {0,0,0,0,1,0,0,1}, // 9
    {0,0,0,1,0,0,0,1}, // A
    {1,1,0,0,0,0,0,1}, // b
    {0,1,1,0,0,0,1,1}, // C
    {1,0,0,0,0,1,0,1}, // d
    {0,1,1,0,0,0,0,1}, // E
    {0,1,1,1,0,0,0,1}  // F
};

uint8_t Sequencia[Rondas];
int Num_sequencia = 0;

void MuestraDisplay(uint8_t n) {
    for (int i = 0; i < 8; i++) {
        gpio_put(Segmentos[i], MapaDisplay[n & 0xF][i]);
    }
}

void Blink(uint8_t iL, uint32_t ms) {
    gpio_put(LEDS[iL], 1);
    sleep_ms(ms);
    gpio_put(LEDS[iL], 0);
}

int PresionaBoton() {
    while (1) {
        for (int i = 0; i < 4; i++) {
            if (!gpio_get(Botones[i])) { 
                sleep_ms(Debounce);
                while (!gpio_get(Botones[i])); 
                return i;
            }
        }
        sleep_ms(10);
    }
}

void EsperarBoton() {
    while (1) {
        for (int i = 0; i < 4; i++) {
            if (!gpio_get(Botones[i])) {
                sleep_ms(Debounce);
                while (!gpio_get(Botones[i])); // espera a quitar el botón presionado
                return;
            }
        }
        sleep_ms(10);
    }
}

void IniciarLeds() {
    for (int i = 0; i < 4; i++) {
        gpio_init(LEDS[i]);
        gpio_set_dir(LEDS[i], true);
    }
}

void IniciarBotones() {
    for (int i = 0; i < 4; i++) {
        gpio_init(Botones[i]);
        gpio_set_dir(Botones[i], false);
        gpio_pull_up(Botones[i]);
    }
}

void IniciarDisplay() {
    for (int i = 0; i < 8; i++) {
        gpio_init(Segmentos[i]);
        gpio_set_dir(Segmentos[i], true);
    }
}

// Reproduce la secuencia actual
void ReproducirSecuencia(int lim) {
    sleep_ms(300);
    for (int i = 0; i < lim; i++) {
        Blink(Sequencia[i], Parpadeo);
        sleep_ms(Pausa);
    }
}

// Game Over
void GameOver(uint8_t score) {
    MuestraDisplay(score > 15 ? 15 : score);

   
    for (int j = 0; j < 6; j++) {
        for (int i = 0; i < 4; i++) gpio_put(LEDS[i], 1);
        sleep_ms(120);
        for (int i = 0; i < 4; i++) gpio_put(LEDS[i], 0);
        sleep_ms(120);
    }

   
    EsperarBoton();

    Num_sequencia = 0;
    MuestraDisplay(0);
}

// Genera un nuevo color aleatorio y lo agrega a la secuencia
void SiguienteRonda() {
    Sequencia[Num_sequencia++] = rand() & 0x3;
    if (Num_sequencia > Rondas) Num_sequencia = Rondas;
}

bool PresionarSecuencia() { //La función que tiene que hacer el jugador físicamente
    for (int i = 0; i < Num_sequencia; i++) {
        int presionar = PresionaBoton();
        Blink(presionar, 120);
        if (presionar != Sequencia[i]) return false;
    }
    return true;
}



int main() {
    stdio_init_all();
    IniciarLeds();
    IniciarBotones();
    IniciarDisplay();

    // Aleatoriedad: ADC + tiempo
    adc_init();
    adc_gpio_init(26);
    adc_select_input(0);
    uint16_t noise = adc_read();
    srand(to_us_since_boot(get_absolute_time()) ^ noise);

    MuestraDisplay(0);

    while (1) {
        
        EsperarBoton();

        while (1) {
            SiguienteRonda();
            MuestraDisplay(Num_sequencia);
            sleep_ms(400);
            ReproducirSecuencia(Num_sequencia);

            bool Correcto = PresionarSecuencia();

            if (!Correcto) {
                GameOver(Num_sequencia - 1);
                break; // reinicia juego
            }

            if (Num_sequencia >= Rondas) {
                GameOver(Rondas);
                break; // reinicia juego
            }
        }
    }
}

```

---

### Diagrama y video

![Diagrama Parcial1](IMGSTareas/IMG/Diagramasimondice.png){ width="600" align=center}

<iframe width="560" height="315"
src="https://www.youtube.com/embed/e2gSABZMsjI"
title="YouTube video player"
frameborder="0"
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
allowfullscreen>
</iframe>

---

