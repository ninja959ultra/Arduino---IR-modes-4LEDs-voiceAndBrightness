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

  else if (brightnessValue > 500 && voiceValue < 533) {  // High brightness mode
    turnOffLEDs(blueLED, redLED, greenLED);
    digitalWrite(whiteLED, HIGH);
  }

  else if (brightnessValue > 540 && brightnessValue < 580 && voiceValue > 530) {
    turnOffLEDs(redLED, whiteLED, greenLED);
    alert(blueLED);
  }

  else {
    turnOffLEDs(redLED, blueLED, whiteLED);
    digitalWrite(greenLED, HIGH);
  }
}


void manualMode() {

  Serial.println("IN manual");

  if (signal == 71 && ir.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT) {
    digitalWrite(buzzer, HIGH);
  }

  else{
    digitalWrite(buzzer, LOW);
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
    manualMode();
  }

  if (autoState) {
    autoMode();
  }

  Serial.print("bright= ");
  Serial.println(brightnessValue);
  Serial.print("voice= ");
  Serial.println(voiceValue);

  delay(300);
}

```
