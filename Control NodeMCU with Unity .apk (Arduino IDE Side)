#include <ESP8266WiFi.h>
//#include <Wire.h>
IPAddress ip(192,168,1,150);  //Node static IP
IPAddress gateway(192,168,1,1);
IPAddress subnet(255,255,255,0);
char* ssid = "Milk";
char* password = "huawei23";
int port = 1234;
bool State;
IPAddress IP_Code;
//WiFi.hostname("MyESP8266");
WiFiServer server(port);


String gpio5_state = "Off";
String gpio4_state = "Off";
int gpio5_pin = 5; //D1 - GreenLED
int gpio4_pin = 4; //D2 - RedLED


void setup() {
  //Wire.begin(); // sda, scl
  Serial.begin(115200);
  delay(200);

  // preparing GPIOs
  pinMode(gpio5_pin, OUTPUT);
  digitalWrite(gpio5_pin, LOW);
  pinMode(gpio4_pin, OUTPUT);
  digitalWrite(gpio4_pin, LOW);
  
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.disconnect();
  WiFi.begin(ssid, password);
  WiFi.config(ip, gateway, subnet);
  Serial.println();
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");       
  }
  Serial.println("WiFi connected");
  // Start the server
  server.begin();
  Serial.print("NodeMCU Connected at: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  WiFiClient client;
  client = server.available();
  if (client) {
      Serial.println(" in client if s");   ////////////////////////////////////////s
    // Wait until the client sends some data
    int j = 0;
    while ((!client.available()) && (j < 100)) {  delay(10);  j++;  }
    
      Serial.println(" in client if e");   ////////////////////////////////////////e
  }
  if (client.available())
  {
    
    Serial.println(" in client.available() if s");   ////////////////////////////////////////s
    String val = client.readString();
    
    //Serial.print("Request: ");
    //Serial.println(val);

    if(val.indexOf("Command = User&Pass >> User:Pass")>=0)
    {
      Serial.println("in first if for pass s");   ////////////////////////////////////////s
      
      Serial.println("ID is Correct");
      client.print("ID is correct");
      client.print("\n");
      IP_Code = client.remoteIP();
      val="";
      State = true;
      while(State)
      {
        if( client.remoteIP() != IP_Code &&  client.readString()!="")
        {
          client.print("Warning: another client is trying to send command !");
          Serial.println("Warning: another client is trying to send command");
        }
        else
        {
          Serial.println("in while loop  s");   ////////////////////////////////////////s
          val = client.readString();
          Serial.println(val);
          if(val.indexOf("Command = Close The Connection")>=0)
          {
            State = false;
            Serial.println("Connection is Closed from client Side");
            val = "";
          }
          else if(val.indexOf("Command = GreenLED >> On")>=0)
          {
            Serial.println("GPIO 5 On");
            gpio5_state = "On";
            digitalWrite(gpio5_pin, HIGH);
            client.println("gpio5_state:"+gpio5_state);
            val = "";
          }
          else if(val.indexOf("Command = GreenLED >> Off")>=0)
          {
            Serial.println("GPIO 5 Off");
            gpio5_state = "Off";
            digitalWrite(gpio5_pin, LOW);
            client.println("gpio5_state:"+gpio5_state);
            val = "";
          }
          else if(val.indexOf("Command = RedLED >> On")>=0)
          {
            Serial.println("GPIO 4 On");
            gpio4_state = "On";
            digitalWrite(gpio4_pin, HIGH);
            client.println("gpio4_state:"+gpio4_state);
            val = "";
          }
          else if(val.indexOf("Command = RedLED >> Off")>=0)
          {
            Serial.println("GPIO 4 Off");
            gpio4_state = "Off";
            digitalWrite(gpio4_pin, LOW);
            client.println("gpio4_state:"+gpio4_state);
            val = "";
          }
          
          Serial.println("in while loop  e");   ////////////////////////////////////////e
          if (!client)
          {
            State = false;
          }
        }
        
      }
        
        /*
        char* userInput = serialString();
        if (userInput!=NULL)
        {
          if(userInput=="Log Out")
          {
            State = false;
          }
        }*/
     
      Serial.println("in first if for pass e");   ////////////////////////////////////////e
    }else
    {
      Serial.println("User&Pass is inCorrect...");
      client.print("ID is not correct ");
      client.print("\n");
      Serial.println("Client Stopped");
    }

      Serial.println(" in client.available() if e");   ////////////////////////////////////////e
    //byte Data[10] = {};
    //val.getBytes(Data, val.length() + 1);    
    //Wire.beginTransmission(8);
    //Wire.write(Data, val.length() + 1);
    //Wire.endTransmission();
    client.flush();
    client.stop();
  }
  else client.stop();
}

char* serialString()
{
  static char str[21]; // For strings of max length=20
  if (!Serial.available()) return NULL;
  delay(64); // wait for all characters to arrive
  memset(str,0,sizeof(str)); // clear str
  byte count=0;
  while (Serial.available())
  {
    char c=Serial.read();
    if (c>=32 && count<sizeof(str)-1)
    {
      str[count]=c;
      count++;
    }
  }
  str[count]='\0'; // make it a zero terminated string
  return str;
}
