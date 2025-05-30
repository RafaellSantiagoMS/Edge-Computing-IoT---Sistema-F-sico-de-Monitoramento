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

1. Inicie a Simulação
- Clique no botão **"Start Simulation"** (ícone ▶️ no canto superior).

2.  Monitor Serial (opcional)
- Abra o **"Serial Monitor"** na barra lateral.
- Ele exibirá as leituras dos sensores em tempo real — útil para verificar se tudo está funcionando corretamente.


3. Simulando o Nível da Água (Sensor Ultrassônico)

- No **Wokwi**, clique no sensor **HC-SR04** e **ajuste a distância manualmente** na simulação.
- Menor distância → nível de água mais alto  
- Maior distância → nível de água mais baixo  
- Observe:
  - Mudança dos **LEDs**
  - **Alarme sonoro**
  - Atualização no **display LCD**

4.  Simulando a Chuva (Potenciômetro)

- O potenciômetro simula a **intensidade de chuva**.
- Gire o botão para alterar a entrada analógica (A0):
  - 🔽 Gire para menor valor (próximo de 0V): **chuva intensa**
  - 🔼 Gire para valor alto (próximo de 5V): **sem chuva**

>  O Arduino interpreta esse valor como porcentagem (via `map()`), então valores baixos no potenciômetro indicam chuva forte!


### 🔎 O Que Observar

#### 📺 LCD Display
- Mostra:
  - Nível de água (`Agua: XXcm`)
  - Intensidade da chuva (`Chuva: XX%`)
  - Situação atual (`SITUACAO NORMAL`, `ATENCAO! ALERTA`, `PERIGO! EVACUAR`)

#### 💡 LEDs

| LED         | Situação                                    |
|-------------|---------------------------------------------|
| 🟢 Verde     | Situação normal                             |
| 🟡 Amarelo   | Alerta (nível médio de água ou chuva)      |
| 🔴 Vermelho  | Crítico (nível elevado de água ou chuva)   |

#### 🔊 Buzzer

| Tipo de Som        | Situação                           |
|--------------------|------------------------------------|
| Silencioso         | Situação normal                    |
| Bips intermitentes | Alerta                             |
| Alarme contínuo    | Situação crítica                   |

---

## 🎥 Vídeo Demonstrativo

📽️ Demonstração prática do projeto:  
🔗 [Clique aqui para assistir no YouTube](#)

---

## 💻 Código Fonte

> Projeto desenvolvido e testado com **Arduino Uno no Wokwi**.

```cpp
#include <Wire.h>                      // Biblioteca para comunicação I2C
#include <LiquidCrystal_I2C.h>         // Biblioteca do LCD 16x2 com interface I2C

// =======================
// 🔧 Parâmetros do Sistema
// =======================
const int NIVEL_CRITICO = 20;          // cm - Nível crítico de água
const int NIVEL_ALERTA = 10;           // cm - Nível de alerta
const int CHUVA_CRITICA = 80;          // %  - Chuva crítica (alta intensidade)
const int CHUVA_ALERTA = 50;           // %  - Chuva moderada
const int ALTURA_REFERENCIA = 60;      // cm - Distância entre o sensor e o chão

// =======================
// 🧩 Definição de Pinos
// =======================
const int trigPin = 9;                 // Pino de trigger do sensor ultrassônico
const int echoPin = 10;                // Pino de echo do sensor ultrassônico
const int sensorChuva = A0;            // Entrada analógica do potenciômetro (chuva)
const int ledVermelho = 7;             // LED vermelho - nível crítico
const int ledAmarelo = 6;              // LED amarelo - nível de alerta
const int ledVerde = 5;                // LED verde - nível normal
const int buzzer = 4;                  // Buzzer - alarme sonoro

// =======================
// 📺 Inicialização do LCD
// =======================
LiquidCrystal_I2C lcd(0x27, 16, 2);    // Endereço I2C comum do display LCD

void setup() {
  // Configuração de pinos de entrada/saída
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledVermelho, OUTPUT);
  pinMode(ledAmarelo, OUTPUT);
  pinMode(ledVerde, OUTPUT);
  pinMode(buzzer, OUTPUT);

  // Inicializa o LCD com backlight
  lcd.init();
  lcd.backlight();
  lcd.print("Iniciando sistema");
  lcd.setCursor(0, 1);
  lcd.print("de monitoramento...");
  delay(2000);                         // Tempo de apresentação

  Serial.begin(9600);                  // Ativa a comunicação serial para debug
}

void loop() {
  // 🔄 Leitura dos sensores
  int nivelAgua = medirNivelAgua();    // Lê o nível de água (em cm)
  int intensidadeChuva = analogRead(sensorChuva);  // Lê o valor analógico do potenciômetro
  intensidadeChuva = map(intensidadeChuva, 0, 1023, 0, 100); // Converte para porcentagem

  // 🖥️ Atualiza display com dados
  lcd.clear();
  lcd.print("Agua: ");
  lcd.print(nivelAgua);
  lcd.print("cm");
  lcd.setCursor(0, 1);
  lcd.print("Chuva: ");
  lcd.print(intensidadeChuva);
  lcd.print("%");

  // 🚦 Lógica de alerta baseada nos limites definidos
  if (nivelAgua > NIVEL_CRITICO || intensidadeChuva > CHUVA_CRITICA) {
    // 🚨 Situação Crítica
    acionarAlerta(HIGH, LOW, LOW, true, "PERIGO! EVACUAR");
    tone(buzzer, 1000);                // Som contínuo
  } 
  else if (nivelAgua > NIVEL_ALERTA || intensidadeChuva > CHUVA_ALERTA) {
    // ⚠️ Situação de Alerta
    acionarAlerta(LOW, HIGH, LOW, true, "ATENCAO! ALERTA");
    tone(buzzer, 600, 500);            // Bip intermitente
    delay(500);                        // Pausa entre os bips
    noTone(buzzer);
  } 
  else {
    // ✅ Situação Normal
    acionarAlerta(LOW, LOW, HIGH, false, "SITUACAO NORMAL");
    noTone(buzzer);                    // Sem som
  }

  delay(1000);                         // Atualiza os dados a cada 1s
}

// =====================================================
// 📏 Função para medir o nível da água com HC-SR04
// =====================================================
int medirNivelAgua() {
  // Envia pulso para o sensor ultrassônico
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Mede o tempo de retorno do pulso
  long duracao = pulseIn(echoPin, HIGH);
  int distancia = duracao * 0.034 / 2;   // Converte tempo em cm

  // Filtra valores inválidos ou fora do alcance
  if(distancia > 100 || distancia < 2) {
    return 0; // Valor inválido, retorna 0
  }

  // Retorna o nível da água com base na altura de referência
  return max(0, ALTURA_REFERENCIA - distancia);
}

// =====================================================
// 🚨 Função para acionar LEDs, alarme e mensagem LCD
// =====================================================
void acionarAlerta(int vermelho, int amarelo, int verde, bool alarme, String mensagem) {
  digitalWrite(ledVermelho, vermelho);
  digitalWrite(ledAmarelo, amarelo);
  digitalWrite(ledVerde, verde);

  // Exibe mensagem no LCD
  lcd.clear();
  lcd.print(mensagem);
}

