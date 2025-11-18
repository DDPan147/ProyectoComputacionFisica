
# Project Title

A brief description of what this project does and who it's for


## Código
### Servo
#include <Servo.h>

Servo servo;
const int PIN_SERVO = 10;

void setup() {
  servo.attach(PIN_SERVO); // D9
}

void loop() {
 for(int i  = 0; i<=180; i+=20){
   servo.write(i);
   delay(150);
 }
 for(int i  = 180; i>=0; i-=20){
   servo.write(i);
   delay(150);
 }
}
### Ultrasonidos
int trigPin = 13;
int echoPin = 12;
float duration, distance;
float umbral = 20;

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  Serial.begin(9600);

}

void loop() {
  
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);
  distance = (duration*.0343)/2;
  Serial.print("Distance: ");

  
  delay(100);
  Serial.println(distance);
  if(distance < umbral){
      Serial.print(" Muy cerca ");
  }
  else{
    Serial.print(" Lejos ");
  }
}

