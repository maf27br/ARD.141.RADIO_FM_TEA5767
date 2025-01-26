# Projeto Radio FM com TEA5767
Montagem de radio usando módulo TEA5767, RTC 1307 e encoder KY-040.

Os deasafios deste projeto foram lidar com a alta captação de ruido pelo TEA5767 nas linhas I2C, mesmo com resistores pull-up, ajuste do rotary encoder para a operação na banda FM brasileira e mecanismo de troca de estações mais "adequado" a realidade, com a separação de 200KHz.
O melhor ajuste acabou sendo com a biblioteca GyverOLED (mesmo sendo exemplos em russo) que possui menos consumo em refresh, atualização display stand-by a cada 60s e resisitores pull-up. O resultado é que o ruído (estática) ficou somente a cada minuto e ao trocar de estação.

## Lista de material utilizado:
- Arduino Nano
- display oled 128x64, SSD1306
- Tiny RTC 1307 com AT24C32 (eeprom)
- 2 x 4,7KΩ 
- TEA 5767 (módulo FM)
- Rotary encoder KY-040
- jumpers
  
## Prototipação
Nesta primeira etapa, foi feita a montagem em breadboard, para testes e desenvolvimento do sketch.

### Esquema (fritzing)
![RADIO TEA5767_bb](https://github.com/user-attachments/assets/57de7a1d-037c-4be7-bd59-4c2d222fa449)

## Sketch
#include <Wire.h>
#include <radio.h>
#include <TEA5767.h>
#include <GyverOLED.h>
#include "RTClib.h"
#include <RotaryEncoder.h>

// inicia estrutura de tempo (t)
RTC_DS1307 rtc;
const static char *DiasdaSemana[] = { "DOM", "SEG", "TER", "QUA", "QUI", "SEX", "SAB" };
DateTime now;
unsigned long tanterior_RTC;// declara variável de atualização
//encoder
RotaryEncoder encoder(A2, A3, RotaryEncoder::LatchMode::TWO03); //encoder acelerado
int nowPos, newPos, startPos = 0;
//display
GyverOLED<SSD1306_128x64, OLED_NO_BUFFER> oled;
//radio
#define FIX_BAND RADIO_BAND_FM
double frequencia = 87.5;
TEA5767 radio;
RADIO_INFO info;
char buf[20];

void setup() {
  Wire.begin();
  Serial.begin(9600);
  
  //Display
  oled.init();
  
  //RTC
  if (!rtc.begin()) {  
    Serial.println("DS1307 não encontrado");
    while (1);
  }
  if (!rtc.isrunning()) {               
    Serial.println("DS1307 rodando!");
    //rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));  //CAPTURA A DATA E HORA EM QUE O SKETCH É COMPILADO
  }
  now = rtc.now();
  
  //verifica ultima estação sintonizada (Rotary Encoder). Pega valor da NVRAM e atualiza frequencia
  Serial.print("li no setup: ");Serial.println(rtc.readnvram(0));
  startPos = rtc.readnvram(0);
  nowPos = startPos+newPos;
  frequencia = 87.5 + ((double) nowPos * (0.2));
  
  // Inicializa o radio
  radio.init();
  radio.setBandFrequency(FIX_BAND,(int)(frequencia*100));
  radio.getRadioInfo(&info);
  refreshDisplay();
}

void loop() {
  //Atualiza RTC - 1s
  unsigned long atual = millis();
  if (atual - tanterior_RTC > 60000) {
    now = rtc.now();
    tanterior_RTC = atual;
    refreshDisplay();
  }
  //static int pos = 0;
  encoder.tick();
  newPos = encoder.getPosition();
  if (nowPos != startPos+newPos) {
    nowPos = startPos+newPos;
    frequencia = 87.5 + ((double)nowPos * (0.2));
    Serial.print("Valor pos: ");Serial.println(newPos);
    Serial.print("Freq: ");Serial.print(frequencia);Serial.println(" MHz");
    radio.setBandFrequency(FIX_BAND,(int)(frequencia*100)); 
    radio.getRadioInfo(&info);
    Serial.print("Sinal: ");Serial.println(info.rssi);
    Serial.print("stereo: ");Serial.println(info.stereo);
    Serial.print("mono: ");Serial.println(info.mono);
    Serial.print("tuned: ");Serial.println(info.tuned);    
    Serial.print("active: ");Serial.println(info.active); 
    //radio.debugRadioInfo();
    refreshDisplay();
    rtc.writenvram(0, (byte)nowPos);
    Serial.print("gravei: ");Serial.println(nowPos);
    Serial.print("li: ");Serial.println(rtc.readnvram(0)); 
  }
}

void refreshDisplay(void) {  //função que irá atualizar o display, na inicialização, loop e mudança de estação
  //iniciliza
  oled.clear();
  oled.home();
  //linha de tempo
  oled.setScale(1);
  oled.setCursorXY(0, 0);
  sprintf(buf, "%02d/%02d/%02d  %s   %02d:%02d", now.day(), now.month(), now.year() - 2000, DiasdaSemana[now.dayOfTheWeek()], now.hour(), now.minute());
  oled.print(buf);
  oled.fastLineH(10, 0, 128);
  oled.fastLineH(48, 0, 128);
  //status
  oled.fastLineH(31, 102, 126);
  //oled.fastLineV(126, 31, 16);
  for (int x = 1; x <= info.rssi ; x++) {
    for (int i = 1; i<=8; i++) {
    oled.fastLineV(104+(x+i), 31, 31-x);  
    }
  }
  oled.setCursorXY(114, 38);
  if (info.stereo) {
    oled.print("ST");
  }else {
    oled.print("MO");
  }
  //Frequencia selecionada
  oled.setScale(3);
  oled.setCursorXY(0, 20);
  dtostrf(frequencia, 3, 1, buf);
  oled.print(buf);
  oled.update();
}


### Montagem Breadboard
![Image_1737904374920](https://github.com/user-attachments/assets/3936dc31-deb7-4c46-873b-30f348384df9)

# Sketch
Consulte:

