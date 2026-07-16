# Práctica 12 - Comunicación

## Objetivo
Dominar UART, I2C, SPI, WiFi y Bluetooth en ESP32 con la Base.

## Nivel
**Advanced** | Duración: ~5 horas

## Hardware disponible en Base
| Bus | Pines | Dispositivos en Base | Expansión |
|-----|-------|---------------------|-----------|
| **UART0** | GPIO1(TX), GPIO3(RX) | USB-Serial (debug) | - |
| **UART1** | GPIO9,10 (flash) | No usar | - |
| **UART2** | GPIO16,17 | Libre (J_GPIO) | GPS, LoRa, etc. |
| **I2C** | GPIO21(SDA), GPIO19(SCL) | Libre (J_I2C) | BMP280, OLED, RTC |
| **SPI** | GPIO23(MOSI),19(MISO),18(SCK),5(CS) | Libre (J_SPI) | SD, TFT, RF |
| **WiFi** | Interno | 2.4GHz b/g/n | AP/Station |
| **BLE** | Interno | 4.2/5.0 | Beacons, GATT |

---

## 1. UART - Comunicación Serie

### UART0 (USB - Debug)
```cpp
// Ya funciona con Serial.begin(115200)
void setup() {
  Serial.begin(115200);
}
void loop() {
  if(Serial.available()) {
    char c = Serial.read();
    Serial.write(c);  // Echo
  }
}
```

### UART2 (Pines libres: GPIO16,17)
```cpp
#define RX2 16
#define TX2 17

HardwareSerial Serial2(RX2, TX2);  // O usar Serial2 directamente

void setup() {
  Serial.begin(115200);
  Serial2.begin(9600, SERIAL_8N1, RX2, TX2);
}

void loop() {
  if(Serial2.available()) {
    String msg = Serial2.readStringUntil('\n');
    Serial.print("Recibido: "); Serial.println(msg);
  }
  // Enviar
  Serial2.println("Hola desde ESP32");
  delay(1000);
}
```

---

## 2. I2C - Sensores y Displays

### Escanear bus
```cpp
#include <Wire.h>
#define SDA_PIN 21
#define SCL_PIN 19

void scanI2C() {
  Wire.begin(SDA_PIN, SCL_PIN);
  Serial.println("Escaneando I2C...");
  for(byte addr=1; addr<127; addr++) {
    Wire.beginTransmission(addr);
    if(Wire.endTransmission() == 0) {
      Serial.printf("Dispositivo en 0x%02X\n", addr);
    }
  }
}
```

### Leer BMP280 (ejemplo sensor externo)
```cpp
#include <Adafruit_BMP280.h>
Adafruit_BMP280 bmp;

void setup() {
  Wire.begin(21, 19);
  if(!bmp.begin(0x76)) Serial.println("BMP280 no encontrado");
}

void loop() {
  Serial.printf("T: %.1f°C  P: %.0fPa\n", 
                bmp.readTemperature(), bmp.readPressure());
  delay(1000);
}
```

---

## 3. SPI - Tarjeta SD, Pantallas

### Inicialización
```cpp
#include <SPI.h>
#define SCK  18
#define MISO 19
#define MOSI 23
#define CS   5

SPIClass spi = SPIClass(HSPI);  // HSPI o VSPI

void setup() {
  spi.begin(SCK, MISO, MOSI, CS);
  // Configurar dispositivo...
}
```

### Tarjeta SD (ejemplo)
```cpp
#include <SD.h>
#define SD_CS 5

void setup() {
  SPI.begin(18, 19, 23, 5);
  if(!SD.begin(5)) Serial.println("SD falló");
  else Serial.println("SD OK");
  
  File f = SD.open("/test.txt", FILE_WRITE);
  if(f) { f.println("Hola SD"); f.close(); }
}
```

---

## 4. WiFi - Station + AP

### Station (Cliente)
```cpp
#include <WiFi.h>

const char* ssid = "TU_SSID";
const char* pass = "TU_PASS";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, pass);
  while(WiFi.status() != WL_CONNECTED) {
    delay(500); Serial.print(".");
  }
  Serial.printf("\nIP: %s\n", WiFi.localIP().toString().c_str());
}
```

### Access Point (Servidor Web)
```cpp
#include <WiFi.h>
#include <WebServer.h>

const char* ap_ssid = "BaseESP32";
const char* ap_pass = "12345678";
WebServer server(80);

void handleRoot() {
  server.send(200, "text/html", 
    "<h1>Base ESP32</h1>"
    "<p>Temp: " + String(readTemp()) + "°C</p>"
    "<p><a href='/led/on'>LED ON</a> | <a href='/led/off'>LED OFF</a></p>"
  );
}

void handleLED() {
  digitalWrite(4, server.uri() == "/led/on");
  server.sendHeader("Location", "/");
  server.send(303);
}

void setup() {
  Serial.begin(115200);
  pinMode(4, OUTPUT);
  
  WiFi.softAP(ap_ssid, ap_pass);
  Serial.printf("AP IP: %s\n", WiFi.softAPIP().toString().c_str());
  
  server.on("/", handleRoot);
  server.on("/led/on", handleLED);
  server.on("/led/off", handleLED);
  server.begin();
}

void loop() { server.handleClient(); }
```

### HTTP Client (GET/POST)
```cpp
#include <HTTPClient.h>

void sendData(float temp) {
  HTTPClient http;
  http.begin("http://api.thingspeak.com/update");
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");
  
  String payload = "api_key=TU_KEY&field1=" + String(temp);
  int code = http.POST(payload);
  Serial.printf("HTTP: %d\n", code);
  http.end();
}
```

---

## 5. Bluetooth Low Energy (BLE)

### BLE Server (Periférico)
```cpp
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUID "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHAR_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"

BLEServer* pServer;
BLECharacteristic* pCharacteristic;
bool deviceConnected = false;

class MyCallbacks: public BLECharacteristicCallbacks {
  void onWrite(BLECharacteristic* pChar) {
    String value = pChar->getValue().c_str();
    if(value == "ON") digitalWrite(4, HIGH);
    if(value == "OFF") digitalWrite(4, LOW);
  }
};

void setup() {
  Serial.begin(115200);
  pinMode(4, OUTPUT);
  
  BLEDevice::init("BaseESP32");
  pServer = BLEDevice::createServer();
  
  BLEService* pService = pServer->createService(SERVICE_UUID);
  pCharacteristic = pService->createCharacteristic(
    CHAR_UUID,
    BLECharacteristic::PROPERTY_READ | BLECharacteristic::PROPERTY_WRITE | BLECharacteristic::PROPERTY_NOTIFY
  );
  pCharacteristic->setCallbacks(new MyCallbacks());
  pCharacteristic->addDescriptor(new BLE2902());
  
  pService->start();
  BLEAdvertising* pAdv = BLEDevice::getAdvertising();
  pAdv->addServiceUUID(SERVICE_UUID);
  pAdv->start();
  Serial.println("BLE Server listo");
}

void loop() {
  if(deviceConnected) {
    float t = readTemp();
    pCharacteristic->setValue(String(t).c_str());
    pCharacteristic->notify();
    delay(1000);
  }
}
```

### BLE Client (Central) - Escanear
```cpp
#include <BLEDevice.h>

void scanBLE() {
  BLEScan* pScan = BLEDevice::getScan();
  pScan->setActiveScan(true);
  BLEScanResults results = pScan->start(5);
  
  for(int i=0; i<results.getCount(); i++) {
    BLEAdvertisedDevice dev = results.getDevice(i);
    Serial.printf("MAC: %s  RSSI: %d  Name: %s\n",
                  dev.getAddress().toString().c_str(),
                  dev.getRSSI(),
                  dev.getName().c_str());
  }
}
```

---

## 6. OTA (Over-The-Air) Updates

### ArduinoOTA
```cpp
#include <ArduinoOTA.h>

void setup() {
  // ... WiFi conectado ...
  
  ArduinoOTA.setHostname("base-esp32");
  ArduinoOTA.setPassword("admin123");
  
  ArduinoOTA.onStart([]() { Serial.println("OTA Start"); });
  ArduinoOTA.onEnd([]() { Serial.println("\nOTA End"); });
  ArduinoOTA.onProgress([](unsigned int p, unsigned int t) {
    Serial.printf("Progreso: %u%%\r", (p / (t / 100)));
  });
  ArduinoOTA.onError([](ota_error_t e) {
    Serial.printf("Error[%u]: ", e);
  });
  
  ArduinoOTA.begin();
}

void loop() {
  ArduinoOTA.handle();
  // Tu código...
}
```

---

## Prácticas integradas

| Combo | Descripción |
|-------|-------------|
| **Sensor + WiFi** | Enviar temp a Thingspeak cada 30s |
| **Sensor + BLE** | App móvil lee temperatura en tiempo real |
| **UART + I2C** | GPS (UART) + BMP280 (I2C) → Log SD (SPI) |
| **WebServer + Motor** | Controlar robot desde navegador |
| **BLE + OTA** | Actualizar firmware desde app móvil |

## Verificación
1. **UART**: Conectar GPS/LoRa a GPIO16/17 → Leer NMEA
2. **I2C**: Conectar OLED SSD1306 (0x3C) → Mostrar temp
3. **SPI**: Tarjeta SD → Escribir/leer archivo
4. **WiFi AP**: Conectar móvil a "BaseESP32" → Abrir 192.168.4.1
5. **BLE**: App nRF Connect → Conectar "BaseESP32" → Escribir ON/OFF
6. **OTA**: Arduino IDE → Tools → Port → "base-esp32 at IP" → Upload

## Retos

| Nivel | Reto |
|-------|------|
| 🟡 Medio | MQTT: Medio | Thingspeak/InfluxDB logging automático |
| 🟡 Medio | Servidor web con gráficas Chart.js en tiempo real |
| 🔴 Difícil | Mesh WiFi: Varios Base ESP32 sincronizados |
| 🔴 Difícil | BLE Mesh: Controlar múltiples servos desde app |
| 🔴 Difícil | LoRaWAN: Enviar datos a The Things Network |

## Referencias
- [ESP32 Arduino WiFi](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFi)
- [ESP32 Arduino BLE](https://github.com/espressif/arduino-esp32/tree/master/libraries/BLE)
- [ArduinoOTA](https://github.com/espressif/arduino-esp32/tree/master/libraries/ArduinoOTA)
- [ESP32 Technical Reference - Communication](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)