# Practica_7

## Código_7.1_Altavoz
```cpp
#include <Arduino.h>
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"
AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;
void setup(){
Serial.begin(115200);
in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
aac = new AudioGeneratorAAC();
out = new AudioOutputI2S();
out -> SetGain(5);
out -> SetPinout(26,25,22);
aac->begin(in, out);
}

void loop(){
if (aac->isRunning()) {
aac->loop();
} else {
aac -> stop();
Serial.printf("Sound Generator\n");
delay(1000);
}
}
```
## Funcionamiento
Para la práctica necesitamos la librería ESP8266Audio que usaremos para generar el sonido y hacer la comunicación I2S. Además de esta, también debemos añadir unas librerías extra debido a la dependencia que tienen unas con otras y la ESP8266Audio mencionada anteriormente.

Una vez en el loop inicializamos la comunicación serie. Declaramos la variable in que llevará el archivo de audio guardado en el fichero de tipo sampleaac.h.  aac se encargará de empezar la comunicación entre el decodificador y nuestra placa. Para acabar, out nos da la opción de modificar configuraciones como la ganancia, los pines de salida de la ESP32, etc.

Finalmente en cuanto al loop, las tres funciones principales que usamos son de la clase AudioGenerator ACC: isRunning(), loop() y stop(). La variable isRunning permite confirmar al sistema que no existe ningún error y el fichero de audio se a recibido. Si esta es true, se llama al loop() que reproduce la muestras del fichero de audio y si es false se llama a stop() para cerrar el fichero.

## Código_7.2_Reproducción_SD
```cpp
#include "Audio.h"
#include "SD.h"
#include "FS.h"
// Digital I/O used
#define SD_CS 5
#define SPI_MOSI 23
#define SPI_MISO 19
#define SPI_SCK 18
#define I2S_DOUT 22
#define I2S_BCLK 26
#define I2S_LRC 25
Audio audio;
void setup(){
pinMode(SD_CS, OUTPUT);
digitalWrite(SD_CS, HIGH);
SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI);
Serial.begin(115200);
SD.begin(SD_CS);
audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
audio.setVolume(21); // 0...21
audio.connecttoFS(SD, "prueba.wav");
}

void loop(){
audio.loop();
}
// optional
void audio_info(const char *info){
Serial.print("info "); Serial.println(info);
}
void audio_id3data(const char *info){ //id3 metadata
Serial.print("id3data ");Serial.println(info);
}
void audio_eof_mp3(const char *info){ //end of file
Serial.print("eof_mp3 ");Serial.println(info);
}
void audio_showstation(const char *info){
Serial.print("station ");Serial.println(info);
}
void audio_showstreaminfo(const char *info){
Serial.print("streaminfo ");Serial.println(info);
}
void audio_showstreamtitle(const char *info){
Serial.print("streamtitle ");Serial.println(info);
}
void audio_bitrate(const char *info){
Serial.print("bitrate ");Serial.println(info);
}
void audio_commercial(const char *info){ //duration in sec
Serial.print("commercial ");Serial.println(info);
}
void audio_icyurl(const char *info){ //homepage
Serial.print("icyurl ");Serial.println(info);
}
void audio_lasthost(const char *info){ //stream URL played
Serial.print("lasthost ");Serial.println(info);
}
void audio_eof_speech(const char *info){
Serial.print("eof_speech ");Serial.println(info);
}
```
## Funcionamiento
En este ejercicio el objetivo es reproducir un archivo .wav situado en una SD externa. Para esto vamos a usar dos tipos de buses: SPI e I2S, para los que necesitaremos también tres librerías distintas: Audio.h, SD.h, FS.h. Antes de entrar en el setup() lo único que declaramos son los pines necesarios para conectarnos con la SD(SPI_MOSI,SPI_MISO,SPI_SCK,SPI_CS) y los pines necesarios para el amplificador MAX98357A (2S_DOUT, I2S_BCLK, I2S_LRC).

Justo antes del setup() creamos el objeto Audio que nos servirá para configurar pines, volumen, etc. Ya dentro del setup() inicializamos el puerto serie y se establece el pin CS de la SD como salida. Con SPI.begin() configuramos los pines para la recepción de datos se haga desde la SD e inicializamos también la comunicación. Con el bus SPI ya funcionando, empezamos iniciando la función SD.begin() y configurando la comunicación I2S con la que enviaremos el audio leído de la SD.  Se establecen los pines de salida con audio.setPinout() y se puede cambiar el volumen con audio.setVolume(). Por último solo necesitamos usar la función audio.connectorFS() para marcar que fichero vamos a leer de la SD. Pasamos directamente el objeto SD y título de la canción a reproducir.

Después de todos los preparativos, ya solo queda usar la función loop() de la librería audio que nos hace un bucle del fichero para reproducirlo en el void loop(). Finalmente, también se muestra información del sistema por el puerto serie al final del código.




