# ESP32 I2S Audio Pass-Through: INMP441 a MAX98357A

Aquest repositori conté un codi per a ESP32 (escrit en C++ per a l'entorn Arduino/PlatformIO) que implementa un sistema de pass-through d'àudio en temps real. El sistema llegeix l'àudio d'un micròfon digital I2S i l'envia directament a un amplificador d'àudio I2S.

## 📝 Descripció del Projecte

El programa configura dos perifèrics I2S independents a l'ESP32:
*   **Port I2S 1 (RX):** Connectat al micròfon omnidireccional **INMP441**.
*   **Port I2S 0 (TX):** Connectat a l'amplificador d'àudio **MAX98357A**.

L'àudio es captura a una freqüència de mostreig de **16,000 Hz**. Atès que el micròfon INMP441 envia dades en format de 32 bits i l'amplificador espera rebre dades de 16 bits, el bucle principal (`loop`) s'encarrega de fer la conversió de format en temps real, aplicant un desplaçament de bits (bit shifting) amb una funció de "clamping" per evitar distorsions per *overflow*.

## 🛠 Hardware Requerit

*   Placa ESP32 (compatible amb els pins indicats o modificable).
*   Micròfon I2S INMP441.
*   Amplificador/Decodificador I2S MAX98357A.
*   Altaveu compatible amb l'amplificador.
*   Cables de connexió (Jumpers).

## 🔌 Esquema de Connexions (Pinout)

Assegura't de connectar correctament els pins de l'ESP32 als mòduls I2S:

### Micròfon INMP441 (Entrada d'àudio)
| INMP441 Pin | ESP32 GPIO | Descripció |
| :--- | :--- | :--- |
| **SCK** | GPIO 6 | Serial Clock |
| **WS** | GPIO 7 | Word Select (LRCK) |
| **SD** | GPIO 8 | Serial Data |
| **L/R** | GND | Selecció de canal (GND = Esquerre) |
| **VDD** | 3.3V | Alimentació |
| **GND** | GND | Terra |

### Amplificador MAX98357A (Sortida d'àudio)
| MAX98357A Pin | ESP32 GPIO | Descripció |
| :--- | :--- | :--- |
| **BCLK** | GPIO 1 | Bit Clock |
| **LRC** | GPIO 2 | Left/Right Clock (Word Select) |
| **DIN** | GPIO 4 | Data In |
| **VIN** | 5V o 3.3V| Alimentació (segons la placa) |
| **GND** | GND | Terra |

*(Nota: Si fas servir una placa ESP32 clàssica on els pins 1, 2, 6, 7 i 8 s'utilitzen per altres funcions internes com la memòria flaix o el port sèrie RX/TX, és molt recomanable que reassignis aquests pins a la capçalera del codi per evitar conflictes).*

## 🚀 Instal·lació i Ús

1.  Obre l'Arduino IDE o PlatformIO.
2.  Assegura't de tenir instal·lat el paquet de plaques per a **ESP32** d'Espressif.
3.  Copia i enganxa el codi al teu fitxer `main.cpp` o `.ino`.
4.  Si fas servir pins diferents, modifica les directives `#define` a la part superior del codi.
5.  Connecta la placa ESP32 i compila/puja el codi.
6.  Obre el Monitor Sèrie a **115200 bauds**. 
7.  Hauries de veure el missatge: `Iniciando pass-through micro → altavoz...` seguit de `Listo. Habla por el micrófono.`.
8.  Pots parlar pel micròfon i escoltar-te immediatament per l'altaveu.

## 🧠 Com funciona el processament de l'àudio

Dins de la funció `loop()`, succeeix el següent:
1.  **Lectura:** L'ESP32 llegeix un bloc (buffer) de mostres de 32 bits des del micròfon.
2.  **Conversió:** S'itera sobre cada mostra. Com que les dades útils del INMP441 es troben als bits superiors, es desplaça el valor cap a la dreta (`>> 11`).
3.  **Clamping:** Es limita el valor resultant entre `-32768` i `32767` per assegurar-se que encaixa perfectament en una variable de 16 bits (`int16_t`) sense desbordar-se (el qual causaria un soroll molt fort i desagradable).
4.  **Escriptura:** Les noves dades de 16 bits s'envien al port de l'amplificador per ser reproduïdes.
