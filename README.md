
#include "mbed.h"

// Definición de pines para el TM1637 y botones
#define CLK_PIN D3  // Comunicación TM1637
#define DIO_PIN D4  // Comunicación TM1637
#define B_UNI D5
#define B_DEC D6
#define B_CEN D7
#define B_MIL D8
#define B_OK D9  // Botón de confirmación
#define LED_PIN LED1       // LED en la tarjeta Nucleo

// Inicialización de los pines
DigitalOut clk(CLK_PIN);
DigitalInOut dio(DIO_PIN);
DigitalIn unidades(B_UNI);
DigitalIn decenas(B_CEN);
DigitalIn centenas(B_DEC);
DigitalIn miles(B_MIL);
DigitalIn confirm(B_OK);
DigitalOut led(LED_PIN);

// Tiempo de espera para las señales del protocolo TM1637
#define DELAY_US 5

// Variables para guardar el número generado y el número ingresado
int randomNumber;
int userNumber[4] = {0, 0, 0, 0};

// Funciones básicas para manejar el TM1637
void startCondition() {
    dio.output();
    dio = 1;
    clk = 1;
    wait_us(DELAY_US);
    dio = 0;
    wait_us(DELAY_US);
    clk = 0;
}

void stopCondition() {
    dio.output();
    clk = 0;
    dio = 0;
    wait_us(DELAY_US);
    clk = 1;
    dio = 1;
}

void sendByte(char data) {
    for (int i = 0; i < 8; i++) {
        clk = 0;
        dio = (data >> i) & 0x01;  // Envía bit por bit
        wait_us(DELAY_US);
        clk = 1;
        wait_us(DELAY_US);
    }
    clk = 0;
    dio.input();  // Lee el bit de reconocimiento
    wait_us(DELAY_US);
    clk = 1;
    wait_us(DELAY_US);
    dio.output();  // Regresa a modo de salida
}

void writeCommand(char cmd) {
    startCondition();
    sendByte(cmd);
    stopCondition();
}

void displayDigit(int digit, int position) {
    const char digitToSegment[] = {
        0x3F, // 0
        0x06, // 1
        0x5B, // 2
        0x4F, // 3
        0x66, // 4
        0x6D, // 5
        0x7D, // 6
        0x07, // 7
        0x7F, // 8
        0x6F  // 9
    };

    writeCommand(0x44);  // Modo de dirección fija
    startCondition();   // Inicia la secuencia de transmisión de datos hacia el TM1637
    sendByte(0xC0 | position);  // Dirección de la posición
    sendByte(digitToSegment[digit]);  // Dígito a mostrar
    stopCondition();
}

void clearDisplay() {
    for (int i = 0; i < 4; i++) {
        displayDigit(0, i);  // Limpia las posiciones del display
    }
}

int generaRandomNumber() {
    return rand() % 10000;  // Genera un número aleatorio entre 0000 y 9999
}

void displayNumber(int number) {
    // Descomponer el número en miles, centenas, decenas y unidades
    int miles = number / 1000;
    int centenas = (number / 100) % 10;
    int decenas = (number / 10) % 10;
    int unidades = number % 10;

    // Mostrar cada dígito en el display
    displayDigit(miles, 3);
    displayDigit(centenas, 2);
    displayDigit(decenas, 1);
    displayDigit(unidades, 0);
}

void resetGame() {
    clearDisplay();
    userNumber[0] = 0;
    userNumber[1] = 0;
    userNumber[2] = 0;
    userNumber[3] = 0;
    displayNumber(0);
}

bool checkUserInput(int randomNumber) {
    int userEnteredNumber = userNumber[3] * 1000 + userNumber[2] * 100 + userNumber[1] * 10 + userNumber[0];
    return userEnteredNumber == randomNumber;
}

void updateUserInput() {
    // Verificar si los botones fueron presionados y actualizar los números
    if (unidades.read() == 0) {
        userNumber[0] = (userNumber[0] + 1) % 10;  // Aumenta las unidades
        displayDigit(userNumber[0], 0);
        ThisThread::sleep_for(300ms);  // Evitar rebotes
    }
    if (decenas.read() == 0) {
        userNumber[1] = (userNumber[1] + 1) % 10;  // Aumenta las decenas
        displayDigit(userNumber[1], 1);
        ThisThread::sleep_for(300ms);  // Evitar rebotes
    }
    if (centenas.read() == 0) {
        userNumber[2] = (userNumber[2] + 1) % 10;  // Aumenta las centenas
        displayDigit(userNumber[2], 2);
        ThisThread::sleep_for(300ms);  // Evitar rebotes
    }
    if (miles.read() == 0) {
        userNumber[3] = (userNumber[3] + 1) % 10;  // Aumenta las unidades de mil
        displayDigit(userNumber[3], 3);
        ThisThread::sleep_for(300ms);  // Evitar rebotes
    }
}

void showFeedback(bool success) {
    if (success) {
        led = 1;  // LED encendido si el usuario acierta
        displayNumber(1111);
    } else {
        for (int i = 0; i < 10; i++) {
            led = !led;  // Parpadea el LED
            ThisThread::sleep_for(500ms);
        }
        led = 0;  // Apaga el LED después del parpadeo
        displayNumber(0);
    }
    ThisThread::sleep_for(5s);  // Espera 5 segundos antes de reiniciar el juego
    clearDisplay();  // Limpia el display
}

int main() {
    // Configuración inicial
    writeCommand(0x88);  // Configurar brillo (medio)
    
    // Generar número aleatorio
    randomNumber = generaRandomNumber(); //generar un número aleatorio entre 0000 y 9999.
    displayNumber(randomNumber);
    ThisThread::sleep_for(1s);  // Mostrar el número durante 1 segundo
    clearDisplay();  // Limpiar el display

    while (true) {
        if (confirm.read() == 0) {  // Verificar si el botón de confirmación fue presionado
            if (checkUserInput(randomNumber)) {
                // Si el usuario acierta
                showFeedback(true);
                resetGame();  // Reiniciar el juego
                randomNumber = generaRandomNumber();  // Nuevo número
                displayNumber(randomNumber);
                ThisThread::sleep_for(1s);
                clearDisplay();
            } else {
                // Si el usuario falla
                showFeedback(false);
                resetGame();  // Reiniciar el juego
                randomNumber = generaRandomNumber();  // Nuevo número
                displayNumber(randomNumber);
                ThisThread::sleep_for(1s);
                clearDisplay();
            }
        }
        updateUserInput();  // Actualizar los valores ingresados por el usuario
    }
}
