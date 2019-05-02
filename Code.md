/*
1) Wifi konexioaren izena eta pasahitza defenitu ssi[] eta password[]
2) Kodea bidali NODEMCUra
3) NODEMCU-a berrabiarazi
4) Sorturiko Wifi konexiora konektatu PC edo mugikor batetik
5) 192.168.4.1 web orrialdea ireki */
 
#include <ESP8266WiFi.h> // ESP8266WiFi liburutegia gehitu
const char ssid[] = "******"; //Nodemcuak sortuko duen Wifi sarearen izena
const char password[] = "************"; //Nodemcuak sortuko duen Wifi sarearen pasahitza
WiFiServer server(80); //komunikazio portua

/* define L298N or L293D motor control pins */
int leftMotorForward = 5;     /* GPIO2(D1) -> IN3   */
int rightMotorForward = 4;   /* GPIO15(D2) -> IN1  */
int leftMotorBackward = 0;    /* GPIO0(D3) -> IN4   */
int rightMotorBackward = 2;  /* GPIO13(D4) -> IN2  */


/* define L298N or L293D enable pins */
int rightMotorENB = 14; /* GPIO14(D5) -> Motor-A Enable */
int leftMotorENB = 12;  /* GPIO12(D6) -> Motor-B Enable */

WiFiClient client;
String  data =""; 

 
void setup() {
/* initialize motor control pins as output */
  pinMode(leftMotorForward, OUTPUT);
  pinMode(rightMotorForward, OUTPUT); 
  pinMode(leftMotorBackward, OUTPUT);  
  pinMode(rightMotorBackward, OUTPUT);

  /* initialize motor enable pins as output */
  pinMode(leftMotorENB, OUTPUT); 
  pinMode(rightMotorENB, OUTPUT);

Serial.begin(115200);
server.begin(); //Zerbitzailea abiarazi
WiFi.mode(WIFI_AP);
WiFi.softAP(ssid, password); //Klabedun sarea, 1 kanalean eta ikusgarri
//WiFi.softAP(ssid); //Sare irekia
Serial.println();
Serial.print("Access Pointaren IP zenbakia: "); //IP zenbakia inprimatzen du
Serial.println(WiFi.softAPIP());
Serial.print("Access Pointaren MAC zenbakia: "); //MAC zenbakia inprimatzen du
Serial.println(WiFi.softAPmacAddress());
}
 
void loop(){
// Bezeroa konektatu den konprobatzen du
client = server.available();
if (!client) {return;} // BEzeroak eskaeraren bat egin bitartean itxaroten du
Serial.println("Bezero berria");

data = checkClient ();

client.flush();
// Eskaera konprobatu
Serial.println(data);
if (data == "aurrera") {MotorForward();}
else if (data == "atzera") {MotorBackward();}
else if (data == "ezkerrera"){TurnLeft();}
else if (data == "eskubira"){TurnRight();}
else if (data == "gelditu"){MotorStop();}
}
/********************************************* FORWARD *****************************************************/
void MotorForward(void)   
{
  Serial.println("Motorra aurrera");
  digitalWrite(leftMotorENB,HIGH);
  digitalWrite(rightMotorENB,HIGH);
  digitalWrite(leftMotorForward,HIGH);
  digitalWrite(rightMotorForward,HIGH);
  digitalWrite(leftMotorBackward,LOW);
  digitalWrite(rightMotorBackward,LOW);
}

/********************************************* BACKWARD *****************************************************/
void MotorBackward(void)   
{
 Serial.println("Motorra atzera");
  digitalWrite(leftMotorENB,HIGH);
  digitalWrite(rightMotorENB,HIGH);
  digitalWrite(leftMotorBackward,HIGH);
  digitalWrite(rightMotorBackward,HIGH);
  digitalWrite(leftMotorForward,LOW);
  digitalWrite(rightMotorForward,LOW);
}

/********************************************* TURN LEFT *****************************************************/
void TurnLeft(void)   
{
  Serial.println("Motorra ezkerrera");
  digitalWrite(leftMotorENB,HIGH);
  digitalWrite(rightMotorENB,HIGH); 
  digitalWrite(leftMotorForward,LOW);
  digitalWrite(rightMotorForward,HIGH);
  digitalWrite(rightMotorBackward,LOW);
  digitalWrite(leftMotorBackward,HIGH);  
}

/********************************************* TURN RIGHT *****************************************************/
void TurnRight(void)   
{
  Serial.println("Motorra eskubira");
  digitalWrite(leftMotorENB,HIGH);
  digitalWrite(rightMotorENB,HIGH);
  digitalWrite(leftMotorForward,HIGH);
  digitalWrite(rightMotorForward,LOW);
  digitalWrite(rightMotorBackward,HIGH);
  digitalWrite(leftMotorBackward,LOW);
}

/********************************************* STOP *****************************************************/
void MotorStop(void)   
{
  Serial.println("Motorra gelditu");
  digitalWrite(leftMotorENB,LOW);
  digitalWrite(rightMotorENB,LOW);
  digitalWrite(leftMotorForward,LOW);
  digitalWrite(leftMotorBackward,LOW);
  digitalWrite(rightMotorForward,LOW);
  digitalWrite(rightMotorBackward,LOW);
}

/********************************** RECEIVE DATA FROM the APP ******************************************/
String checkClient(void)
{
  while(!client.available()) delay(1);
  Serial.printf("Access Point-era konektaturiko bezeroak: %dn", WiFi.softAPgetStationNum());
  String request = client.readStringUntil('\r');
  Serial.println(request);
  request.remove(0, 5);
  request.remove(request.length()-9,9);
  return request;
}
