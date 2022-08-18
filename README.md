#include <LiquidCrystal.h>
#include <WiFi.h>
#include "ThingSpeak.h" // always include thingspeak header file after other header files and custom macros

const char* ssid = "ranjith";
const char* pass =  "12345678";  // your network password          
//inizialization
float value0;
int value1;
int value2;
int turbidity;
int sensorpin1=35;//TEMPERATURE
int sensorpin2=32;//TURBITIDY
int sensorpin3=33;//PH
const int rs = 13, en = 12, d4 = 14, d5 = 27, d6 = 26, d7 = 25;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);


WiFiClient  client;
unsigned long myChannelNumber =1528992;
const char * myWriteAPIKey = "A4KE5SPV0AR1IL91";


void setup()
{
  //
  lcd.clear();
  pinMode(35,INPUT);
  pinMode(15,OUTPUT);
  pinMode(33,INPUT);
  pinMode(2,OUTPUT);
  pinMode(32,INPUT);
  pinMode(4,OUTPUT);
  Serial.begin(115200);
   
  //Initialize serial
  while (!Serial)
  {
    ; // wait for serial port to connect. Needed for Leonardo native USB port only
  }
 
  WiFi.mode(WIFI_STA);  
  ThingSpeak.begin(client);  // Initialize ThingSpeak
}

void loop()
{

  // Connect or reconnect to WiFi
  if(WiFi.status() != WL_CONNECTED)
  {
    Serial.print("Attempting to connect to SSID: ");
    while(WiFi.status() != WL_CONNECTED)
    {
      WiFi.begin(ssid, pass);  // Connect to WPA/WPA2 network. Change this line if using open or WEP network
      Serial.print(".");
      delay(5000);    
    }
    Serial.println("\nConnected.");
  }

 //TEMPERATURE SENSOR FORMULA
value0 = analogRead(sensorpin1);
float voltage = (value0 / 1024.0) * 5.0;
float tempC = (voltage - .5) * 100;
float tempF = (tempC * 1.8) + 32;
Serial.println("temperature_value: ");
Serial.println(tempF);
//TEMPERATURE SENSOR
if (tempF <= 450||tempF>=500)
{
digitalWrite(15, HIGH);
Serial.println("TEMPERATURE  NORMAL ");
lcd.setCursor(0,0);
lcd.print("TEMPERATURE NORMAL ");

}
else
{
digitalWrite(15, LOW);
Serial.println("TEMPERATURE HIGH");
lcd.setCursor(0,0);
delay(1000);
lcd.print("TEMPERATURE HIGH ");
digitalWrite(2,LOW);
}

//TURBIDITY FORMULA
value1 = analogRead(sensorpin2);
int ntu=value1 * (5.0 / 1024.0);
Serial.println("Turbitity_value:");
Serial.println(ntu);
if(ntu<20)
{
digitalWrite(2,HIGH);
Serial.println("TURBIDITY  NORMAL.");
lcd.setCursor(0,0);
delay(1000);
lcd.print("TURBIDITY  NORMAL");
}
else
{
 digitalWrite(2,LOW);
 Serial.println("TURBIDITY HIGH");
 lcd.setCursor(0,0);
 delay(1000);
 lcd.print("TURBIDITY  HIGH ");

}

//PH SENSOR FORMULA
value2=analogRead(sensorpin3);
Serial.println("PH VALUE:");
Serial.println(value2);
 if(value2>4.25||value2<6.50)
{
  digitalWrite(4,HIGH);
 Serial.println("PH VALUE NORMAL.");
 lcd.setCursor(0,0);
 delay(1000);
 lcd.print("PH VALUE NORMAL ");
}
else
{
  digitalWrite(4,LOW);
 Serial.println(" PH VALUE  HIGH");
 lcd.setCursor(0,0);
 lcd.print("PH VALUE HIGH ");
 delay(1000);

}
  // set the fields with the values
  ThingSpeak.setField(1, tempF);
  ThingSpeak.setField(2,ntu);
  ThingSpeak.setField(3,value2);
  // write to the ThingSpeak channel
 
  int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
  if(x == 200)
  {
    Serial.println("Channel update successful.");
   
  }
  else
  {
    Serial.println("Problem updating channel. HTTP error code " + String(x));
  }
   // Wait 20 seconds to update the channel again
 delay(15000);
} 
