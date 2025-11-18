
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

### Giro del Ultrasonidos cuando detecta un objeto cerca
    #include <Servo.h>
    
    Servo servo;
    const int PIN_SERVO = 10;
    
    int trigPin = 13;
    int echoPin = 12;
    float duration, distance;
    float umbral = 20;
    int rotacionServo = 90;
    int direccionServo = 1;
    void setup() {
      pinMode(trigPin, OUTPUT);
      pinMode(echoPin, INPUT);
      servo.attach(PIN_SERVO); // D9
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
    while(distance < umbral)
    {
    
    if(rotacionServo >= 180){
      direccionServo = -1;
    }
    else if(rotacionServo <= 0){
      direccionServo = 1;
    }
    rotacionServo += (5 * direccionServo); 
    servo.write(rotacionServo);
    delay(150);
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
    
    }
  

  
}


