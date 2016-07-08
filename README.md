// Wifi-Temp-sensor
//Code er mwyn creu synhwyrydd tymheredd gyda wifi



//trio cysylltu hwn efo'r synhwyrydd tymheredd. 

#include <ESP8266WiFi.h>  //cynnwys library o'r tu allan
#include <OneWire.h>
#include <DallasTemperature.h>

const char* ssid     = "Moelwyn"; //
const char* password = "scratcharetehvs5a"; //?

const char* host = "emoncms.org";

void setup() {           //setio'r system i fyny
  Serial.begin(115200);
  delay(10);

  // cysylltu gyda'r wifi

  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password); //angen password moelwyn ^^fyny fana, wedi ei wneud yn barod
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");  
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

int value = 0;


void loop() {
  delay(5000);   //hwn sydd yn achosi i'r signal yrru pob 5 eiliad
  ++value; //mae value yn cychwyn ar 0 ac yn mynd i fyny bob tro mae'r loop yn rhedeg
  //yma mae'n rhaid ysgrifennu rhaglen er mwyn mesur tymheredd gan ddefnyddio'r DS19B20

  Serial.print("connecting to ");
  Serial.println(host);
  
  // Use WiFiClient class to create TCP connections
  WiFiClient client;
  const int httpPort = 80;
  if (!client.connect(host, httpPort)) {
    Serial.println("connection failed");
    return;
  }






  
  // We now create a URI for the request
  String url = "/input/post.json?csv=100,200,300&apikey=a073d6c3c27fdf68aa3ac5b0beb239bf"; //emonCMS key- darganfod hwn oddi ar y wefan
  
  Serial.print("Requesting URL: ");
  Serial.println(url);
  
  // This will send the request to the server
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" + 
               "Connection: close\r\n\r\n");
  unsigned long timeout = millis();
  while (client.available() == 0) {
    if (millis() - timeout > 5000) {
      Serial.println(">>> Client Timeout !");
      client.stop();
      return;
    }
  }
  
  // Read all the lines of the reply from server and print them to Serial
  while(client.available()){
    String line = client.readStringUntil('\r');
    Serial.print(line);
  }
  
  Serial.println();
  Serial.println("closing connection");
}
