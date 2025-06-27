# Arduino---IR-modes-4LEDs-voiceAndBrightness



# Description:
This project is an advanced Arduino-based control system that integrates a sound sensor, a light sensor (LDR), an IR remote, and a buzzer to respond intelligently to environmental conditions. The system supports two main modes: Auto Mode and Manual Mode.

---

Auto Mode ( button 1 )

In Auto Mode, the system reacts based on combinations of light and sound levels. It includes multiple smart conditions:

Condition 1 – Low light + Loud sound:
If the room is dark (light level is below a defined threshold) and a loud sound is detected, the red LED turns ON and a buzzer alarm is triggered.

Condition 2 – High light + No sound:
If the light level is above normal (too bright) and no sound is detected, the white LED turns ON without any alarm.

Condition 3 – Normal light (between 540 and 600) + Loud sound:
If light levels are within the normal range and a loud sound occurs, the blue LED and the buzzer are activated as an alert.

Any other condition:
If none of the above conditions are met, the system turns ON the green LED to indicate that everything is normal.

---

Manual Mode or emergency Mode( button 2 )

Activated by pressing a button on the IR remote, Manual Mode disables all sensors.
In this mode, pressing the designated remote button will manually trigger the buzzer for testing or warning purposes.
Switching back to Auto Mode re-enables sensor monitoring.


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

// Times
unsigned long lastTimeBuzzer = 0;
const int timeOut = 300;


// global variables
int brightnessValue;
int voiceValue;
int signal;
byte count;



// Status
bool buzzerState = false;
bool autoState = false;
bool manualState = false;
bool isRepeat;

void turnOffLEDs(byte LED1, byte LED2, byte LED3) {
  digitalWrite(LED1, LOW);
  digitalWrite(LED2, LOW);
  digitalWrite(LED3, LOW);
}


void alert(byte LED){
  count = 0;

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

  Serial.println("in auto");

  if (brightnessValue < 250 && voiceValue > 530){ // Darkness mode
    turnOffLEDs(blueLED, whiteLED, greenLED);
    digitalWrite(redLED, HIGH);
    alert(redLED);
  }

  else if (brightnessValue > 620 && voiceValue < 533) {  // High brightness mode
    turnOffLEDs(blueLED, redLED, greenLED);
    digitalWrite(whiteLED, HIGH);
  }

  else if (brightnessValue > 540 && brightnessValue < 600 && voiceValue > 533) {
    turnOffLEDs(redLED, whiteLED, greenLED);
    alert(blueLED);
  }

  else {
    turnOffLEDs(redLED, blueLED, whiteLED);
    digitalWrite(greenLED, HIGH);
  }
}



void setup() {
  Serial.begin(9600);
  ir.enableIRIn();
  pinMode(redLED, OUTPUT);
  pinMode(blueLED, OUTPUT);
  pinMode(whiteLED, OUTPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(voice, INPUT);
}


void loop() {

  if (ir.decode()){
    signal = ir.decodedIRData.command;
    isRepeat = ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT;
    
    ir.resume();
  }

  if (signal == 69 && !(ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT)){
    autoState = true;
    manualState = false;    
  }

  else if (signal == 70 && !(ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT)){
    autoState = false;
    manualState = true;
  }

  if (manualState){
    digitalWrite(buzzer, HIGH);
  }

  else{
    digitalWrite(buzzer, LOW);
  }

  if (autoState) {
    autoMode();
  }

  Serial.print("bright= ");
  Serial.println(brightnessValue);
  Serial.print("voice= ");
  Serial.println(voiceValue);
  Serial.println("Manual state= ");
  Serial.println(manualState);

  delay(300);
}

```
