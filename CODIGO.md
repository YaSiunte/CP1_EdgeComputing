#include <LiquidCrystal.h>

LiquidCrystal lcd(12, 11, 10, 5, 4, 3, 2);

byte c1_11[8] = { 
B00010, 
B00010,
B00010, 
B00010, 
B00001, 
B00000, 
B00000, 
B00000,
};

byte f0_12[8] = { 
B00000, 
B00000, 
B00000, 
B00001, 
B00011, 
B00010, 
B00010, 
B00010,
};

byte f1_12[8] = { 
B00110, 
B00100, 
B00100, 
B00100, 
B10101, 
B10101, 
B01110, 
B00100,
};

byte c0_13[8] = { 
B00000,
B00000, 
B00000, 
B11111, 
B11111,
B00000, 
B00000, 
B00000,
};

byte c1_14[8] = { 
B00000, 
B01111, 
B11111, 
B11111, 
B11111, 
B11111, 
B01111, 
B00000,
};

byte c0_15[8] = { 
B00000, 
B11111, 
B11111, 
B11111, 
B11111, 
B11111, 
B11111,
B00000,
};

byte c1_13[8] = {
B01000,
B01000,
B01000,
B01000,
B10000,
B00000,
B00000,
B00000,
};


// ---------- Pinos LEDs / Buzzer / LDR ----------
const int LED_VERDE    = 6;
const int LED_AMARELO  = 7;
const int LED_VERMELHO = 8;
const int BUZZER       = 9;
const int PINO_LDR     = A0;

// ---------- Limites em % ----------
const int LIMITE_OK_MIN       = 30;
const int LIMITE_OK_MAX       = 70;
const int LIMITE_PROBLEMA_MIN = 15;
const int LIMITE_PROBLEMA_MAX = 85;

// ---------- Média móvel ----------
const int N_AMOSTRAS = 10;
int amostras[N_AMOSTRAS];
int idxAmostra = 0;

// ---------- Buzzer ----------
bool buzzerTocando = false;
unsigned long inicioBuzzer = 0;
const unsigned long DURACAO_BUZZER = 3000;

// ===== Caracteres customizados do logo InVino =====

void setup() {
  lcd.begin(16,2);
  lcd.clear();
  
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AMARELO, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);
  pinMode(BUZZER, OUTPUT);

  Serial.begin(9600);

  // 1) Inicializa LCD PRIMEIRO
  lcd.begin(16, 2);
  lcd.clear();

  // 2) DEPOIS cria os caracteres customizados
  lcd.createChar(0, c1_11);
  lcd.createChar(1, f0_12);
  lcd.createChar(2, f1_12);
  lcd.createChar(3, c0_13);
  lcd.createChar(4, c1_14);
  lcd.createChar(5, c1_13);
  lcd.createChar(6, c0_15);

  // 3) Escreve a tela de boas-vindas
  lcd.setCursor(3, 0);
  lcd.print("InVino");
  lcd.setCursor(0, 1);
  lcd.print("Boas-vindas");

  // Logo - linha superior
  lcd.setCursor(12, 0);
  lcd.write(byte(1));
  lcd.setCursor(13, 0); 
  lcd.write(byte(3));
  lcd.setCursor(14, 0); 
  lcd.write(byte(4));
  lcd.setCursor(15, 0); 
  lcd.write(byte(6));

  // Logo - linha inferior
  lcd.setCursor(11, 1);
  lcd.write(byte(0));
  lcd.setCursor(12, 1);
  lcd.write(byte(2));
  lcd.setCursor(13, 1);
  lcd.write(byte(5));

  // 4) Mantém na tela por 5 segundos
  delay(5000);
  lcd.clear();

  // Inicializa buffer da média
  int leitura = analogRead(PINO_LDR);
  for (int i = 0; i < N_AMOSTRAS; i++) amostras[i] = leitura;
}


void loop() {
  // 1) Lê ADC
  int leituraADC = analogRead(PINO_LDR);

  // 2) Média móvel
  amostras[idxAmostra] = leituraADC;
  idxAmostra = (idxAmostra + 1) % N_AMOSTRAS;
  long soma = 0;
  for (int i = 0; i < N_AMOSTRAS; i++) soma += amostras[i];
  int mediaADC = soma / N_AMOSTRAS;

  // 3) Converte para 0..100%
  int luminosidade = map(mediaADC, 0, 1023, 0, 100);
  luminosidade = constrain(luminosidade, 0, 100);

  // 4) Estado
  String estado;
  if (luminosidade < LIMITE_PROBLEMA_MIN || luminosidade > LIMITE_PROBLEMA_MAX) {
    estado = "PROBLEMA";
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_AMARELO, LOW);
    digitalWrite(LED_VERMELHO, HIGH);
    pararBuzzer();
  }
  else if (luminosidade < LIMITE_OK_MIN || luminosidade > LIMITE_OK_MAX) {
    estado = "ALERTA  ";
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_AMARELO, HIGH);
    digitalWrite(LED_VERMELHO, LOW);
    gerenciarBuzzerAlerta();
  }
  else {
    estado = "OK      ";
    digitalWrite(LED_VERDE, HIGH);
    digitalWrite(LED_AMARELO, LOW);
    digitalWrite(LED_VERMELHO, LOW);
    pararBuzzer();
  }

  // 5) LCD - sempre com espaços no final p/ apagar lixo
  lcd.setCursor(0, 0);
  lcd.print("Luz: ");
  lcd.print(luminosidade);
  lcd.print("%   ");
  lcd.setCursor(0, 1);
  lcd.print("Status: ");
  lcd.print(estado);

  // 6) Debug
  Serial.print("ADC="); Serial.print(mediaADC);
  Serial.print(" | Lum="); Serial.print(luminosidade);
  Serial.print("% | "); Serial.println(estado);

  delay(200);
}

void gerenciarBuzzerAlerta() {
  unsigned long agora = millis();
  if (!buzzerTocando) {
    tone(BUZZER, 1000);
    buzzerTocando = true;
    inicioBuzzer = agora;
  } else if (agora - inicioBuzzer >= DURACAO_BUZZER) {
    noTone(BUZZER);
    buzzerTocando = false;
    inicioBuzzer = agora;
  }
}

void pararBuzzer() {
  if (buzzerTocando) {
    noTone(BUZZER);
    buzzerTocando = false;
  }
}
