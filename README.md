Se sube por este medio el codigo fuente . Para probar el código presione en el archivo .bin y luego en view rar.
Se adjunta el código por este medio.
->
->
->
#include "I2C.h"
#include "ThisThread.h"
#include "mbed.h"
#include "Adafruit_GFX.h"
#include "Adafruit_SSD1306.h" 
#include <cstring>

#define TMP102_ADDRESS 0x90  
#define tiempo_muestreo 1s

BufferedSerial serial(USBTX, USBRX);
I2C i2c(D14, D15);
Adafruit_SSD1306_I2c oled(i2c, D0); 
AnalogIn ain(A0);                  

float Vin = 0.0;
char buffer[40];
char comando[1] = {0x00};  
char data[2];            

void init_oled() {
    oled.begin();
    oled.clearDisplay();
    oled.setTextSize(1);
    oled.setTextColor(1);
    oled.display();
}

void mostrar_oled_serial(const char* mensaje, int len) {
    // Mostrar en OLED
    oled.clearDisplay();
    oled.setTextCursor(0, 2);
    oled.printf(mensaje);
    oled.display();
    serial.write(mensaje, len); 
}

float leer_analogico() {
    return ain.read() * 3.3; 
}

float leer_temperatura() {
    i2c.write(TMP102_ADDRESS, comando, 1);   
    i2c.read(TMP102_ADDRESS, data, 2);      
    int16_t temp = (data[0] << 4) | (data[1] >> 4); 
    return temp * 0.0625; 
}

int main() {
    init_oled();
    const char* msg_inicio = "Arranque del programa\n\r";
    mostrar_oled_serial(msg_inicio, 22); 
    while (true) {
        Vin = leer_analogico();
        int len_voltaje = sprintf(buffer, "Voltaje: %1.4f V\n\r", Vin);
        mostrar_oled_serial(buffer, len_voltaje);
        float temperatura = leer_temperatura();
        int len_temp = sprintf(buffer, "Temp: %2.4f C\n\r", temperatura);
        mostrar_oled_serial(buffer, len_temp);
        ThisThread::sleep_for(tiempo_muestreo); 
    }
}


