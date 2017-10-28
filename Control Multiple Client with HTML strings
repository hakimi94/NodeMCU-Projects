#include <ESP8266WiFi.h>
const char* ssid = "your-ssid";
const char* password = "your-password";
#define MAX_CLIENTS 10
#define MAX_LINE_LENGTH 50
// Create an instance of the server
// specify the port to listen on as an argument
WiFiServer server(80);
WiFiClient *clients[MAX_CLIENTS] = { NULL };
char inputs[MAX_CLIENTS][MAX_LINE_LENGTH] = { 0 };

void setup() {
  Serial.begin(115200);
  delay(10);

  // prepare GPIO2
  pinMode(2, OUTPUT);
  digitalWrite(2, 0);

  // Connect to WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  // Start the server
  server.begin();
  Serial.println("Server started");

  // Print the IP address
  Serial.println(WiFi.localIP());
}

void loop() {
  // Check if a new client has connected
  WiFiClient newClient = server.available();
  if (client) {
    Serial.println("new client");
    // Find the first unused space
    for (int i=0 ; i<MAX_CLIENTS ; ++i) {
        if (NULL == clients[i]) {
            clients[i] = new WiFiClient(newClient);
            break;
        }
     }
  }

  // Check whether each client has some data
  for (int i=0 ; i<MAX_CLIENTS ; ++i) {
    // If the client is in use, and has some data...
    if (NULL != clients[i] && clients[i]->available() ) {
      // Read the data 
      char newChar = clients[i]->read();

      // If we have the end of a string
      // (Using the test your code uses)
      if ('\r' == newChar) {
        // Blah blah, do whatever you want with inputs[i]

        // Empty the string for next time
        inputs[i][0] = NULL;

        // The flush that you had in your code - I'm not sure
        // why you want this, but here it is
        clients[i]->flush();

        // If you want to disconnect the client here, then do this:
        clients[i]->stop();
        delete clients[i];
        clients[i] = NULL;

      } else {
        // Add it to the string
        strcat(inputs[i], newChar);
        // IMPORTANT: Nothing stops this from overrunning the string and
        //            trashing your memory. You SHOULD guard against this.
        //            But I'm not going to do all your work for you :-)
      }
    }
  }

}
