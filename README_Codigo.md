
## Código
### Prototipo 1
    //    The direction of the car's movement
    //  ENA   ENB   IN1   IN2   IN3   IN4   Description  
    //  HIGH  HIGH  HIGH  LOW   LOW   HIGH  Car is runing forward
    //  HIGH  HIGH  LOW   HIGH  HIGH  LOW   Car is runing back
    //  HIGH  HIGH  LOW   HIGH  LOW   HIGH  Car is turning left
    //  HIGH  HIGH  HIGH  LOW   HIGH  LOW   Car is turning right
    //  HIGH  HIGH  LOW   LOW   LOW   LOW   Car is stoped
    //  HIGH  HIGH  HIGH  HIGH  HIGH  HIGH  Car is stoped
    //  LOW   LOW   N/A   N/A   N/A   N/A   Car is stoped
    
    // Pines correctos para Elegoo Smart Car V1.1
    #define PWMA 5
    #define PWMB 6
    #define AIN1 7
    #define AIN2 11
    #define BIN1 8
    #define BIN2 13
    #define STBY 3
    #define TimeLapse 1000
    #define Speed 125
    
    int trigPin = 13;
    int echoPin = 12;
    float duration, distance;
    float umbral = 30;
    int tiempoDeGiro = 450;
    bool movement=true;
    
    #include <Servo.h>
    
    Servo servo;
    const int PIN_SERVO = 10;
    
    #include <IRremote.h>
    const int PIr = 9;
    int comando;
    int lastCommand;
    
    bool isPressed;
    bool isManual;
    bool canChange;
    
    void setup() {
      Serial.begin(9600);
    
      pinMode(PWMA, OUTPUT);
      pinMode(PWMB, OUTPUT);
      pinMode(AIN1, OUTPUT);
      pinMode(AIN2, OUTPUT);
      pinMode(BIN1, OUTPUT);
      pinMode(BIN2, OUTPUT);
      pinMode(STBY, OUTPUT);
    
      digitalWrite(STBY, HIGH); 
    
      servo.attach(PIN_SERVO); // D9
      servo.write(90);
    
      pinMode(trigPin, OUTPUT);
      pinMode(echoPin, INPUT);
    
      IrReceiver.begin(PIr); 
    }
    
    void loop() { 
      if(IrReceiver.decode()){
        Serial.print("Codigo recibido: ");
    	  comando = IrReceiver.decodedIRData.command;
        Serial.println(comando);
        if(comando == 67){
          if(canChange){
           isManual=!isManual;
           canChange=false;
           stop();
          }
          else
          {
            delay(1000);
            canChange=true;
          }
    
        }
        
        IrReceiver.resume();
      }
      if(isManual){
        ManualMode();
      }
      else{
        AutomaticMode();
      }
      Serial.println(isManual);
      
      
      
    }
    
    // void ManualMode(){
    //   if(IrReceiver.decode()){
    //     //Serial.print("Codigo recibido: ");
    // 	  comando = IrReceiver.decodedIRData.command;
    //     switch(comando){
    //       case 24:
    //       forward();
    //       break;
    //       case 8:
    //       left();
    //       break;
    //       case 90:
    //       right();
    //       break;
    //       case 82:
    //       back();
    //       break;
    //       case 28:
    //       stop();
    //     }
    //     IrReceiver.resume();
    //   }
    // }
    void ManualMode(){
      switch(comando){
          case 24:
          forward();
          break;
          case 90:
          left();
          break;
          case 8:
          right();
          break;
          case 82:
          back();
          break;
          case 28:
          stop();
        }
    }
    void AutomaticMode(){
      float distanciaIzq;
      float distanciaDer;
       if(!UltrasonidosLogic()&&movement){
        forward();
      }
      else{
        stop();
        delay(777);
        GirarServoDerecha();
        distanciaDer = CuantaDistancia();
        GirarServoIzquierda();
        distanciaIzq = CuantaDistancia();
        if(EsMasGrandeLaDerecha(distanciaDer, distanciaIzq)){
          right();
          delay(tiempoDeGiro);
        }
        else{
          left();
          delay(tiempoDeGiro);
        }
        servo.write(90);
        delay(200);
        movement=true;
      }
      
    }
    
    bool EsMasGrandeLaDerecha(float derecha, float izquierda){
      if(derecha >= izquierda){
        return true;
      }
      else{
        return false;
      }
    }
    
    void GirarServoDerecha(){
      for(int i = 90; i <=180; i+=45){
        servo.write(i);
        delay(100);
      }
    }
    
    void GirarServoIzquierda(){
      for(int i = 90; i >=0; i-=45){
        servo.write(i);
        delay(100);
      }
    }
    float CuantaDistancia(){
      float durationXd;
      float distanceXd;
      digitalWrite(trigPin, LOW);
      delayMicroseconds(2);
      digitalWrite(trigPin, HIGH);
      delayMicroseconds(10);
      digitalWrite(trigPin, LOW);
    
      durationXd = pulseIn(echoPin, HIGH);
      distanceXd = (durationXd*.0343)/2;
      //Serial.print("Distance: ");
    
      
      delay(100);
      return distanceXd;
    }
    
    bool UltrasonidosLogic(){
      digitalWrite(trigPin, LOW);
      delayMicroseconds(2);
      digitalWrite(trigPin, HIGH);
      delayMicroseconds(10);
      digitalWrite(trigPin, LOW);
    
      duration = pulseIn(echoPin, HIGH);
      distance = (duration*.0343)/2;
      //Serial.print("Distance: ");
    
      
      delay(100);
      //Serial.println(distance);
      if(distance < umbral){
          //Serial.print(" Muy cerca ");
          movement=false;
          return true;
      }
      else{
        //Serial.print(" Lejos ");
        return false;
      }
    }
    void forward() {
      digitalWrite(STBY, HIGH);
      
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Forward");
    }
    
    void back() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Back");
    }
    
    void left() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Left");
    }
    
    void right() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Right");
    }
    
    void stop() {
      digitalWrite(PWMA, 0);
      digitalWrite(PWMB, 0);
      digitalWrite(STBY, LOW);
    
      Serial.println("Stop");
    }


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
### Motores
    //    The direction of the car's movement
    //  ENA   ENB   IN1   IN2   IN3   IN4   Description  
    //  HIGH  HIGH  HIGH  LOW   LOW   HIGH  Car is runing forward
    //  HIGH  HIGH  LOW   HIGH  HIGH  LOW   Car is runing back
    //  HIGH  HIGH  LOW   HIGH  LOW   HIGH  Car is turning left
    //  HIGH  HIGH  HIGH  LOW   HIGH  LOW   Car is turning right
    //  HIGH  HIGH  LOW   LOW   LOW   LOW   Car is stoped
    //  HIGH  HIGH  HIGH  HIGH  HIGH  HIGH  Car is stoped
    //  LOW   LOW   N/A   N/A   N/A   N/A   Car is stoped
    
    // Pines correctos para Elegoo Smart Car V1.1
    #define PWMA 5
    #define PWMB 6
    #define AIN1 8
    #define AIN2 11
    #define BIN1 12
    #define BIN2 13
    #define STBY 7
    
    void setup() {
      Serial.begin(9600);
    
      pinMode(PWMA, OUTPUT);
      pinMode(PWMB, OUTPUT);
      pinMode(AIN1, OUTPUT);
      pinMode(AIN2, OUTPUT);
      pinMode(BIN1, OUTPUT);
      pinMode(BIN2, OUTPUT);
      pinMode(STBY, OUTPUT);
    
      digitalWrite(STBY, HIGH); // ¡IMPORTANTE!
    }
    
    void loop() { forward(); delay(10000); back(); delay(10000); right(); delay(10000); left(); delay(10000); stop(); delay(20000); }
    
    void forward() {
      digitalWrite(STBY, HIGH);
      
      analogWrite(PWMA, 255);
      analogWrite(PWMB, 255);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Forward");
    }
    
    void back() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, 255);
      analogWrite(PWMB, 255);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Back");
    }
    
    void left() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, 255);
      analogWrite(PWMB, 255);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Left");
    }
    
    void right() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, 255);
      analogWrite(PWMB, 255);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Right");
    }
    
    void stop() {
      digitalWrite(PWMA, 0);
      digitalWrite(PWMB, 0);
      digitalWrite(STBY, LOW);
    
      Serial.println("Stop");
    }

### Movimiento del coche y que detecta pared
    //    The direction of the car's movement
    //  ENA   ENB   IN1   IN2   IN3   IN4   Description  
    //  HIGH  HIGH  HIGH  LOW   LOW   HIGH  Car is runing forward
    //  HIGH  HIGH  LOW   HIGH  HIGH  LOW   Car is runing back
    //  HIGH  HIGH  LOW   HIGH  LOW   HIGH  Car is turning left
    //  HIGH  HIGH  HIGH  LOW   HIGH  LOW   Car is turning right
    //  HIGH  HIGH  LOW   LOW   LOW   LOW   Car is stoped
    //  HIGH  HIGH  HIGH  HIGH  HIGH  HIGH  Car is stoped
    //  LOW   LOW   N/A   N/A   N/A   N/A   Car is stoped
    
    // Pines correctos para Elegoo Smart Car V1.1
    #define PWMA 5
    #define PWMB 6
    #define AIN1 7
    #define AIN2 11
    #define BIN1 8
    #define BIN2 13
    #define STBY 3
    #define TimeLapse 1000
    #define Speed 125
    
    int trigPin = 13;
    int echoPin = 12;
    float duration, distance;
    float umbral = 20;
    bool movement=true;
    
    #include <Servo.h>
    
    Servo servo;
    const int PIN_SERVO = 10;
    
    void setup() {
      Serial.begin(9600);
    
      pinMode(PWMA, OUTPUT);
      pinMode(PWMB, OUTPUT);
      pinMode(AIN1, OUTPUT);
      pinMode(AIN2, OUTPUT);
      pinMode(BIN1, OUTPUT);
      pinMode(BIN2, OUTPUT);
      pinMode(STBY, OUTPUT);
    
      digitalWrite(STBY, HIGH); // ¡IMPORTANTE!
    
      servo.attach(PIN_SERVO); // D9
      servo.write(90);
    
      pinMode(trigPin, OUTPUT);
      pinMode(echoPin, INPUT);
    }
    
    void loop() { 
      if(!UltrasonidosLogic()&&movement){
        forward();
      }
      else{
        stop();
        delay(1000);
        right();
        delay(500);
        movement=true;
      }
    }
    
    
    bool UltrasonidosLogic(){
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
          movement=false;
          return true;
      }
      else{
        Serial.print(" Lejos ");
        return false;
      }
    }
    void forward() {
      digitalWrite(STBY, HIGH);
      
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Forward");
    }
    
    void back() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Back");
    }
    
    void left() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Left");
    }
    
    void right() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Right");
    }
    
    void stop() {
      digitalWrite(PWMA, 0);
      digitalWrite(PWMB, 0);
      digitalWrite(STBY, LOW);
    
      Serial.println("Stop");
    }

### Pruebas con el mando y el coche

        //    The direction of the car's movement
    //  ENA   ENB   IN1   IN2   IN3   IN4   Description  
    //  HIGH  HIGH  HIGH  LOW   LOW   HIGH  Car is runing forward
    //  HIGH  HIGH  LOW   HIGH  HIGH  LOW   Car is runing back
    //  HIGH  HIGH  LOW   HIGH  LOW   HIGH  Car is turning left
    //  HIGH  HIGH  HIGH  LOW   HIGH  LOW   Car is turning right
    //  HIGH  HIGH  LOW   LOW   LOW   LOW   Car is stoped
    //  HIGH  HIGH  HIGH  HIGH  HIGH  HIGH  Car is stoped
    //  LOW   LOW   N/A   N/A   N/A   N/A   Car is stoped
    
    // Pines correctos para Elegoo Smart Car V1.1
    #define PWMA 5
    #define PWMB 6
    #define AIN1 7
    #define AIN2 11
    #define BIN1 8
    #define BIN2 13
    #define STBY 3
    #define TimeLapse 1000
    #define Speed 125
    
    int trigPin = 13;
    int echoPin = 12;
    float duration, distance;
    float umbral = 20;
    bool movement=true;
    
    #include <Servo.h>
    
    Servo servo;
    const int PIN_SERVO = 10;
    
    #include <IRremote.h>
    const int PIr = 9;
    int comando;
    int lastCommand;
    
    bool isPressed;
    
    void setup() {
      Serial.begin(9600);
    
      pinMode(PWMA, OUTPUT);
      pinMode(PWMB, OUTPUT);
      pinMode(AIN1, OUTPUT);
      pinMode(AIN2, OUTPUT);
      pinMode(BIN1, OUTPUT);
      pinMode(BIN2, OUTPUT);
      pinMode(STBY, OUTPUT);
    
      digitalWrite(STBY, HIGH); 
    
      servo.attach(PIN_SERVO); // D9
      servo.write(90);
    
      pinMode(trigPin, OUTPUT);
      pinMode(echoPin, INPUT);
    
      IrReceiver.begin(PIr); 
    }
    
    void loop() { 
      if(IrReceiver.decode()){
        Serial.print("Codigo recibido: ");
    	  comando = IrReceiver.decodedIRData.command;
        switch(comando){
          case 24:
          forward();
          break;
          case 8:
          left();
          break;
          case 90:
          right();
          break;
          case 82:
          back();
          break;
          case 28:
          stop();
        }
        IrReceiver.resume();
      }
      
      
      
    }
    
    
    bool UltrasonidosLogic(){
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
          movement=false;
          return true;
      }
      else{
        Serial.print(" Lejos ");
        return false;
      }
    }
    void forward() {
      digitalWrite(STBY, HIGH);
      
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Forward");
    }
    
    void back() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Back");
    }
    
    void left() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Left");
    }
    
    void right() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Right");
    }
    
    void stop() {
      digitalWrite(PWMA, 0);
      digitalWrite(PWMB, 0);
      digitalWrite(STBY, LOW);
    
      Serial.println("Stop");
    }

### Pruebas para juntar los modos automatico y manual
    
    //    The direction of the car's movement
    //  ENA   ENB   IN1   IN2   IN3   IN4   Description  
    //  HIGH  HIGH  HIGH  LOW   LOW   HIGH  Car is runing forward
    //  HIGH  HIGH  LOW   HIGH  HIGH  LOW   Car is runing back
    //  HIGH  HIGH  LOW   HIGH  LOW   HIGH  Car is turning left
    //  HIGH  HIGH  HIGH  LOW   HIGH  LOW   Car is turning right
    //  HIGH  HIGH  LOW   LOW   LOW   LOW   Car is stoped
    //  HIGH  HIGH  HIGH  HIGH  HIGH  HIGH  Car is stoped
    //  LOW   LOW   N/A   N/A   N/A   N/A   Car is stoped
    
    // Pines correctos para Elegoo Smart Car V1.1
    #define PWMA 5
    #define PWMB 6
    #define AIN1 7
    #define AIN2 11
    #define BIN1 8
    #define BIN2 13
    #define STBY 3
    #define TimeLapse 1000
    #define Speed 125
    
    int trigPin = 13;
    int echoPin = 12;
    float duration, distance;
    float umbral = 20;
    bool movement=true;
    
    #include <Servo.h>
    
    Servo servo;
    const int PIN_SERVO = 10;
    
    #include <IRremote.h>
    const int PIr = 9;
    int comando;
    int lastCommand;
    
    bool isPressed;
    bool isManual;
    bool canChange;
    
    void setup() {
      Serial.begin(9600);
    
      pinMode(PWMA, OUTPUT);
      pinMode(PWMB, OUTPUT);
      pinMode(AIN1, OUTPUT);
      pinMode(AIN2, OUTPUT);
      pinMode(BIN1, OUTPUT);
      pinMode(BIN2, OUTPUT);
      pinMode(STBY, OUTPUT);
    
      digitalWrite(STBY, HIGH); 
    
      servo.attach(PIN_SERVO); // D9
      servo.write(90);
    
      pinMode(trigPin, OUTPUT);
      pinMode(echoPin, INPUT);
    
      IrReceiver.begin(PIr); 
    }
    
    void loop() { 
      if(IrReceiver.decode()){
        Serial.print("Codigo recibido: ");
    	  comando = IrReceiver.decodedIRData.command;
        Serial.println(comando);
        if(comando == 67){
          if(canChange){
           isManual=!isManual;
           canChange=false;
           stop();
          }
          else
          {
            delay(1000);
            canChange=true;
          }
    
        }
        
        IrReceiver.resume();
      }
      if(isManual){
        ManualMode();
      }
      else{
        AutomaticMode();
      }
      Serial.println(isManual);
      
      
      
    }
    
    // void ManualMode(){
    //   if(IrReceiver.decode()){
    //     //Serial.print("Codigo recibido: ");
    // 	  comando = IrReceiver.decodedIRData.command;
    //     switch(comando){
    //       case 24:
    //       forward();
    //       break;
    //       case 8:
    //       left();
    //       break;
    //       case 90:
    //       right();
    //       break;
    //       case 82:
    //       back();
    //       break;
    //       case 28:
    //       stop();
    //     }
    //     IrReceiver.resume();
    //   }
    // }
    void ManualMode(){
      switch(comando){
          case 24:
          forward();
          break;
          case 8:
          left();
          break;
          case 90:
          right();
          break;
          case 82:
          back();
          break;
          case 28:
          stop();
        }
    }
    void AutomaticMode(){
       if(!UltrasonidosLogic()&&movement){
        forward();
      }
      else{
        stop();
        delay(1000);
        right();
        delay(500);
        movement=true;
      }
    }
    
    bool UltrasonidosLogic(){
      digitalWrite(trigPin, LOW);
      delayMicroseconds(2);
      digitalWrite(trigPin, HIGH);
      delayMicroseconds(10);
      digitalWrite(trigPin, LOW);
    
      duration = pulseIn(echoPin, HIGH);
      distance = (duration*.0343)/2;
      //Serial.print("Distance: ");
    
      
      delay(100);
      //Serial.println(distance);
      if(distance < umbral){
          //Serial.print(" Muy cerca ");
          movement=false;
          return true;
      }
      else{
        //Serial.print(" Lejos ");
        return false;
      }
    }
    void forward() {
      digitalWrite(STBY, HIGH);
      
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Forward");
    }
    
    void back() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Back");
    }
    
    void left() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, LOW);
      digitalWrite(AIN2, HIGH);
      digitalWrite(BIN1, HIGH);
      digitalWrite(BIN2, LOW);
    
      Serial.println("Left");
    }
    
    void right() {
      digitalWrite(STBY, HIGH);
    
      analogWrite(PWMA, Speed);
      analogWrite(PWMB, Speed);
    
      digitalWrite(AIN1, HIGH);
      digitalWrite(AIN2, LOW);
      digitalWrite(BIN1, LOW);
      digitalWrite(BIN2, HIGH);
    
      Serial.println("Right");
    }
    
    void stop() {
      digitalWrite(PWMA, 0);
      digitalWrite(PWMB, 0);
      digitalWrite(STBY, LOW);
    
      Serial.println("Stop");
    }




