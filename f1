#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <ESP32Servo.h>
 
const char* ssid = "hi";
const char* password = "12345678";
 
const String botToken = "7852824612:AAHGBiK0MGN1DivBwAHb7OgHWsmzC0D8dN0";
const String chatID = "1478316368";
 
Servo myservo;
int servoPin = 13;  // i use gpio pin num 13 for servo
 
String currentPassword = "1214";
int lastUpdateID = 0;
bool awaitingOldPassword = false;
bool awaitingNewPassword = false;
 
void connectToWiFi() {
  WiFi.begin(ssid, password);
  Serial.print("Connecting to Wi-Fi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("Connected to Wi-Fi!");
} 

void sendMessage(String message) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String url = "https://api.telegram.org/bot" + botToken + "/sendMessage";
    http.begin(url);
    http.addHeader("Content-Type", "application/json");

    String payload = "{\"chat_id\":\"" + chatID + "\",\"text\":\"" + message + "\"}";
    Serial.println("Sending payload: " + payload);

    int httpCode = http.POST(payload);
    if (httpCode > 0) {
      String response = http.getString();
      Serial.println("Message sent successfully! Response:");
      Serial.println(response);
    } else {
      Serial.print("Error on HTTP request: ");
      Serial.println(http.errorToString(httpCode));
    }
    http.end();
  } else {
    Serial.println("WiFi not connected");
  }
}
 
int getLastUpdateID() {
  HTTPClient http;
  String url = "https://api.telegram.org/bot" + botToken + "/getUpdates";
  http.begin(url);
  int httpCode = http.GET();
  String payload = http.getString();
  http.end();

  if (httpCode == 200) {
    StaticJsonDocument<1024> doc;
    deserializeJson(doc, payload);
    int maxUpdateID = 0;
    for (JsonVariant update : doc["result"].as<JsonArray>()) {
      maxUpdateID = max(maxUpdateID, update["update_id"].as<int>());
    }
    return maxUpdateID;
  }
  return 0;
}
 
String getUpdates(int lastUpdateID) {
  HTTPClient http;
  String url = "https://api.telegram.org/bot" + botToken + "/getUpdates?offset=" + String(lastUpdateID + 1) + "&limit=1";
  http.begin(url);
  int httpCode = http.GET();
  String payload = http.getString();
  http.end();

  if (httpCode == 200) {
    StaticJsonDocument<1024> doc;
    deserializeJson(doc, payload);
    if (doc["result"].size() > 0) {
      return doc["result"][0]["message"]["text"].as<String>();
    }
  }
  return "";
}
 
void sp(int position) {
  myservo.write(position);
  delay(10);
}

void s_open() {
  for (int pos = 0; pos <= 180; pos += 10) {
    sp(pos);
  }
}

void s_close() {
  for (int pos = 180; pos >= 0; pos -= 10) {
    sp(pos);
  }
}

void handlePasswordChange(String message) {
  if (awaitingOldPassword) {
    if (message == currentPassword) {
      sendMessage("Enter new password");
      awaitingOldPassword = false;
      awaitingNewPassword = true;
    } else {
      sendMessage("Wrong password. Try again.");
      awaitingOldPassword = false;
    }
  } else if (awaitingNewPassword) {
    currentPassword = message;
    sendMessage("Your new password is now set.");
    awaitingNewPassword = false;
  }
}

void handleCommands(String message) {
  if (awaitingOldPassword || awaitingNewPassword) {
    handlePasswordChange(message);
    return;
  }

  if (message == "open") {
    sendMessage("Enter password");
    delay(5000); 
    s_open();
    sendMessage("Lock opened");
  } else if (message == "lock") {
    sendMessage("Enter password");
    delay(5000);  
    s_close();
    sendMessage("Lock closed");
  } else if (message == "change password") {
    sendMessage("Enter current password");
    awaitingOldPassword = true;
  }
}

void setup() {
  Serial.begin(115200);
  myservo.attach(servoPin);
  connectToWiFi();
  sendMessage("System is online and ready");
}

void loop() {
  String message = getUpdates(lastUpdateID);

  if (message != "") {
    Serial.println("Received message: " + message);
    lastUpdateID = getLastUpdateID();

    handleCommands(message);
  }
  delay(1000);  
}
