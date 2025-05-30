# 🌊 Sistema de Monitoramento de Enchentes com Arduino

Projeto de Arduino preventivo para alagamentos em áreas urbanas.

---

## 📌 Descrição do Problema

As enchentes urbanas são um problema recorrente em cidades com infraestrutura de drenagem insuficiente, causando:

- ✔️ Danos materiais (residências, veículos, comércios)  
- ✔️ Riscos à vida (afogamentos, deslizamentos)  
- ✔️ Prejuízos econômicos (bloqueio de vias, interrupção de serviços)

**Cenário típico:**  
Chuvas intensas elevam rapidamente o nível da água em bueiros, rios e ruas, sem nenhum sistema de alerta antecipado. Este projeto propõe uma solução de monitoramento em tempo real para minimizar riscos.

---

## 🛠️ Visão Geral da Solução

### 🎯 Objetivo

Desenvolver um sistema embarcado com Arduino que **monitore o nível de água** e **intensidade de chuva**, emitindo **alertas visuais, sonoros e informativos** em tempo real.

### 🔧 Funcionalidades

| Função                     | Componente                          |
|---------------------------|--------------------------------------|
| Medição de nível da água  | Sensor Ultrassônico HC-SR04         |
| Simulação de chuva        | Potenciômetro                       |
| Exibição de status        | Display LCD 16x2 I2C                |
| Alerta visual             | LEDs RGB (Verde, Amarelo, Vermelho) |
| Alerta sonoro             | Buzzer                              |

### ⚠️ Níveis de Risco

| Estado     | Condição                                     | Sinalização                                   |
|------------|----------------------------------------------|-----------------------------------------------|
| ✅ Normal   | Água < 10cm e chuva < 50%                    | LED Verde, sem alarme                         |
| ⚠️ Alerta   | Água > 10cm ou chuva > 50%                  | LED Amarelo, buzzer intermitente              |
| 🚨 Crítico  | Água > 20cm ou chuva > 80%                  | LED Vermelho, buzzer contínuo + alerta no LCD |

---

## 🔌 Diagrama do Circuito

📷 <img src="![Sistema de monitoriamento das enchentes](https://github.com/user-attachments/assets/b41648f0-4c75-4ef1-ac76-01afee6c7b7b)
" alt="Circuito Arduino">

---

## 🧪 Guia para Simulação

### 1️⃣ Acesso

- 🔗 [Abrir no Wokwi](https://wokwi.com/projects/432330200928587777)

### 2️⃣ Instruções para Teste

1. Gire o **potenciômetro** para simular intensidade da chuva.
2. Mova o objeto virtual no **sensor ultrassônico** para simular nível da água.
3. **Observe os comportamentos:**
   - LCD exibe dados em tempo real
   - LEDs mudam de cor
   - Buzzer emite som conforme o risco

---

## 🎥 Vídeo Demonstrativo

📽️ Assista ao sistema em ação:  
🔗 [Clique aqui para ver o vídeo](#)

---

## 💻 Código Fonte

### 🧾 Código Completo (Arduino IDE)

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Configurações de alerta
const int NIVEL_CRITICO = 20;    // cm
const int NIVEL_ALERTA = 10;     // cm
const int CHUVA_CRITICA = 80;    // %
const int CHUVA_ALERTA = 50;     // %
const int ALTURA_REFERENCIA = 60; // cm do sensor até o solo

// Pinos do Arduino
const int trigPin = 9;
const int echoPin = 10;
const int sensorChuva = A0;
const int ledVermelho = 7;
const int ledAmarelo = 6;
const int ledVerde = 5;
const int buzzer = 4;

// Inicialização do LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledVermelho, OUTPUT);
  pinMode(ledAmarelo, OUTPUT);
  pinMode(ledVerde, OUTPUT);
  pinMode(buzzer, OUTPUT);

  lcd.init();
  lcd.backlight();
  lcd.print("Iniciando sistema");
  lcd.setCursor(0, 1);
  lcd.print("de monitoramento...");
  delay(2000);

  Serial.begin(9600);
}

void loop() {
  int nivelAgua = medirNivelAgua();
  int intensidadeChuva = analogRead(sensorChuva);
  intensidadeChuva = map(intensidadeChuva, 0, 1023, 0, 100);

  lcd.clear();
  lcd.print("Agua: ");
  lcd.print(nivelAgua);
  lcd.print("cm");
  lcd.setCursor(0, 1);
  lcd.print("Chuva: ");
  lcd.print(intensidadeChuva);
  lcd.print("%");

  if (nivelAgua > NIVEL_CRITICO || intensidadeChuva > CHUVA_CRITICA) {
    acionarAlerta(HIGH, LOW, LOW, true, "PERIGO! EVACUAR");
    tone(buzzer, 1000); 
  } else if (nivelAgua > NIVEL_ALERTA || intensidadeChuva > CHUVA_ALERTA) {
    acionarAlerta(LOW, HIGH, LOW, true, "ATENCAO! ALERTA");
    tone(buzzer, 600, 500); 
    delay(500);
    noTone(buzzer);
  } else {
    acionarAlerta(LOW, LOW, HIGH, false, "SITUACAO NORMAL");
    noTone(buzzer);
  }

  delay(1000);
}

int medirNivelAgua() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duracao = pulseIn(echoPin, HIGH);
  int distancia = duracao * 0.034 / 2;

  if(distancia > 100 || distancia < 2) {
    return 0;
  }

  return max(0, ALTURA_REFERENCIA - distancia);
}

void acionarAlerta(int vermelho, int amarelo, int verde, bool alarme, String mensagem) {
  digitalWrite(ledVermelho, vermelho);
  digitalWrite(ledAmarelo, amarelo);
  digitalWrite(ledVerde, verde);
  lcd.clear();
  lcd.print(mensagem);
}
