#include <EveryTimer.h>
#define PERIOD_MS 1000

EveryTimer timer;
bool active = true;

//--------------------------------------------------------ULTRA SONIC-WASTE---------------------------------------------------------------------------------------------


int trigPin0 = 13;    // Trigger DE
int echoPin0 = 12; // Echo DE



int trigPin1 = 5;    // Trigger DE
int echoPin1 = 23; // Echo DE


int value0;
long duration0, cm0, inches0;

int value1;
long duration1, cm1, inches1;





//------------------------------------------------AIR----------------------------------------------------
#define air1 36
//-----------------------------------------------blynk----------------------------------------
// int buttonread;

#define BLYNK_PRINT Serial  
#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>

#define BLYNK_TEMPLATE_ID "TMPL3EaNYmxZB"
#define BLYNK_TEMPLATE_NAME "WET AND DRY SEGREGATION"
#define BLYNK_AUTH_TOKEN "nAk88hUvNT08mXa6d6Jb8Ums0jHgi0ak"



char ssid[] = "IOT";
char pass[] = "123456789";
char auth[] = BLYNK_AUTH_TOKEN;
String receved;

int buttonread=0;

//-----------------------------------------servo-------------------------------------

#include <ESP32Servo.h>

Servo myservo;  // create servo object to control a servo


int pos = 0;   

//-----------------------------------------ultra sonic-------------------------------------
int trigPin = 17;    // Trigger
int echoPin = 16; // Echo
int value;
long duration, cm, inches;
int sec=0;

//------------------------------------------LCD-------------------------------------
#include <LCD_I2C.h>
LCD_I2C lcd(0x27);

//------------------------------------------water sensor-----------------------------------------------
#define water 39
//----------------------------------------------set up---------------------------------------

void setup()
{
  
    Serial.begin(9600);
//------------------------------------------water sensor-----------------------------------------------
     pinMode(water,INPUT);
//-----------------------------------------------AIR--------------------------------------
     pinMode(air1,INPUT);
//-----------------------------------------ultra sonic-------------------------------------
 pinMode(trigPin, OUTPUT);
 timer.Every(PERIOD_MS, action);
 
//-----------------------------------------servo------------------------------------
 Serial.begin(9600);
  myservo.attach(15);
  myservo.write(80);  // attaches the servo on pin 18 to the servo object
  Serial.println("--------------------------motor");
 //--------------------------------------------------------ULTRA WET -----------------------------------------   
  pinMode(trigPin0, OUTPUT);
  pinMode(echoPin0, INPUT);
//--------------------------------------------------------ULTRA DRY-----------------------------------------
  pinMode(trigPin1, OUTPUT);
  pinMode(echoPin1, INPUT);
//------------------------------------------LCD-----------------------------------------



  lcd.begin(); 
  lcd.backlight();
  lcd.setCursor(0,0);
  lcd.print("  WET AND DRY ");
  lcd.setCursor(0,1);
  lcd.print("WASTESEGRECATION ");
  delay(3000);
  lcd.clear();
//-------------------------------------blynk------------------------------------------------
 Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);
  
   
}

//----------------------------------------------loop---------------------------------------

void loop() 
{

int air1val=analogRead(air1);
   Serial.print("MAP___________________________________________________________________AIR = ");
   Serial.println(air1val);
   air1val=map(air1val,100,800,0,100);


   if(air1val<=0)
  {
      air1val=0;
  }
  
  else if(air1val>=100)
  {
       air1val=100;
  }
  
   Serial.print("---------------------------------------------OG---air1 = ");
   Serial.println(air1val);
//------------------------------------------water sensor-----------------------------------------------
Serial.print("WAter :");
Serial.println(analogRead(water));
delay(500);


//-----------------------------------------ultra sonic-------------------------------------

  digitalWrite(trigPin, LOW);
  delayMicroseconds(15);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  pinMode(echoPin, INPUT);
  duration = pulseIn(echoPin, HIGH);


 
  // Convert the time into a distance
  cm = (duration/2) / 29.1;     // Divide by 29.1 or multiply by 0.0343
  inches = (duration/2) / 74;   // Divide by 74 or multiply by 0.0135
  
   Serial.print("------------------------VOL : ");
  Serial.println(cm);

Serial.print("--------------------------------Sec:");
Serial.println(sec);

//-------------------------------------------------object detected-----------------------------------

if(cm<=15 && cm>0)
  {
    
  Serial.println("OBJECT PLACED");
 timer.Update();


   lcd.setCursor(0,0);
  lcd.print("OBJECT PLACED");
  }

  else
  {

  lcd.setCursor(0,0);
  lcd.print("                     ");
    
 }

//------------------------------------------dry obj--------------------------------------------------------------------------------------------------
if(analogRead(water)>=4000 && cm<=15)
  {
        Serial.print("--------------Dry object");
        lcd.setCursor(0,1);  
        lcd.print("   DRY WASTE");
        Blynk.virtualWrite(V0,"     OBJECT DETECTED");
        Blynk.virtualWrite(V1,"    DRY WASTE");
        lcd.setCursor(0,1);
        lcd.print("   DRY WASTE");
        sec=0;
        delay(2000);
        Blynk.virtualWrite(V0,"     ");
        Blynk.virtualWrite(V1,"   ");
        lcd.setCursor(0,1);
        lcd.print("              ");
        dry();
  }

//---------------------------------------------wet obj--------------------------------------------------------------
 if(analogRead(water)<=4000 && cm<=15)
 {
      Serial.println("------------------4------5------4-------------WET OBJECT");
      Blynk.virtualWrite(V0,"    OBJECT DETECTED");
      Blynk.virtualWrite(V1,"     WET WASTE");
      lcd.setCursor(0,1); 
      lcd.print("   WET WASTE");
      delay(2000);
      Blynk.virtualWrite(V0,"     ");
      Blynk.virtualWrite(V1,"   ");
      lcd.setCursor(0,1);
      lcd.print("                ");
      wet();
      sec=0;
 }

 delay(300);

ultra0();
ultra1();
//--------------------------------------------------------ULTRA WET -----------------------------------------
if(cm0<=4)
{
  Serial.println("WET WASTE IS HIGH");
    Blynk.virtualWrite(V6,"WET WASTE IS HIGH");
}
else
{
      Blynk.virtualWrite(V6,"                    ");
}
//--------------------------------------------------------ULTRA  DRY -----------------------------------------

if(cm1<=4)
{
  Serial.println("DRY WASTE IS HIGH");
  Blynk.virtualWrite(V5,"DRY WASTE IS HIGH");
}

else
{
      Blynk.virtualWrite(V5,"                    ");
}

if(buttonread==1)
{
      Blynk.virtualWrite(V3,air1val);
      Blynk.virtualWrite(V2,cm);
}

}
//-----------------------------------------dry--------------------------------------------------------
void dry()
{
   for (pos = 80; pos >= 50; pos -= 1) 
  {
   
    myservo.write(pos);    
    delay(15); 
    Serial.println("-------------------------DRY");            
  }

delay(800);

    for (pos = 50; pos <= 80; pos+= 1) 
  {
   
    myservo.write(pos);    
    delay(15); 
    Serial.println("--------------------------D/R/Y");            
  }
}


//-----------------------------------------wet--------------------------------------------------------
void wet()
{
    for (pos = 80; pos <= 120; pos += 1) 
  {
   
    myservo.write(pos);    
    delay(15); 
    Serial.println("--------------------------WET");            
  }

delay(800);

    for (pos = 120; pos >= 80; pos -= 1) 
  {
   
    myservo.write(pos);    
    delay(15); 
    Serial.println("--------------------------W^E^T");            
  }
}


void action()
{
  sec++;
}




 BLYNK_WRITE(V4) 
 {

  int button = param.asInt();
  Serial.print("IOT ON");
  Serial.println(button);
  buttonread=button;
}




 //--------------------------------------------------------ULTRA DRY -----------------------------------------
void ultra0()
 {
digitalWrite(trigPin0, LOW);
 delayMicroseconds(15);
 digitalWrite(trigPin0, HIGH);
 delayMicroseconds(10);
 digitalWrite(trigPin0, LOW);
 pinMode(echoPin0, INPUT);
 duration0 = pulseIn(echoPin0, HIGH);


cm0 = (duration0/2) / 29.1; 
 

 Serial.print("MAPING CM:");
 Serial.println(cm0);
 delay(300);

  // lcd.setCursor(11,1);
  // lcd.print("D:");
  //  if(cm<=9){lcd.print("00");lcd.print(cm);}
  //  else if(cm<=99){lcd.print("0");lcd.print(cm);}
  //  else if(cm<=999){lcd.print("");lcd.print(cm);}
  //  delay(100);

  }



 //--------------------------------------------------------ULTRA WET -----------------------------------------

void ultra1()
 {
digitalWrite(trigPin1, LOW);
 delayMicroseconds(15);
 digitalWrite(trigPin1, HIGH);
 delayMicroseconds(10);
 digitalWrite(trigPin1, LOW);
 pinMode(echoPin1, INPUT);
 duration1 = pulseIn(echoPin1, HIGH);


cm1 = (duration1/2) / 29.1; 
 

 Serial.print("MAPING CM1:");
 Serial.println(cm1);
 delay(300);

  // lcd.setCursor(11,1);
  // lcd.print("D:");
  //  if(cm1<=9){lcd.print("00");lcd.print(cm);}
  //  else if(cm<=99){lcd.print("0");lcd.print(cm);}
  //  else if(cm<=999){lcd.print("");lcd.print(cm);}
  //  delay(100);

  }















