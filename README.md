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



### Montagem Breadboard
![Image_1737904374920](https://github.com/user-attachments/assets/3936dc31-deb7-4c46-873b-30f348384df9)

# Sketch
Consulte:

