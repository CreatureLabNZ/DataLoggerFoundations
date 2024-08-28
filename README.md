#Data-logger foundation project

# Data-logger foundation project

## Session 1


<details>
  <Summary>Electronic component selection and assembly</Summary>
  
_What bits, why, and what are the strengths and limitations_

>Parts List:

|Part | Cost | Description | Datasheet |
|-----|------|-------------|-----------|
| Micro controller | 2.96NZD | [ESP32-C3 SuperMini WIFI/Bluetooth Development Board](https://www.aliexpress.com/item/1005005877531694.html) | Datasheet |
| Temp/Humidty/Pressure Sensor | 3.27NZD | [BME280 3.3V](https://www.aliexpress.com/item/1005006098059302.html) | Datasheet |
| Real Time Clock module | 1.64NZD | [DS3231 I2C](https://www.aliexpress.com/item/32822420722.html) | Datasheet |
| TF Card module | 0.69NZD | [TF Card Memory Shield Module SPI](https://www.aliexpress.com/item/32346771288.html) | Datasheet |
| TF memory card | 3.00NZD | [Off-brand 16GB TF/micro SD Card](https://www.aliexpress.com/item/1005001617961938.html?algo_exp_id=3e8b6944-24e4-4192-9359-7344f5438bc5-4&utparam-url=scene%3Asearch%7Cquery_from%3A) | Datasheet |
| Prototyping PCB | 2.10NZ | [DKS-SOLDERBREAD-02](https://www.digikey.co.nz/en/products/detail/digikey/DKS-SOLDERBREAD-02/15970925) | Datasheet |
| Battery holder | 0.37NZD | [Single 18650](https://www.aliexpress.com/item/32580480645.html) | Datasheet |
| Battery 18650 | 12.49NZD | [Molicel M35A 18650 3500mAh 10A](https://cell-supply.co.nz/product/molicel-m35a-18650-3500mah-10a-battery/) | Datasheet |
| Charging Board | 0.51NZD | [TP4056](https://www.aliexpress.com/item/32467578996.html) | [Datasheet] |
| LDO Regulator | 0.76NZD | [MCP1700-3302E](https://www.digikey.co.nz/en/products/detail/microchip-technology/MCP1700-3302E-TO/652680?s=N4IgTCBcDaILIGEAKBGA7ABgwWgMy4zAFEQBdAXyA) | [Datasheet]((https://ww1.microchip.com/downloads/en/DeviceDoc/MCP1700-Data-Sheet-20001826F.pdf)) |
| 100uF Electrolytic Capacitor | 0.13NZD | [CAP ALUM 100UF 20% 10V RADIAL TH](https://www.digikey.com/en/products/detail/chinsan-elite/PF1A101MP20511F3U/16497042?s=N4IgTCBcDaICwEYCcCC0AFAYgggggDAgLLpj4CsCCmAzAKoDCAKqgHIAiIAugL5A) | Datasheet |
| 100nF Ceramic Capacitor | 0.09NZD | [CAP CER 0.1UF 50V X7R RADIAL](https://www.digikey.com/en/products/detail/kemet/C320C104M5R5TA/3726028?s=N4IgTCBcDaIMwE4EFoEHY0DZkDkAiIAugL5A) | Datasheet |
| 27K Ohm resistor | 0.04NZD | [RES 27K OHM 5% 1/4W AXIAL](https://www.digikey.com/en/products/detail/stackpole-electronics-inc/CFM14JT27K0/1742158?s=N4IgTCBcDaIMpgOwGkCKBhAKgWgHIBEQBdAXyA) | Datasheet |
| 100K Ohm resistor | 0.03NZD | [RES 100K OHM 5% 1/8W AXIAL](https://www.digikey.com/en/products/detail/stackpole-electronics-inc/CF18JT100K/1741564?s=N4IgTCBcDaIMIDECMAOAUgFSQBmwaTgwFoA5AERAF0BfIA) | Datasheet |


![Breadboard overview](https://github.com/user-attachments/assets/5d93920e-aa86-407c-98df-cfda81e5d644)
</Details>

## Session 2


<details>
  <Summary>Software setup and configuration</Summary>  
_Setting up our boards to read sensors, format and save data, and communication and user interface design._

Minimum working example to set up over-the-air software uploads and blink and LED from a web interface
  
```
#include <WiFi.h>
#include <ESPmDNS.h>
#include <WiFiUdp.h>
#include <ArduinoOTA.h>
#include <WebServer.h>

const char* ssid = "WIFI NETWORK NAME"; ## Wifi name
const char* password = "PASSWORD"; ## Wifi password

IPAddress local_ip(192,168,1,222);
IPAddress gateway(192,168,1,254);
IPAddress subnet(255,255,255,0);

WebServer server(80);

//variabls for blinking an LED with Millis

const int led = 4; // ESP32 Pin to which onboard LED is connected
// unsigned long previousMillis = 0;  // will store last time LED was updated
// const long interval = 1000;  // interval at which to blink (milliseconds)
bool ledState = LOW;  // ledState used to set the LED


void setup() {

  pinMode(led, OUTPUT);  
  Serial.begin(115200);
  Serial.println("Booting");

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
      Serial.println("Connection Failed! Rebooting...");
      delay(5000);
      ESP.restart();
    }

  ArduinoOTA.setPort(3232); // Port defaults to 3232
  ArduinoOTA.setHostname("creature_log");// Hostname defaults to esp3232-[MAC] ## Need to set a unique hostname for your device!
  ArduinoOTA.setPassword("admin"); // No authentication by default
  
  // MD5(admin) = 21232f297a57a5a743894a0e4a801fc3  // Password can be set with it's md5 value as well
  // ArduinoOTA.setPasswordHash("21232f297a57a5a743894a0e4a801fc3");
  ArduinoOTA
    .onStart([]() {
      String type;
      if (ArduinoOTA.getCommand() == U_FLASH)
        type = "sketch";
      else // U_SPIFFS
        type = "filesystem";
      // NOTE: if updating SPIFFS this would be the place to unmount SPIFFS using SPIFFS.end()
      Serial.println("Start updating " + type);
    })
    .onEnd([]() {
      Serial.println("\nEnd");
    })
    .onProgress([](unsigned int progress, unsigned int total) {
      Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
    })
    .onError([](ota_error_t error) {
      Serial.printf("Error[%u]: ", error);
      if (error == OTA_AUTH_ERROR) Serial.println("Auth Failed");
      else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
      else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
      else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
      else if (error == OTA_END_ERROR) Serial.println("End Failed");
    });

  ArduinoOTA.begin();

  Serial.println("Ready");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

// The first line of the code snippet below, for example, 
// indicates that when a server receives an HTTP request on the root (/) path, 
// it will call the handle_OnConnect() function. 
// It is important to note that the URL specified is a relative path.
// Similarly, we must specify two more URLs to handle the two states of the led.
// if the client requests a URL that isn’t specified with server.on() . 
// It should give a 404 error (Page Not Found) as a response. To accomplish this, 
// we use the server.onNotFound() method.
server.on("/", handle_OnConnect);
server.on("/ledOn", handle_ledOn);
server.on("/ledOff", handle_ledOff);
server.onNotFound(handle_NotFound);

//Now, to start the server, we call the server object’s begin() method.
server.begin();
Serial.println("HTTP server started");

}

void loop() {

  ArduinoOTA.handle();  

  // Actual incoming HTTP requests are handled in the loop function. 
  // For this, we use the server object’s handleClient() method. 
  // We also change the state of LEDs based on the request.
  // Now we must write the handle_OnConnect() function, which we previously attached to the root (/) URL with server.on. 
  // We begin this function by setting the status of both LEDs to LOW (initial state of LEDs) and printing it on the serial monitor.
  // We use the send method to respond to an HTTP request. 
  server.handleClient();
  if(ledState)
  {digitalWrite(led, HIGH);}
  else
  {digitalWrite(led, LOW);}

}

  // Although the method can be called with a number of different arguments, the simplest form requires the HTTP response code, 
  // the content type, and the content.
  // The first parameter we pass to the send method is the code 200 (one of the HTTP status codes), 
  // which corresponds to the OK response. Then we specify the content type as “text/html,” 
  // and finally we pass the SendHTML() custom function, which generates a dynamic HTML page with the LED status.
  void handle_OnConnect() {
  ledState = LOW;
  Serial.println("GPIO4 Status: OFF");
  server.send(200, "text/html", SendHTML(ledState)); 
  }

  // Similarly, we write three more functions to handle LED ON/OFF requests and the 404 Error page.
  void handle_ledOn() {
  ledState = HIGH;
  Serial.println("GPIO4 Status: ON");
  server.send(200, "text/html", SendHTML(true)); 
  }
  void handle_ledOff() {
  ledState = LOW;
  Serial.println("GPIO4 Status: OFF");
  server.send(200, "text/html", SendHTML(false)); 
  }
  void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
  }

  // ! -- Displaying the HTML Web Page -- !
  // Whenever the ESP32 web server receives a request from a web client, the sendHTML() function generates a web page. 
  // It simply concatenates HTML code into a long string and returns to the server.send()
  // The function uses the status of the LEDs as a parameter to generate HTML content dynamically.
  // The first text you should always send is the <!DOCTYPE> declaration, which indicates that we’re sending HTML code.
  String SendHTML(uint8_t ledStat){

  String ptr = "<!DOCTYPE html> <html>\n";

  ptr +="<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=no\">\n";
  ptr +="<title>LED Control</title>\n";
  ptr +="<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}\n";
  ptr +="body{margin-top: 50px;} h1 {color: #444444;margin: 50px auto 30px;} h3 {color: #444444;margin-bottom: 50px;}\n";
  ptr +="p {font-size: 14px;color: #888;margin-bottom: 10px;}\n";
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
  if(ledStat)
   {ptr +="<p>LED Status: ON</p><a class=\"button button-off\" href=\"/ledOff\">OFF</a>\n";}
  else
   {ptr +="<p>LED Status: OFF</p><a class=\"button button-on\" href=\"/ledOn\">ON</a>\n";}

  ptr +="</body>\n";
  ptr +="</html>\n";
  return ptr;
  }
  ```



