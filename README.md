# Sistema com Microcontrolador
#include <Keypad.h>
#include <Servo.h>

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {5, 4, 3, 2};
byte colPins[COLS] = {9, 8, 7, 6};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);
Servo servoFechadura;

#define LED_VERDE 11
#define LED_VERMELHO 12
#define BUZZER 13

String codigoAcesso = "9999"; // Código de acesso predefinido

void setup() {
  Serial.begin(9600);
  servoFechadura.attach(10);
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  servoFechadura.write(0); // Fechadura trancada
}

void loop() {
  char key = keypad.getKey();
  if (key) {
    String codigoInserido = "";
    codigoInserido += key;
    Serial.print("Tecla pressionada: ");
    Serial.println(key);

    // Ler o resto do código inserido
    for (int i = 1; i < codigoAcesso.length(); i++) {
      while (key == 0) {
        key = keypad.getKey();
      }
      codigoInserido += key;
      Serial.print("Tecla pressionada: ");
      Serial.println(key);
      key = 0;
    }

    Serial.print("Código inserido: ");
    Serial.println(codigoInserido);

    if (codigoInserido == codigoAcesso) {
      Serial.println("Acesso autorizado!");
      digitalWrite(LED_VERDE, HIGH);
      servoFechadura.write(90); // Abre a fechadura
      tone(BUZZER, 1000); // Toca um som no buzzer
      delay(1000); // Mantém o som por 1 segundo
      noTone(BUZZER); // Desliga o som
      delay(4000); // Mantém a fechadura aberta por 4 segundos
      servoFechadura.write(0); // Tranca a fechadura
      digitalWrite(LED_VERDE, LOW);
    } else {
      Serial.println("Acesso negado!");
      digitalWrite(LED_VERMELHO, HIGH);
      delay(2000);
      digitalWrite(LED_VERMELHO, LOW);
    }
  }
}
