#include <Wire.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include <time.h>

// SDA and SCL pins for the decibel meter
#define I2C_SDA     22
#define I2C_SCL     23
#define DBM_ADDR    0x48
#define DBM_REG_DECIBEL 0x0A

const char* ssid = "BakersDozen";
const char* password = "MiloKinja";
const char* localMqttServer = "172.16.0.202";  // Raspberry Pi's IP Address
const char* thingspeakMqttServer = "mqtt3.thingspeak.com";
const int mqttPort = 1883;
const char* mqttTopic = "prod/decibel/#";
const char* tsPassword = "odTDEYCrxbWLhnSvluTkbMdK";
const char* tsclientID = "GyEONRoVGTQ9FBQLPDAELRk";
const char* tsUser = "GyEONRoVGTQ9FBQLPDAELRk";

String clientId;
WiFiClient espClient;
PubSubClient localClient(espClient);
PubSubClient thingspeakClient(espClient);

TwoWire dbmeter = TwoWire(0);
int averageDecibel = 0;

// Read decibel value
uint8_t dbmeter_readreg(uint8_t reg) {
    dbmeter.beginTransmission(DBM_ADDR);
    dbmeter.write(reg);
    dbmeter.endTransmission(false);
    dbmeter.requestFrom(DBM_ADDR, (uint8_t)1);
    return dbmeter.available() ? dbmeter.read() : 255;
}

void ensureWiFiConnected() {
    if (WiFi.status() != WL_CONNECTED) {
        Serial.println("⚠️ Wi-Fi Disconnected! Attempting to reconnect...");
        WiFi.begin(ssid, password);
        unsigned long startAttemptTime = millis();
        
        while (WiFi.status() != WL_CONNECTED && millis() - startAttemptTime < 10000) {
            delay(1000);
            Serial.print(".");
        }
        
        if (WiFi.status() == WL_CONNECTED) {
            Serial.println("\n✅ Reconnected to Wi-Fi!");
        } else {
            Serial.println("\n❌ Wi-Fi Reconnection Failed!");
        }
    }
}

void setupMQTT() {
    localClient.setServer(localMqttServer, 1883);
    thingspeakClient.setServer(thingspeakMqttServer, 1883);

    if (localClient.connect(clientId.c_str())) {
        localClient.subscribe("prod/decibel/AvgDecibel");
        Serial.println("✅ Connected to Local MQTT Broker");
        }

    if (thingspeakClient.connect(tsclientID, tsUser, tsPassword)) {
        Serial.println("✅ Connected to ThingSpeak MQTT");
        }
    }

void ensureMQTTConnected() {
    ensureWiFiConnected();  // ✅ First, ensure Wi-Fi is stable

    // 🔹 Check and reconnect LOCAL MQTT Broker (Raspberry Pi)
    if (!localClient.connected()) {
        Serial.println("MQTT Disconnected! Attempting to reconnect...");
        if (localClient.connect(clientId.c_str())) {
            Serial.println("✅ Reconnected to Local MQTT Broker.");
            localClient.subscribe("prod/decibel/AvgDecibel");  // ✅ Subscribe again
        } else {
            Serial.println("❌ Local MQTT Reconnection Failed!");
        }
    }

    // 🔹 Check and reconnect THINGSPEAK MQTT Broker
    Serial.printf("Connecting to MQTT Broker: %s\n", thingspeakMqttServer);
    Serial.printf("Client ID: %s\n | API Key: %s\n", "GyEONRoVGTQ9FBQLPDAELRk", "odTDEYCrxbWLhnSvluTkbMdK");

//     if (!thingspeakClient.connect(thingspeakClient.connect(tsclientID, tsUser, tsPassword))) {
//         Serial.printf("❌ Connection Failed, Error Code: %d\n", thingspeakClient.state());
// }
    if (!thingspeakClient.connected()) {
        Serial.println("ThingSpeak MQTT Disconnected! Attempting to reconnect...");
        if (thingspeakClient.connect(tsclientID, tsUser, tsPassword)) {
            Serial.println("✅ Reconnected to ThingSpeak MQTT.");
        } else {
            Serial.println("❌ ThingSpeak MQTT Reconnection Failed!");
        }
    }
}

void setup() {
    Serial.begin(115200);
    dbmeter.begin(I2C_SDA, I2C_SCL);

    pinMode(3, OUTPUT); // pinMode(3, OUTPUT);
    digitalWrite(3, LOW); // digitalWrite(3, LOW); // Activate RF switch control

    delay(100);

    pinMode(14, OUTPUT); // pinMode(14, OUTPUT);
    digitalWrite(14, HIGH); // digitalWrite(14, HIGH); // Use external antenna
    
    WiFi.begin(ssid, password);
        while (WiFi.status() != WL_CONNECTED) {
            delay(1000);
            Serial.print(".");
        }
        Serial.println("\nConnected to Wi-Fi!");

    uint8_t mac[6];
    WiFi.macAddress(mac);  // Get MAC address array

    char macStr[18];  // Buffer for formatted MAC
    snprintf(macStr, sizeof(macStr), "%02X:%02X:%02X:%02X:%02X:%02X", 
            mac[0], mac[1], mac[2], mac[3], mac[4], mac[5]);

    clientId = "ESP32_Client_" + String(macStr);

    Serial.print("Client ID: ");
    Serial.println(clientId);

    setupMQTT();
        
    // client.setServer(mqttServer, mqttPort);
    // if (client.connect(clientId.c_str())) {
    //     Serial.println("Connected to MQTT Broker!");
    // } else {
    //     Serial.println("MQTT Connection Failed!");
    // }
    
    // 🔹 Configure NTP to get UTC time
    delay(5000);
    configTime(0, 0, "time.google.com");  
    Serial.println("NTP Time Synced!");

    time_t now = time(nullptr);
    struct tm timeinfo;
    localtime_r(&now, &timeinfo);  // Convert UTC to local time

    int utcOffset = timeinfo.tm_isdst ? -7 : -8;  // Handle DST
    timeinfo.tm_hour += utcOffset;  // Apply Pacific Time offset

    char pacificTimestamp[25];
    strftime(pacificTimestamp, sizeof(pacificTimestamp), "%Y-%m-%dT%H:%M:%SZ", &timeinfo);

    Serial.printf("Pacific Time Timestamp: %s\n", pacificTimestamp);

}

void loop() {
    ensureMQTTConnected();  // ✅ Keep MQTT connections alive
    localClient.loop();  // ✅ Maintain local MQTT connection
    thingspeakClient.loop();  // ✅ Maintain ThingSpeak MQTT connection

//     ensureWiFiConnected();
//     if (!localClient.connected()) {
//     Serial.println("MQTT Disconnected! Attempting to reconnect...");
//     if (client.connect(clientId.c_str())) {
//         Serial.println("Reconnected to MQTT Broker.");
//         localClient.subscribe("prod/decibel/AvgDecibel");
//     } else {
//         Serial.println("MQTT Reconnection Failed!");
//     }
// }
    // time_t now = time(nullptr);
    // struct tm timeinfo;
    // localtime_r(&now, &timeinfo);  // Convert UTC to local time

    // int utcOffset = timeinfo.tm_isdst ? -7 : -8;  // Handle DST for Pacific Time
    // timeinfo.tm_hour += utcOffset;  // Apply offset

    // char pacificTimestamp[25];  // Declare `pacificTimestamp` inside loop()
    // strftime(pacificTimestamp, sizeof(pacificTimestamp), "%Y-%m-%dT%H:%M:%S", &timeinfo);

    uint8_t decibelValue = dbmeter_readreg(DBM_REG_DECIBEL);
    if (decibelValue == 255) {
        Serial.println("Error: Failed to read decibel value.");
        return;
    }

    // Compute moving average (simplified)
    averageDecibel = (averageDecibel * 0.95) + (decibelValue * 0.05);

    // char mqttPayload[128];
    // snprintf(mqttPayload, sizeof(mqttPayload),
    //     "{\"timestamp\":\"%s\",\"actual_decibel\":%d,\"average_decibel\":%d}",
    //     pacificTimestamp, decibelValue, averageDecibel);
    localClient.publish("prod/decibel/AvgDecibel", String(averageDecibel).c_str());
    String payload = "field1=" + String(averageDecibel) + "&status=MQTTPUBLISH";
    thingspeakClient.publish("channels/2445567/publish", payload.c_str());  // ThingSpeak
    localClient.publish("prod/decibel/ActDecibel", String(decibelValue).c_str());

    //Serial.printf("Sent Avg Decibel: %d\n", averageDecibel);
    Serial.printf("[%lu ms] Actual Decibel: %d | Avg Decibel: %d\n", millis(), decibelValue, averageDecibel);
    delay(1000);  // Adjust for sampling rate
}
