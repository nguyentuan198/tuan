#include <DHT.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h> 
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

WiFiClient client;

String Address= "http://nguyentuantnt.000webhostapp.com/nienluan/add.php?";
String request_string;

long previousMillis = 0;
long currentMillis = 0;

HTTPClient http;

DHT dht(D3, DHT11);

LiquidCrystal_I2C lcd(0x27, 20, 4);


byte degree[8] = {
  0B01110,
  0B01010,
  0B01110,
  0B00000,
  0B00000,
  0B00000,
  0B00000,
  0B00000
};

 void bat_den(){
    for(int i =0; i<20; i++){   
     digitalWrite(D2,LOW);delay(50);
     digitalWrite(D2,HIGH);delay(50);
     }
}

void setup()
{  
    Serial.begin(115200);
  
  //Ket noi WiFi
    WiFi.disconnect();
   
    WiFi.begin("VivoTNT","23577777");
    while ((!(WiFi.status() == WL_CONNECTED))){
        delay(300);
    }

  //DHT11
    dht.begin();  

  //Den
    pinMode(D2, OUTPUT);digitalWrite(D2, HIGH);

  //Loa
  pinMode(D0, OUTPUT);digitalWrite(D0, HIGH);
  
  //LCD 
   Wire.begin(D5, D4); // SDA(D5) & SCL(D4)
  lcd.clear();
  lcd.init();
  lcd.backlight();
  lcd.begin(20, 4);
  lcd.print("Nhiet do:");
  lcd.setCursor(0,1);
  lcd.print("Do am:");
  lcd.createChar(1, degree);
    
}


void loop()
{
    float h = dht.readHumidity();   
    float t = dht.readTemperature();
     if(isnan(t) || isnan(h)){
      Serial.println("Khong doc duoc du lieu!!!");
   }else{
   /// --In ra man hinh
      lcd.setCursor(10,0);
        lcd.print(round(t));
        lcd.print(" ");
        lcd.write(1);
        lcd.print("C");

        lcd.setCursor(10,1);
        lcd.print(round(h));
        lcd.print(" %");

         Serial.print("Nhiet do:");
         Serial.print(t);
         Serial.println(" *C");
         Serial.print("Do am: ");
         Serial.print(h);
         Serial.println(" %");
         Serial.println();
         
   }

   /// --Dieu kien
    if(h>80 || h<60 || t>32 || t<25){
          bat_den();
          digitalWrite(D0,LOW);
    }else{
       digitalWrite(D0,HIGH);
    }
    
  /// --Goi du lieu len server
    currentMillis = millis();
 if (currentMillis - previousMillis > 30000){
        previousMillis = currentMillis;
    if (client.connect("nguyentuantnt.000webhostapp.com",80)) {
      request_string = Address;
      request_string += "key=";
      request_string += "M2DC2R4P4LSBG5ZI";
      
      request_string += "&";
      request_string += "t";
      request_string += "=";
      request_string += t; 
         
      request_string += "&";
      request_string += "h";
      request_string += "=";
      request_string += h;
      http.begin(request_string);
      http.GET();
      http.end();   
    }
 }
 delay(10000);
  
}
