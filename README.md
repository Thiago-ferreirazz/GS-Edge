# GS-Edge


---

# Sistema de Monitoramento Blue Ocean

## Integrantes
- Gabriel Gouvea - RM555528
- Miguel Kapicius - RM556198
- Thiago Ferreira - RM555608

## Descrição do Projeto
O projeto Blue Ocean visa criar uma plataforma integrada para monitoramento da qualidade da água utilizando sensores e uma interface Arduino. O sistema mede parâmetros críticos como temperatura, pH, nível da água e realiza alertas através de LEDs e buzzer. Os dados são exibidos em um display LCD.

## Componentes Utilizados
- Arduino Uno
- Display LCD 16x2 com I2C
- Sensor de temperatura e umidade DHT22
- Sensor ultrassônico HC-SR04
- Potenciômetro
- Servo motor
- LEDs (verde e vermelho)
- Buzzer
- Protoboard e jumpers

## Conexões

### Display LCD
- VCC -> 5V Arduino
- GND -> GND Arduino
- SDA -> A4 Arduino
- SCL -> A5 Arduino

### Sensor DHT22
- VCC -> 5V Arduino
- GND -> GND Arduino
- DATA -> Pino 7 Arduino

### Sensor Ultrassônico HC-SR04
- VCC -> 5V Arduino
- GND -> GND Arduino
- TRIG -> Pino 13 Arduino
- ECHO -> Pino 12 Arduino

### Potenciômetro
- VCC -> 5V Arduino
- GND -> GND Arduino
- Sinal -> A0 Arduino

### Servo Motor
- Sinal -> Pino 3 Arduino
- VCC -> 5V Arduino
- GND -> GND Arduino

### LEDs e Buzzer
- LED Verde (anodo) -> Pino 6 Arduino
- LED Vermelho (anodo) -> Pino 5 Arduino
- Buzzer -> Pino 4 Arduino

## Código Fonte

```cpp
// Gabriel Gouvea #rm 555528
// Miguel Kapicius #rm 556198
// Thiago Ferreira #rm 555608

#include <LiquidCrystal_I2C.h>
#include <DHT.h>
#include <Servo.h>

// Definições do display LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);  // Endereço I2C do display pode variar

// Definição da tela do LCD
int lcdState = 0;
// Definições do sensor DHT22
#define DHTPIN 7       // Pino de dados do DHT22
#define DHTTYPE DHT22  // Modelo do sensor DHT

DHT dht(DHTPIN, DHTTYPE);

// Definição do sensor ultra-sônico
int trigger = 13;
int echo = 12;
float dist = 0;
String levelState = "oi";

// Definição do servo
int servoPin = 3;
Servo servoMotor;

// Definições do potenciômetro
#define POTPIN A0      // Pino do potenciômetro
String phState = "oi";

// Definições leds e buzzer:
const int ledVerde = 6;
const int ledVermelho = 5;
const int buzzer = 4;

void setup() {
  // Inicialização do monitor serial
  Serial.begin(9600);

  // Inicialização do display LCD
  lcd.begin(16, 2);
  lcd.backlight();

  // Inicialização do sensor DHT
  dht.begin();

  // Inicialização do sensor ultra-sônico
  pinMode(trigger, OUTPUT);
  pinMode(echo, INPUT);

  // Inicialização do servo
  pinMode(servoPin, OUTPUT);
  servoMotor.attach(servoPin);

  // Inicialização dos leds e buzzer
  pinMode(ledVerde, OUTPUT);
  pinMode(ledVermelho, OUTPUT);
  pinMode(buzzer, OUTPUT);
}

void loop() {
  // Leitura da temperatura do sensor DHT22
  float temperature = dht.readTemperature();
  
  // Verificação se a leitura foi bem-sucedida
  if (isnan(temperature)) {
    Serial.println("Falha na leitura do sensor DHT22!");
    return;
  }

  // Leitura do valor do potenciômetro
  int potValue = analogRead(POTPIN);
  float phValue = map(potValue, 0, 1023, 0, 120) / 10.0;  // Mapeamento para 0-12.0

  // Definição do estado do PH
  if (phValue < 7) {
    phState = "Baixo";
  } else if (phValue > 8) {
    phState = "Alto";
  } else {
    phState = "Comum";
  }

  // Ativação do sensor ultra-sônico
  digitalWrite(trigger, LOW);
  delayMicroseconds(5);
  digitalWrite(trigger, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigger, LOW);
  
  dist = pulseIn(echo, HIGH) / 58.0;

  // Definição do estado do nível da água e sistema de segurança
  if (dist < 150) {
    levelState = "Nivel: Baixo";
    servoMotor.write(0);
  } else if (dist > 350) {
    levelState = "Nivel: Alto";
    servoMotor.write(90);
  } else {
    levelState = "Nivel: Padrao";
    servoMotor.write(0);
  }

  // Configuração dos leds e buzzer
  if (levelState != "Nivel: Padrao" || phState != "Comum") {
    digitalWrite(ledVerde, LOW);
    digitalWrite(ledVermelho, HIGH);
    tone(buzzer, 500, 900);
  } else {
    digitalWrite(ledVermelho, LOW);
    digitalWrite(ledVerde, HIGH);
    noTone(buzzer);
  }

  // Alterna entre os estados do LCD
  if (lcdState == 0) {
    lcdState = 1;  // Alterna para a próxima tela
  } else {
    lcdState = 0;  // Volta para a tela anterior
  }

  // Exibição dos valores no display LCD
  lcd.clear();
  if (lcdState == 0) {
    lcd.setCursor(0, 0);
    lcd.print("Temp: ");
    lcd.print(temperature);
    lcd.print(" C");

    lcd.setCursor(0, 1);
    lcd.print("PH: ");
    lcd.print(phValue);
    lcd.print(" - ");
    lcd.print(phState);
  } else {
    lcd.setCursor(0, 0);
    lcd.print("Nivel: ");
    lcd.print(dist);
    lcd.print("cm");
    lcd.setCursor(0, 1);
    lcd.print(levelState);
  }

  // Intervalo de atualização
  delay(3000);
}
```

## Como simular
1. Monte o circuito conforme a imagem fornecida.
2. Carregue o código no Arduino.
3. Abra o monitor serial para verificar as leituras.
4. O display LCD alternará entre mostrar temperatura e pH, e o nível da água.
5. O LED verde acenderá se todos os parâmetros estiverem dentro dos limites normais. O LED vermelho e o buzzer serão ativados se algum parâmetro estiver fora dos limites.

---

## Intuito dos Componentes
1. **Potenciômetro** -- Simula valores alternados de PH.
2. **DHT22** -- Detecta a temperatura da superfície da água.
3. **Sensor ultrassônico** -- Simula o aumento ou baixa do nível da água.
4. **Servo-Motor** -- Simula mecanismo de defesa contra nível de água elevado.
5. **Leds, lcd e Buzzer** -- Avisos sonoros e visuais. 
