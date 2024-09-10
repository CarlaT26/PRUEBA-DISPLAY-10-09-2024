
#include "mbed.h"

//Alias

//defines
#define escritura       0x40
#define poner_brillo    0x88
#define dir_display     0xC0

// prototipos
void condicion_start(void);
void condicion_stop (void);
void send_byte (char data);
void send_data (int number);


//hilos

//variables globales 

//pines 
DigitalOut  sclk (D2);
DigitalInOut dio (D3);

const char digitToSegment[10] = { 0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x6D, 0x7D,0x07,0x7F,0x6F};
                               // 0X81,	0XCf,	0X92,	0X86,	0XCC,	0XA4,	0XA0,	0X8F,	0X80,	0X84
int main()
{

    while (true) {
        for (int i = 0; i <= 9999; i++) 
            {
            send_data(i);
            ThisThread::sleep_for(100ms); // Espera 1 segundo antes de mostrar el siguiente número

            }
    }
}

void condicion_start(void)
{
 sclk = 1;
 dio.output();
 dio = 1;
 wait_us(1);
 dio = 0;
 sclk = 0;
}

void condicion_stop (void)
{
    sclk = 0;
    dio.output();
    dio = 0;
    wait_us(1);
    sclk = 1;
    dio = 1;
}

void send_byte(char data)
{
    dio.output();
    for (int i=0; i<8; i++)
    {
          sclk = 0;
          dio = (data&0x01) ? 1:0;
          data >>= 1;   
          sclk = 1;
    }
    // Esperar el ack
    dio.input();
    sclk = 0;
    wait_us(1);
    // Espera la señal de ACK
  

        if (dio==0) 
        {
            sclk=1;
            sclk=0;
        }
}

void send_data(int number)
{
    condicion_start();
    send_byte(escritura);
    condicion_stop();
    condicion_start();
    send_byte(dir_display);
    
    // Descomponer el número en dígitos y enviar al display
    for (int i = 0; i < 4; i++)
    {
        int digit = number % 10;
        send_byte(digitToSegment[digit]);
        number /= 10;
    }
    condicion_stop();
    condicion_start();
    send_byte(poner_brillo + 1);
    condicion_stop();

}#include "mbed.h"

//Alias

//defines
#define escritura       0x40
#define poner_brillo    0x88
#define dir_display     0xC0

// prototipos
void condicion_start(void);
void condicion_stop (void);
void send_byte (char data);
void send_data (int number);


//hilos

//variables globales 

//pines 
DigitalOut  sclk (D2);
DigitalInOut dio (D3);

const char digitToSegment[10] = { 0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x6D, 0x7D,0x07,0x7F,0x6F};
                               // 0X81,	0XCf,	0X92,	0X86,	0XCC,	0XA4,	0XA0,	0X8F,	0X80,	0X84
int main()
{

    while (true) {
        for (int i = 0; i <= 9999; i++) 
            {
            send_data(i);
            ThisThread::sleep_for(100ms); // Espera 1 segundo antes de mostrar el siguiente número

            }
    }
}

void condicion_start(void)
{
 sclk = 1;
 dio.output();
 dio = 1;
 wait_us(1);
 dio = 0;
 sclk = 0;
}

void condicion_stop (void)
{
    sclk = 0;
    dio.output();
    dio = 0;
    wait_us(1);
    sclk = 1;
    dio = 1;
}

void send_byte(char data)
{
    dio.output();
    for (int i=0; i<8; i++)
    {
          sclk = 0;
          dio = (data&0x01) ? 1:0;
          data >>= 1;   
          sclk = 1;
    }
    // Esperar el ack
    dio.input();
    sclk = 0;
    wait_us(1);
    // Espera la señal de ACK
  

        if (dio==0) 
        {
            sclk=1;
            sclk=0;
        }
}

void send_data(int number)
{
    condicion_start();
    send_byte(escritura);
    condicion_stop();
    condicion_start();
    send_byte(dir_display);
    
    // Descomponer el número en dígitos y enviar al display
    for (int i = 0; i < 4; i++)
    {
        int digit = number % 10;
        send_byte(digitToSegment[digit]);
        number /= 10;
    }
    condicion_stop();
    condicion_start();
    send_byte(poner_brillo + 1);
    condicion_stop();

}
