#include <WiFi.h>

const char* ssid     = "TECNO CAMON 30S";
const char* password = "100@shanny";

int led1 = 33;
int led2 = 32;
int led3 = 27;
int ldrPin = 34;

bool autoMode = true;

WiFiServer server(80);

void setup() {
  Serial.begin(115200);

  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  pinMode(ldrPin, INPUT);

  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop() {
  int lightLevel = analogRead(ldrPin);
  Serial.print("LDR Light Level: ");
  Serial.println(lightLevel);
  delay(500);

  WiFiClient client = server.accept();
  if (client) {
    Serial.println("New Client.");
    String currentLine = "";

    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        Serial.write(c);

        if (c == '\n') {
          if (currentLine.length() == 0) {
            // Send styled HTML dashboard
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            client.println("<!DOCTYPE html><html><head><title>Faith Systems Lighting</title>");
            client.println("<style>");
            client.println("body{font-family:'Segoe UI',sans-serif;background:#1e1e2f;color:white;text-align:center;padding:20px;}");
            client.println("h1{color:#00d1b2;margin-bottom:10px;}");
            client.println(".card{background:#2e2e3f;border-radius:10px;max-width:600px;margin:20px auto;padding:20px;box-shadow:0 4px 10px rgba(0,0,0,0.3);}");
            client.println("button{padding:10px 18px;font-size:16px;border:none;border-radius:5px;cursor:pointer;margin:5px;}");
            client.println("button:hover{opacity:0.85;}");
            client.println(".on{background:#2ecc71;color:white;}");
            client.println(".off{background:#e74c3c;color:white;}");
            client.println(".switch{background:#3498db;color:white;}");
            client.println("footer{margin-top:30px;color:#aaa;font-size:14px;}");
            client.println("</style></head><body>");

            client.println("<div class='card'>");
            client.println("<h1>Faith Systems</h1>");
            client.println("<h2>Intelligent Lighting Dashboard</h2>");

            client.print("<p>Ambient Light Level: <strong>");
            client.print(lightLevel);
            client.println("</strong></p>");

            client.print("<p>Current Mode: <strong>");
            client.print(autoMode ? "Automatic" : "Manual");
            client.println("</strong></p>");

            if (autoMode) {
              client.println("<a href=\"/mode/manual\"><button class='switch'>Switch to Manual</button></a>");
            } else {
              client.println("<a href=\"/mode/auto\"><button class='switch'>Switch to Automatic</button></a>");
            }

            if (!autoMode) {
              client.println("<hr>");
              client.println("<p>LED 1: <a href=\"/led1/on\"><button class='on'>ON</button></a> <a href=\"/led1/off\"><button class='off'>OFF</button></a></p>");
              client.println("<p>LED 2: <a href=\"/led2/on\"><button class='on'>ON</button></a> <a href=\"/led2/off\"><button class='off'>OFF</button></a></p>");
              client.println("<p>LED 3: <a href=\"/led3/on\"><button class='on'>ON</button></a> <a href=\"/led3/off\"><button class='off'>OFF</button></a></p>");
            }

            client.println("</div>");
            client.println("<footer>&copy; 2025 Dushime Faith Rose | Team Innovators</footer>");
            client.println("</body></html>");
            client.println();
            break;
          } else {
            currentLine = "";
          }
        } else if (c != '\r') {
          currentLine += c;

          if (currentLine.endsWith("GET /mode/manual")) {
            autoMode = false;
            Serial.println("Switched to Manual Mode");
          }
          if (currentLine.endsWith("GET /mode/auto")) {
            autoMode = true;
            Serial.println("Switched to Automatic Mode");
          }

          if (!autoMode) {
            if (currentLine.endsWith("GET /led1/on")) digitalWrite(led1, HIGH);
            if (currentLine.endsWith("GET /led1/off")) digitalWrite(led1, LOW);
            if (currentLine.endsWith("GET /led2/on")) digitalWrite(led2, HIGH);
            if (currentLine.endsWith("GET /led2/off")) digitalWrite(led2, LOW);
            if (currentLine.endsWith("GET /led3/on")) digitalWrite(led3, HIGH);
            if (currentLine.endsWith("GET /led3/off")) digitalWrite(led3, LOW);
          }
        }
      }
    }

    client.stop();
    Serial.println("Client Disconnected.");
  }

  if (autoMode) {
    if (lightLevel < 1500) {
      digitalWrite(led1, HIGH);
      digitalWrite(led2, HIGH);
      digitalWrite(led3, HIGH);
    } else {
      digitalWrite(led1, LOW);
      digitalWrite(led2, LOW);
      digitalWrite(led3, LOW);
    }
  }
}
