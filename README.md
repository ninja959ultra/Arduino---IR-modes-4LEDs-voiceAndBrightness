# Arduino---IR-modes-4LEDs-voiceAndBrightness


# Code:
```cpp
#include <IRremote.hpp>

IRrecv ir(2);


// Pins
byte redLED = 8;
byte whiteLED = 9;
byte blueLED = 10;
byte greenLED = 11;
byte buzzer = 6;
byte voice = A5;
byte brightness = A0;


// glbal variables
int brightnessValue;
int voiceValue;
int signal;
byte count;


void turnOffLEDs(byte LED1, byte LED2) {
  digitalWrite(LED1, LOW);
  digitalWrite(LED2, LOW);
}


void alert(LED){
  count = 0

  do{
    digitalWrite(LED, LOW);
    digitalWrite(buzzer, HIGH);
    delay(300);
    digitalWrite(LED, HIGH);
    digitalWrite(buzzer, LOW);
    delay(300);

    count++;
  }

  while (count < 3);
}


void autoMode() {
  brightnessValue = analogRead(brightness);
  voiceValue = analogRead(voice);

  if (brigtnessValue < 250 && voiceValue > 523){ // Darkness mode
    turnOffLEDs(redLED, whiteLED);
    alert(redLED);
  }

  else if (brightnessValue > 500 && voiceValue < 520) {  // Rest mode
    turnOffLEDs(blueLED, redLED);
    digitalWrite(whiteLED, LOW);
  }

  else if (brightnessValue > 300 && brightnessValue < 500 && voiceValue > 523) {
    turnOffLEDs(redLED, whiteLED);
    alert(blueLED);
  }

}



void setup() {
  ir.enableIRIn();
  pinMode(redLED, OUTPUT);
  pinMode(blueLED, OUTPUT);
  pinMode(whiteLED, OUTPUT);
  pinMode(voice, INPUT);
}


void loop() {
  signal = ir.decodedIRData.command;
}

```
