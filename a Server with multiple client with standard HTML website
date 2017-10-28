/*
  SDWebServer - Example WebServer with SD Card backend for esp8266

  Copyright (c) 2015 Hristo Gochkov. All rights reserved.
  This file is part of the ESP8266WebServer library for Arduino environment.

  This library is free software; you can redistribute it and/or
  modify it under the terms of the GNU Lesser General Public
  License as published by the Free Software Foundation; either
  version 2.1 of the License, or (at your option) any later version.

  This library is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
  Lesser General Public License for more details.

  You should have received a copy of the GNU Lesser General Public
  License along with this library; if not, write to the Free Software
  Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA

  Have a FAT Formatted SD Card connected to the SPI port of the ESP8266
  The web root is the SD Card root folder
  File extensions with more than 3 charecters are not supported by the SD Library
  File Names longer than 8 charecters will be truncated by the SD library, so keep filenames shorter
  index.htm is the default index (works on subfolders as well)

  upload the contents of SdRoot to the root of the SDcard and access the editor by going to http://esp8266sd.local/edit

*/
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>
#include <SPI.h>
#include <SD.h>
#include <Wire.h>

#define DBG_OUTPUT_PORT Serial

extern "C"
{
#include "user_interface.h"
}
// Only USE if nothing connected to ADC !!!!!!!!!
// ADC_MODE(ADC_VCC);
const char* ssid = "Your-SSID";
const char* password = "Your-PASSWORD";
const char* host = "ESP-MPU-Gateway";
IPAddress apIP(192, 168, 4, 1);
ESP8266WebServer server(80);

const char* _passcode = "1234";//note: the field "passcode in the html form "frm1" must match the passcode lenght
char passcodeOK='0';//start Code is blank...
String readString;
int analogReadOut;
int dispcont = 0;

// Create placeholders fot MPU6050..........
int16_t AcX, AcY, AcZ, Tmpt, Tmp, GyX, GyY, GyZ, AcXo, AcYo, AcZo, GyXo, GyYo, GyZo;
#define serverPort 80
// define (map) GPIO to HTML Fields (buttons/Indicators), note: not using 0 as base index deliberately
#define input1 0
#define output1 2


static bool hasSD = false;
File uploadFile;


void returnOK() {
  server.sendHeader("Connection", "close");
  server.sendHeader("Access-Control-Allow-Origin", "*");
  server.send(200, "text/plain", "");
}

void returnOKs() {
  server.sendHeader("Connection", "refresh,20");
  server.sendHeader("Access-Control-Allow-Origin", "*");
  server.send(200, "text/plain", "");
}

void returnFail(String msg) {
  server.sendHeader("Connection", "close");
  server.sendHeader("Access-Control-Allow-Origin", "*");
  server.send(500, "text/plain", msg + "\r\n");
}

bool loadFromSdCard(String path){
  String dataType = "text/plain";
  if(path.endsWith("/")) path += "index.htm";

  if(path.endsWith(".src")) path = path.substring(0, path.lastIndexOf("."));
  else if(path.endsWith(".htm")) dataType = "text/html";
  else if(path.endsWith(".css")) dataType = "text/css";
  else if(path.endsWith(".js")) dataType = "application/javascript";
  else if(path.endsWith(".png")) dataType = "image/png";
  else if(path.endsWith(".gif")) dataType = "image/gif";
  else if(path.endsWith(".jpg")) dataType = "image/jpeg";
  else if(path.endsWith(".ico")) dataType = "image/x-icon";
  else if(path.endsWith(".xml")) dataType = "text/xml";
  else if(path.endsWith(".pdf")) dataType = "application/pdf";
  else if(path.endsWith(".zip")) dataType = "application/zip";

  File dataFile = SD.open(path.c_str());
  if(dataFile.isDirectory()){
    path += "/index.htm";
    dataType = "text/html";
    dataFile = SD.open(path.c_str());
  }

  if (!dataFile)
    return false;

  if (server.hasArg("download")) dataType = "application/octet-stream";

  if (server.streamFile(dataFile, dataType) != dataFile.size()) {
    DBG_OUTPUT_PORT.println("Sent less data than expected!");
  }

  dataFile.close();
  return true;
}

void handleFileUpload(){
  if(server.uri() != "/edit") return;
  HTTPUpload& upload = server.upload();
  if(upload.status == UPLOAD_FILE_START){
    if(SD.exists((char *)upload.filename.c_str())) SD.remove((char *)upload.filename.c_str());
    uploadFile = SD.open(upload.filename.c_str(), FILE_WRITE);
    DBG_OUTPUT_PORT.print("Upload: START, filename: "); DBG_OUTPUT_PORT.println(upload.filename);
  } else if(upload.status == UPLOAD_FILE_WRITE){
    if(uploadFile) uploadFile.write(upload.buf, upload.currentSize);
    DBG_OUTPUT_PORT.print("Upload: WRITE, Bytes: "); DBG_OUTPUT_PORT.println(upload.currentSize);
  } else if(upload.status == UPLOAD_FILE_END){
    if(uploadFile) uploadFile.close();
    DBG_OUTPUT_PORT.print("Upload: END, Size: "); DBG_OUTPUT_PORT.println(upload.totalSize);
  }
}

void deleteRecursive(String path){
  File file = SD.open((char *)path.c_str());
  if(!file.isDirectory()){
    file.close();
    SD.remove((char *)path.c_str());
    return;
  }

  file.rewindDirectory();
  while(true) {
    File entry = file.openNextFile();
    if (!entry) break;
    String entryPath = path + "/" +entry.name();
    if(entry.isDirectory()){
      entry.close();
      deleteRecursive(entryPath);
    } else {
      entry.close();
      SD.remove((char *)entryPath.c_str());
    }
    yield();
  }

  SD.rmdir((char *)path.c_str());
  file.close();
}

void handleDelete(){
  if(server.args() == 0) return returnFail("BAD ARGS");
  String path = server.arg(0);
  if(path == "/" || !SD.exists((char *)path.c_str())) {
    returnFail("BAD PATH");
    return;
  }
  deleteRecursive(path);
  returnOK();
}

void handleCreate(){
  if(server.args() == 0) return returnFail("BAD ARGS");
  String path = server.arg(0);
  if(path == "/" || SD.exists((char *)path.c_str())) {
    returnFail("BAD PATH");
    return;
  }

  if(path.indexOf('.') > 0){
    File file = SD.open((char *)path.c_str(), FILE_WRITE);
    if(file){
      file.write((const char *)0);
      file.close();
    }
  } else {
    SD.mkdir((char *)path.c_str());
  }
  returnOK();
}

void printDirectory() {
  if(!server.hasArg("dir")) return returnFail("BAD ARGS");
  String path = server.arg("dir");
  if(path != "/" && !SD.exists((char *)path.c_str())) return returnFail("BAD PATH");
  File dir = SD.open((char *)path.c_str());
  path = String();
  if(!dir.isDirectory()){
    dir.close();
    return returnFail("NOT DIR");
  }
  dir.rewindDirectory();
  server.setContentLength(CONTENT_LENGTH_UNKNOWN);
  server.send(200, "text/json", "");
  WiFiClient client = server.client();

  server.sendContent("[");
  for (int cnt = 0; true; ++cnt) {
    File entry = dir.openNextFile();
    if (!entry)
    break;

    String output;
    if (cnt > 0)
      output = ',';

    output += "{\"type\":\"";
    output += (entry.isDirectory()) ? "dir" : "file";
    output += "\",\"name\":\"";
    output += entry.name();
    output += "\"";
    output += "}";
    server.sendContent(output);
    entry.close();
 }
 server.sendContent("]");
 dir.close();
}

void handleNotFound(){
  if(hasSD && loadFromSdCard(server.uri())) return;
  String message = "SDCARD Not Detected\n\n";
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET)?"GET":"POST";
  message += "\nArguments: ";
  message += server.args();
  message += "\n";
  for (uint8_t i=0; i<server.args(); i++){
    message += " NAME:"+server.argName(i) + "\n VALUE:" + server.arg(i) + "\n";
  }
  server.send(404, "text/plain", message);
  DBG_OUTPUT_PORT.print(message);
}

void setup(void){
  DBG_OUTPUT_PORT.begin(115200);
  Wire.begin();
  Wire.beginTransmission(0x68);                       // 0x68 I2C address of the MPU-6050
  Wire.write(0x6B);                                   // PWR_MGMT_1 register
  Wire.write(0);                                      // set to zero (wakes up the MPU-6050)
  Wire.endTransmission(true);
  DBG_OUTPUT_PORT.setDebugOutput(true);
  DBG_OUTPUT_PORT.print("\n");
  WiFi.mode(WIFI_AP_STA);
  WiFi.softAPConfig(apIP, apIP, IPAddress(255, 255, 255, 0));
  WiFi.softAP("ESP-MPU-Gateway-1","");
  WiFi.begin(ssid, password);
  DBG_OUTPUT_PORT.print("Connecting to ");
  DBG_OUTPUT_PORT.println(ssid);

  // Wait for connection
  uint8_t i = 0;
  while (WiFi.status() != WL_CONNECTED && i++ < 20) {//wait 10 seconds
    delay(500);
  }
  if(i == 21){
    DBG_OUTPUT_PORT.print("Could not connect to");
    DBG_OUTPUT_PORT.println(ssid);
    while(1) delay(500);
  }
  DBG_OUTPUT_PORT.print("Connected! IP address: ");
  DBG_OUTPUT_PORT.println(WiFi.localIP());

  if (MDNS.begin(host)) {
    MDNS.addService("http", "tcp", 80);
    DBG_OUTPUT_PORT.println("MDNS responder started");
    DBG_OUTPUT_PORT.print("You can now connect to http://");
    DBG_OUTPUT_PORT.print(host);
    DBG_OUTPUT_PORT.println(".local");
  }


  server.on("/list", HTTP_GET, printDirectory);
  server.on("/edit", HTTP_DELETE, handleDelete);
  server.on("/edit", HTTP_PUT, handleCreate);
  server.on("/edit", HTTP_POST, [](){ returnOK(); });
  server.on("/diag", HTTP_GET, diags);
  server.on("/fall", HTTP_GET, falld);
  server.on("/sensorupdrq",  HTTP_POST,  [](){ readInputs(); }); 
  server.onNotFound(handleNotFound);
  server.onFileUpload(handleFileUpload);
//  ESP.wdtEnable(WDTO_4S);
  server.begin();
  DBG_OUTPUT_PORT.println("HTTP server started");

  if (SD.begin(SS)){
     DBG_OUTPUT_PORT.println("SD Card initialized.");
     hasSD = true;
  }
}

// send back GPIO Data Refresh
void readInputs() {
  WiFiClient client = server.client();
  client.println("\r\n<p>");//dont miss to wrap the request with some tag, mandatory for ajax to work properly
//  Do not use if LDR Fitted
//  analogReadOut = ESP.getVcc(); 
delay(50);
 analogReadOut = analogRead(A0);
delay(50);
// Accelerometer 0x43
// Gyro 0x3B
  Wire.beginTransmission(0x68);
  Wire.write(0x3B);                                  // starting with register 0x43 (GYRO_XOUT_H)
  Wire.endTransmission(false);
  Wire.requestFrom(0x68, 14, true);                  // request a total of 6 registers or What you need 
  GyX = Wire.read() << 8 | Wire.read();
  GyY = Wire.read() << 8 | Wire.read();
  GyZ = Wire.read() << 8 | Wire.read();
  Tmp = Wire.read() << 8 | Wire.read(); // 0x41 (TEMP_OUT_H) & 0x42 (TEMP_OUT_L)
  AcX = Wire.read() << 8 | Wire.read();
  AcY = Wire.read() << 8 | Wire.read();
  AcZ = Wire.read() << 8 | Wire.read();
  Wire.endTransmission(true);
  Tmpt = Tmp/340.00+36.53;   
  if (analogReadOut < 30){
    client.print("<p>Light Leve is Low:<span style='background-color:#FF0000; font-size:18pt'>");
    client.print(analogReadOut);
    client.println("</span></p>");
  }else{
    client.print("<p>Light Level is:<span style='background-color:#00FF00; font-size:18pt'>");
    client.print(analogReadOut);
    client.println("</span></p>");
  }
  //________________________________Secured Data Start______________________________________________
  // this will ensure sensor condition or user verification before displaying this section
    if ((digitalRead(input1))&&(passcodeOK== '0')) {   
      }else{
    client.println("<BR>Attitude is relative to facing the Device LED<BR><BR>");
    if (Tmpt < 20){
    client.print("<p>Body Temperature:<span style='background-color:#FF0000; font-size:18pt'>");
    client.print(Tmpt);
    client.println("C</span></p>");
  }else{
    client.print("<p>Body Temperature:<span style='background-color:#00FF00; font-size:18pt'>");
    client.print(Tmpt);
    client.println("C</span></p>");
  }
  GyXo = GyX/=100;
  if (GyXo < 0){
    client.print("<p>Device is Upside Down :<span style='background-color:#FF0000; font-size:18pt'>");
    client.print(GyXo);
    client.println("</span></p>");
  }else{
    client.print("<p>Right Side Up 180 is Verticle:<span style='background-color:#00FF00; font-size:18pt'>");
    client.print(GyXo);
    client.println("</span></p>");
  }
  GyYo = GyY/=100;
  if (GyYo < 0){
    client.print("<p>Left Inclination:<span style='background-color:#0000FF; font-size:18pt'>");
    client.print(GyYo);
    client.println("</span></p>");
  }else{
    client.print("<p>Right Inclination:<span style='background-color:#00FF00; font-size:18pt'>");
    client.print(GyYo);
    client.println("</span></p>");
  }
  GyZo = GyZ/=100;
  if (GyZo < 0){
    client.print("<p>Backward Inclination:<span style='background-color:#0000FF; font-size:18pt'>");
    client.print(GyZo);
    client.println("</span></p>");
  }else{
    client.print("<p>Forward Inclination:<span style='background-color:#00FF00; font-size:18pt'>");
    client.print(GyZo);
    client.println("</span></p>");
  }
    passcodeOK='0';
  }
 // ______________________________________secured data end___________________________________________
  client.println("<br />");
  if (digitalRead(input1)) {
    client.println("<p>Sensor ONE is: <span style='background-color:#00FF00; font-size:18pt'>ALL OK</span></p>");
  }else{
    client.println("<p>Sensor ONE is: <span style='background-color:#FF0000; font-size:18pt'>Assistance</span></p>");
  }
  client.println("<br />");
/*  if (digitalRead(input2)) {
    client.println("<p>Sensor TWO is: <span style='background-color:#00FF00; font-size:18pt'>ALL OK</span></p>");
  }else{
    client.println("<p>Sensor TWO is: <span style='background-color:#FF0000; font-size:18pt'>Assistance</span></p>");
  }
    client.println("<br />");
  if (digitalRead(input3)) {
    client.println("<p>Flash Button is: <span style='background-color:#00FF00; font-size:18pt'> ALL OK</span></p>");
  }else{
    client.println("<p>Flash Button is: <span style='background-color:#0000FF; font-size:18pt'>Assistance</span></p>");
  }*/
  client.println("</p>");//this tag completes the request wrap
}

void falld(){
  WiFiClient client = server.client();       
            client.println("HTTP/1.1 200 OK\r\nContent-Type: text/html");
            client.println("\r\n<HTML>");
            client.println("<HEAD>");
            client.println("<meta name=\"viewport\" content=\"width=380\">");
// Important, edit the Following line to match your CSS style sheet location, I suggest you to host at your own controlled server
            client.println("<link rel='stylesheet' type='text/css' href='/style.css' />");
            client.println("<TITLE>DDT Fall Detector 1</TITLE>");
               if (server.hasArg("passcode")){
                 if (server.arg(0)==_passcode){// This is the password compare from the returned data
                passcodeOK='1';
            }else{
                passcodeOK='0';
            }
               }
            // simple javascritp code to intercept anchor clientick and trick HREF field with the Button (or action) ID and add the password from the input field
            client.println("\r\n<script>\r\nfunction SndCommand( obx,swtchNO ){ \r\nobx.href=\"button\"+swtchNO+\"&passcode=\"+document.getElementById(\"frm1\").elements[0].value;\r\n}\r\n</script>\r\n");
            // not so simple script to allow sensor data to be automatically updated
            client.println("\r\n<script>\r\nfunction GetSensorUpd() {\r\nnocache = \"&nocache=\" + Math.random() * 1000000;");
            client.println("var ajaxreq = new XMLHttpRequest();");
            client.println("ajaxreq.open(\"GET\", \"sensorupdrq\" + nocache, true);");
            client.println("ajaxreq.onreadystatechange = function() {");
            client.println("if (ajaxreq.readyState == 4 && ajaxreq.status == 200) {");
            client.println("document.getElementById(\"readX\").innerHTML = ajaxreq.responseText;");
            client.println("}\r\n}\r\najaxreq.send();");
            client.println("setTimeout('GetSensorUpd()', 5000);\r\n}\r\n</script>\r\n");//updates each half second

            client.println("</HEAD>");
//            client.println("<BODY onload=\"GetSensorUpd()\">");//comment this out if you do not want automatic sensor update
            client.println("<BODY>");//uncomment this if you do not want auto update
            client.println("<H1>Demo to Display the Motion Data of MPU6050 Securely<br />With<br />  Open Sensor/Switch Reads on the MCU</H1>");
            client.println("<br />"); 
            client.print("<div id=\"passx\"><form id=\"frm1\">Please Input Password<BR>To Unlock Data:<BR><input name=\"passcode\" size=\"5\" maxlength=\"4\" type=\"password\" value=\"");
            if (passcodeOK== '1') client.print(_passcode);
            client.println("\" style='font-size:24pt'></form></div>");
               
            client.print("<div id=\"readX\">");
            readInputs(); 
            client.println("</div>");

           
            // first ON/OFF Buttons pair
            client.println("<BR /><a href=\"\" onclick=\"SndCommand(this,11)\">Output 1 On</a>");
            //SndCommand is a javascript code running on the clientiet (aka ajax) Parameters are outputIDx and State
            client.println("<a href=\"\" onclick=\"SndCommand(this,10)\">Output 1 Off</a><br /><br />");   
           
            // then repeat for every gpio as output to control
           
           // client.println("<a href=\"\" onclick=\"SndCommand(this,21)\">Output 2 On</a>");
           // client.println("<a href=\"\" onclick=\"SndCommand(this,20)\">Output 2 Off</a><br /><br />"); 
 
           // client.println("<a href=\"\" onclick=\"SndCommand(this,31)\">Output 3 On</a>");
           // client.println("<a href=\"\" onclick=\"SndCommand(this,30)\">Output 3 Off</a><br /><br />"); 
                       
            client.println("<BR>the use of static html with data refreshed dynamically </BODY>");
            client.println("</HTML>");
            //Processing client request
            // Button0
            if (readString.indexOf("button11") >0 && passcodeOK== '1'){
            }
            if (readString.indexOf("button10")>0 && passcodeOK== '1'){
            }
            if (readString.indexOf("button21") >0 && passcodeOK== '1'){
            }
            if (readString.indexOf("button20")>0 && passcodeOK== '1'){
            }
            if (readString.indexOf("button31") >0 && passcodeOK== '1'){
            }
            if (readString.indexOf("button30")>0 && passcodeOK== '1'){

            }     
          }


void diags(){
  WiFiClient client = server.client();
//   float servolt1 = ESP.getVcc();
//   float servolt1 = analogRead(0);
     unsigned long  spdcount = ESP.getCycleCount();
     delay(1);
     unsigned long  spdcount1 = ESP.getCycleCount();
     unsigned long  speedcnt = spdcount1-spdcount;
     FlashMode_t ideMode = ESP.getFlashChipMode();
                                String duration1 = " ";
                                int hr,mn,st;
                                st = millis() / 1000;
                                mn = st / 60;
                                hr = st / 3600;
                                st = st - mn * 60;
                                mn = mn - hr * 60;
                                if (hr<10) {duration1 += ("0");}
                                duration1 += (hr);
                                duration1 += (":");
                                if (mn<10) {duration1 += ("0");}
                                duration1 += (mn);
                                duration1 += (":");
                                if (st<10) {duration1 += ("0");}
                                duration1 += (st);
                         String diagdat="";     
                                diagdat=("<html><head><title>Mini-Server-Diagnostics</title><link href=\"/style.css\" rel=\"stylesheet\" type=\"text/css\" media=\"screen\" /></head><body>");
                                diagdat+=("<div id=\"menu\">\n<ul>\n<li class=\"current_page_item\"><a href=\"#\">Diagnostics</a></li>\n<li><a href=\"/blog/\">Blog</a></li>\n<li><a href=\"/photos/\">Photos</a></li>\n<li>");
                                diagdat+=("<a href=\"/about/\">About</a></li>\n<li><a href=\"/links/\">Links</a></li>\n<li><a href=\"/contact/\">Contact</a></li>\n<li><a href=\"/diag\">Diagnostics</a></li>\n</ul>\n</div>\n<font color=\"#000000\"><body bgcolor=\"#a0dFfe\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=yes\">");
                                diagdat+=("<BR><h1><BR>Mini Server Gateway<BR>SDK Diagnostic Information</h1>");
                                diagdat+=("<BR><a href=\"/edit/\">Editor Page</a><BR>");
                                diagdat+="<BR>  WiFi Station Hostname = ";
                                diagdat+=wifi_station_get_hostname();
                                diagdat+="<BR>  Free RAM = ";
                                client.print(diagdat);
                                client.print((uint32_t)system_get_free_heap_size()/1024);
                                diagdat=" KBytes<BR>  SDK Version = ";                                 
                                diagdat+=ESP.getSdkVersion();
                                diagdat+="<BR>  Boot Version = ";
                                diagdat+=ESP.getBootVersion();
                                diagdat+="<BR>  Free Sketch Space  = ";
                                diagdat+=ESP.getFreeSketchSpace()/1024;
                                diagdat+=" KBytes<BR>  Sketch Size  = ";
                                diagdat+=ESP.getSketchSize()/1024;
                                diagdat+=" KBytes<BR>";
                                client.print(diagdat);
                                client.printf("  Flash Chip id = %08X\n", ESP.getFlashChipId());
                                client.print("<BR>");
                                client.printf("  Flash Chip Mode = %s\n", (ideMode == FM_QIO ? "QIO" : ideMode == FM_QOUT ? "QOUT" : ideMode == FM_DIO ? "DIO" : ideMode == FM_DOUT ? "DOUT" : "UNKNOWN"));
                                diagdat="<BR>  Flash Size By ID = ";
                                diagdat+=ESP.getFlashChipRealSize()/1024;
                                diagdat+=" KBytes<BR>  Flash Size (IDE) = ";
                                diagdat+=ESP.getFlashChipSize()/1024;
                                diagdat+=" KBytes<BR>  Flash Speed = ";
                                diagdat+=ESP.getFlashChipSpeed()/1000000;
                                diagdat+=" MHz<BR>  ESP8266 CPU Speed = ";
                                diagdat+=ESP.getCpuFreqMHz();
                                diagdat+=" MHz<BR>";
                                client.print(diagdat);
                                client.printf("  ESP8266 Chip id = %08X\n", ESP.getChipId());
                                diagdat="<BR>  System Instruction Cycles Per Second = ";
                                diagdat+=speedcnt*1000; 
                                diagdat+="<BR>  Last System Restart Reason = ";
                                diagdat+=ESP.getResetInfo();                                                             
                                //diagdat+="<BR>  System VCC = ";
                                //diagdat+=servolt1/1000, 3;
                                diagdat+="<BR>  System Uptime = ";
                                diagdat+=duration1;
                                client.print(diagdat);
                                client.print("<BR><FONT SIZE=-2>environmental.monitor.log@gmail.com<BR><FONT SIZE=-1>ESP8266-12  Mini Server With SD<BR><FONT SIZE=-2>Compiled Using ver. 1.6.5-1160-gef26c5f, built on Sep 30, 2015<BR>");
                                client.println("<IMG SRC=\"/images/esp-12.jpg\" WIDTH=\"320\" HEIGHT=\"326\" BORDER=\"1\"></body></html>");
                                diagdat = "";
                                client.stop();
                                duration1 = ""; 
}
void loop(void){

  server.handleClient();
}
