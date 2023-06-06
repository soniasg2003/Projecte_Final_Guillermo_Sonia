# Projecte_Final_Guillermo_Sonia

//CAR PROJECT
//Created by :
//Sonia Sahuquillo Guillen
//Guilermo Efren Medina Guaman
/* PA + ULTRASONICO--------------------------------------------------------------------------*/
#include <Arduino.h>
#include <WiFi.h>
#include <WebServer.h>
#include <Servo.h>

/* Put your SSID & Password */
const char* ssid = "ESP32";  // Enter SSID here
const char* password = "cochebrum";  //Enter Password here

/* Put IP Address details */
IPAddress local_ip(192,168,24,1);
IPAddress gateway(192,168,24,1);
IPAddress subnet(255,255,255,0);

WebServer server(80);

uint8_t LED1pin = 26;
bool LED1status = LOW;

uint8_t LED2pin = 27;
bool LED2status = LOW;

uint8_t LED3pin = 25;
bool LED3status = LOW;

uint8_t LED4pin = 33;
bool LED4status = LOW;

// Motor 
int motor1Pin1 = 19; 
int motor1Pin2 = 16; 
int enable1Pin = 14;

const int trigPin = 5;
const int echoPin = 18;

//define sound speed in cm/uS
#define SOUND_SPEED 0.034
#define CM_TO_INCH 0.393701
// ESP32 pin GIOP17 connected to Piezo Buzzer's pin
#define BUZZER_PIN 17 

long duration;
float distanceCm;
float limitzone;
int turnright =105;
int turnleft = 85;

Servo myservo;  // create servo object to control a servo
// twelve servo objects can be created on most boards

int pos = 90 ;

// Setting PWM properties
const int freq = 30000;
const int pwmChannel = 0;
const int resolution = 8;
int dutyCycle = 200;

void handle_OnConnect();
void handle_led1on();
void handle_led1off();
void handle_led2on();
void handle_led2off();
void handle_led3on();
void handle_led3off();
void handle_led4on();
void handle_led4off();
void handle_NotFound();
String SendHTML(uint8_t led1stat,uint8_t led2stat,uint8_t led3stat,uint8_t led4stat);


void setup() {
  Serial.begin(115200);
  pinMode(LED1pin, OUTPUT);
  pinMode(LED2pin, OUTPUT);
  pinMode(LED3pin, OUTPUT);
  pinMode(LED4pin, OUTPUT);

  // sets the pins as outputs:
  pinMode(motor1Pin1, OUTPUT);
  pinMode(motor1Pin2, OUTPUT);
  pinMode(enable1Pin, OUTPUT);

  // configure LED PWM functionalitites
  ledcSetup(pwmChannel, freq, resolution);
  
  // attach the channel to the GPIO to be controlled
  ledcAttachPin(enable1Pin, pwmChannel);

  // set ESP32 pin to output mode
  pinMode(BUZZER_PIN, OUTPUT);
  //sets the limitzone for the stop
  limitzone=30;
  // Sets the trigPin as an Output
  pinMode(trigPin, OUTPUT); 
  // Sets the echoPin as an Input
  pinMode(echoPin, INPUT); 

  // attaches the servo on pin 13 to the servo object
  myservo.attach(13);  
  
  // Connect to Wi-Fi network with SSID
  WiFi.softAP(ssid);
  WiFi.softAPConfig(local_ip, gateway, subnet);
  delay(100);
  
  distanceCm=100;

  server.on("/", handle_OnConnect);
  server.on("/led1on", handle_led1on);
  server.on("/led1off", handle_led1off);
  server.on("/led2on", handle_led2on);
  server.on("/led2off", handle_led2off);
  server.on("/led3on", handle_led3on);
  server.on("/led3off", handle_led3off);
  server.on("/led4on", handle_led4on);
  server.on("/led4off", handle_led4off);
  server.onNotFound(handle_NotFound);

  server.begin();
  Serial.println("HTTP server started");

}
void loop() {
  server.handleClient();
  // Clears the trigPin
  digitalWrite(trigPin, LOW);
  delayMicroseconds(5);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
  
  // Calculate the distance
  distanceCm = duration * SOUND_SPEED/2;
  
  // Prints the distance in the Serial Monitor
  Serial.print("Distance (cm): ");
  Serial.println(distanceCm);
  delay(100);
  
  
  if(LED1status )
  {digitalWrite(LED1pin, HIGH);}
  else
  {digitalWrite(LED1pin, LOW);}
  
  if(LED2status==true)
  {digitalWrite(LED2pin, HIGH);}
  else
  {digitalWrite(LED2pin, LOW);}

  if(LED3status)
  {digitalWrite(LED3pin, HIGH);
  delay(50);
  digitalWrite(LED3pin, LOW);
  }
  else
  {digitalWrite(LED3pin, LOW);}
  
  if(LED4status)
  {digitalWrite(LED4pin, HIGH);
  delay(50);
  digitalWrite(LED4pin, LOW);
  }
  else
  {digitalWrite(LED4pin, LOW);}

  if(distanceCm<=limitzone){
    digitalWrite(BUZZER_PIN, HIGH); // turn on Piezo 
    delay(200);
    digitalWrite(BUZZER_PIN, LOW);
  }
if(LED1status == HIGH){
digitalWrite(motor1Pin1, LOW);
digitalWrite(motor1Pin2, HIGH);
if(distanceCm<=limitzone){
    LED1status = LOW;
    Serial.println("GPIO26 Status: OFF");
    server.send(200, "text/html", SendHTML(false,LED2status,LED3status,LED4status)); 
  }
}else if(LED2status == HIGH){
digitalWrite(motor1Pin1, HIGH);
digitalWrite(motor1Pin2, LOW);
}else{
digitalWrite(motor1Pin1, LOW);
digitalWrite(motor1Pin2, LOW);
}

  if(LED3status == true && pos != turnright){
      for (pos ; pos <= turnright; pos += 1) { // goes from 90 degrees to 180 degree
      myservo.write(pos);              // tell servo to go to position in variable 'pos'
      delay(2);                       // waits 5ms for the servo to reach the position
      handle_led3on();
    }
  }
  if(LED4status == true && pos != turnleft){
      for (pos ; pos >= turnleft; pos -= 1) { // goes from 90 degrees to 0 degrees
      myservo.write(pos);              // tell servo to go to position in variable 'pos'
      delay(2);                       // waits 5ms for the servo to reach the position
      handle_led4on();
    }
  }
  if(LED3status == false && LED4status == false){
    if(pos<=(turnright+1) && pos>90){
        for (pos; pos >= 90; pos -= 1) { // goes from pos degrees to 90 degrees
        myservo.write(pos);              // tell servo to go to position in variable 'pos'
        delay(2);                       // waits 5ms for the servo to reach the position
        
      }
    }else if(pos>=(turnleft-1) && pos<90){
        for (pos ; pos <= 90; pos += 1) { // goes from pos degrees to 90 degrees
        myservo.write(pos);              // tell servo to go to position in variable 'pos'
        delay(2);                       // waits 5ms for the servo to reach the position
      }
    }
  }
  
}

void handle_OnConnect() {
  LED1status = LOW;
  LED2status = LOW;
  LED3status = LOW;
  LED4status = LOW;
  Serial.println("GPIO26 Status: OFF | GPIO27 Status: OFF | GPIO25 Status: OFF | GPIO33 Status: OFF");
  server.send(200, "text/html", SendHTML(LED1status,LED2status,LED3status,LED4status)); 
}

void handle_led1on() {
  LED1status = HIGH;
  Serial.println("GPIO26 Status: ON");
  LED2status = LOW;
  Serial.println("GPIO27 Status: OFF");
  server.send(200, "text/html", SendHTML(true,false,LED3status,LED4status));
}

void handle_led1off() {
  LED1status = LOW;
  Serial.println("GPIO26 Status: OFF");
  server.send(200, "text/html", SendHTML(false,LED2status,LED3status,LED4status)); 
}

void handle_led2on() {
  LED2status = HIGH;
  Serial.println("GPIO27 Status: ON");
  LED1status = LOW;
  Serial.println("GPIO26 Status: OFF");
  server.send(200, "text/html", SendHTML(false,true,LED3status,LED4status));
  
}

void handle_led2off() {
  LED2status = LOW;
  Serial.println("GPIO27 Status: OFF");
  server.send(200, "text/html", SendHTML(LED1status,false,LED3status,LED4status)); 
}
void handle_led3on() {
  LED3status = HIGH;
  Serial.println("GPIO25 Status: ON");
  LED4status = LOW;
  Serial.println("GPIO33 Status: OFF");
  server.send(200, "text/html", SendHTML(LED1status,LED2status,true,false));
}

void handle_led3off() {
  LED3status = LOW;
  Serial.println("GPIO25 Status: OFF");
  server.send(200, "text/html", SendHTML(LED1status,LED2status,false,LED4status)); 
}

void handle_led4on() {
  LED4status = HIGH;
  Serial.println("GPIO33 Status: ON");
  LED3status = LOW;
  Serial.println("GPIO25 Status: OFF");
  server.send(200, "text/html", SendHTML(LED1status,LED2status,false,true));
}

void handle_led4off() {
  LED4status = LOW;
  Serial.println("GPIO33 Status: OFF");
  server.send(200, "text/html", SendHTML(LED1status,LED2status,LED3status,false)); 
}

void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
}

String SendHTML(uint8_t led1stat,uint8_t led2stat,uint8_t led3stat,uint8_t led4stat){
  String ptr = "<!DOCTYPE html> <html>\n";
  ptr +="<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=no\">\n";
  ptr +="<title>LED Control</title>\n";
  ptr +="<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}\n";
  ptr +="body{margin-top: 50px;} h1 {color: #444444;margin: 50px auto 30px;} h3 {color: #444444;margin-bottom: 50px;}\n";
  ptr +=".button {display: block;width: 80px;background-color: #3498db;border: none;color: white;padding: 13px 30px;text-decoration: none;font-size: 25px;margin: 0px auto 35px;cursor: pointer;border-radius: 4px;}\n";
  ptr +=".button-on {background-color: #3498db;}\n";
  ptr +=".button-on:active {background-color: #2980b9;}\n";
  ptr +=".button-off {background-color: #34495e;}\n";
  ptr +=".button-off:active {background-color: #2c3e50;}\n";
  ptr +="p {font-size: 14px;color: #888;margin-bottom: 10px;}\n";
  ptr +="</style>\n";
  ptr +="</head>\n";
  ptr +="<body>\n";
  ptr +="<h1>ESP32 Web Server</h1>\n";
  ptr +="<h3>Using Access Point(AP) Mode</h3>\n";
  
   if(led1stat)
  {ptr +="<p>DRIVE Status: ON</p><a class=\"button button-off\" href=\"/led1off\">OFF</a>\n";}
  else
  {ptr +="<p>DRIVE Status: OFF</p><a class=\"button button-on\" href=\"/led1on\">ON</a>\n";}

  if(led2stat)
  {ptr +="<p>REVERSE Status: ON</p><a class=\"button button-off\" href=\"/led2off\">OFF</a>\n";}
  else
  {ptr +="<p>REVERSE Status: OFF</p><a class=\"button button-on\" href=\"/led2on\">ON</a>\n";}

  if(led3stat)
  {ptr +="<p>RIGHT Status: ON</p><a class=\"button button-off\" href=\"/led3off\">OFF</a>\n";}
  else
  {ptr +="<p>RIGHT Status: OFF</p><a class=\"button button-on\" href=\"/led3on\">ON</a>\n";}

  if(led4stat)
  {ptr +="<p>LEFT Status: ON</p><a class=\"button button-off\" href=\"/led4off\">OFF</a>\n";}
  else
  {ptr +="<p>LEFT Status: OFF</p><a class=\"button button-on\" href=\"/led4on\">ON</a>\n";}

  ptr +="</body>\n";
  ptr +="</html>\n";
  return ptr;
}
/* PA + ULTRASONICO END--------------------------------------------------------------------------*/
