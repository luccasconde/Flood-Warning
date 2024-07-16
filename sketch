#include <Arduino.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <LiquidCrystal.h>

const char *ssid = "Wokwi-GUEST";
const char *password = "";
const String API_key = "da7a6ee00b5b0f924f227254c25c450e";
const String city = "Sorocaba";
String url = "http://api.openweathermap.org/data/2.5/weather?q=" + city + "&appid=" + API_key + "&units=metric";

// Definição dos pinos do ESP32
#define LED_PIN 14
#define BUTTON_PIN 4
#define SDA_PIN 21
#define SCL_PIN 22

bool buttonPressed = false; // Variável para controlar se o botão foi pressionado

// Inicializa o display LCD com endereço I2C e tamanho do display (16 colunas por 2 linhas)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Variáveis para armazenar os dados do tempo
String cidade = "";
float temperatura = 0.0;
int umidade = 0;

// Variável para controlar o tempo de atualização
unsigned long previousMillis = 0;
const long updateInterval = 60000; // Intervalo de 60 segundos para atualização da API

void setup() {
  // Configuração dos pinos
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP); // Ativa o pull-up interno
  pinMode(SDA_PIN, OUTPUT);
  pinMode(SCL_PIN, OUTPUT);

  Serial.begin(9600);
  lcd.begin(16, 2);
  WiFi.begin(ssid, password);
  pinMode(15, OUTPUT);
  Serial.println("Estabelecendo conexão");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
  Serial.println("Rede Conectada!");

  // Inicializa o display LCD
  lcd.init();
  lcd.backlight();

  // Atualiza a primeira vez os dados do tempo
  updateWeatherData();
}

void loop() {
  // Lê o estado atual do botão
  bool buttonState = digitalRead(BUTTON_PIN);

  // Verifica se está na hora de atualizar os dados do tempo
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= updateInterval) {
    previousMillis = currentMillis;
    updateWeatherData();
  }

  // Se o botão foi pressionado, exibe a mensagem de "Risco de enchente"
  if (buttonState == LOW && !buttonPressed) {
    buttonPressed = true;
    digitalWrite(LED_PIN, HIGH);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Risco de");
    lcd.setCursor(0, 1);
    lcd.print("enchente");
  } 
  // Se o botão foi solto, volta a exibir a cidade, temperatura e umidade
  else if (buttonState == HIGH && buttonPressed) {
    buttonPressed = false;
    digitalWrite(LED_PIN, LOW);
    displayWeatherData();
  }

  delay(100); // Pequeno atraso para evitar debounce do botão
}

void updateWeatherData() {
  if (WiFi.status() == WL_CONNECTED) {
    digitalWrite(15, HIGH);
    HTTPClient http;
    http.begin(url);
    int httpCode = http.GET();
    if (httpCode == HTTP_CODE_OK) {
      String resposta = http.getString();
      Serial.println(resposta);

      DynamicJsonDocument doc(1024);
      deserializeJson(doc, resposta);

      cidade = doc["name"].as<String>();
      temperatura = doc["main"]["temp"].as<float>();
      umidade = doc["main"]["humidity"].as<int>();

      // Exibe os dados atualizados no display
      displayWeatherData();
    }
    http.end();
  } else {
    digitalWrite(15, LOW);
    WiFi.begin(ssid, password);
    Serial.println("Reestabelecendo a conexão...");
    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
    }
  }
}

void displayWeatherData() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(cidade);
  lcd.setCursor(0, 1);
  lcd.print("T:");
  lcd.print(temperatura, 1); // Limita a um decimal para melhor visualização
  lcd.print("\xDF");
  lcd.setCursor(8, 1); // Move o cursor para a coluna 8 na segunda linha
  lcd.print(" U:");
  lcd.print(umidade);
  lcd.print("%");
}
