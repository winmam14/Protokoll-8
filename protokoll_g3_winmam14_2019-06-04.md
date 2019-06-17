# Protokoll-8
## Thema: Programm von Gruppe 1 analysiert
**Professor:** SX  
**Übungsdatum:** 04.06.2019  
**Author, KNr.:** Winter Matthias, 17    
**Anwesend:** Alois Vollmaier, Sarah Vezonik, Patrick Wegl, Mercedes Wesonig, Winter Matthias, Winter Thomas  
**Abwesend:** keiner 

---

## Themenübersicht 

### 1. Programmiervorlage
### 2. Erklärung zum Programm
### 3. Das Programm

---

## 1. Programmiervorlage
Im unterricht verwendete die Gruppe 1 eine **Programmiervorlge** in der alle grundlegenden Funktionen eingebaut 
sind die in den meisten Programmen benötigt werden. Doch falls einmal weniger Funktionen benötigt werden, zum Beispiel
für kleinere Übungsprogramme, gibt es verschiedene Kategorien von Programmiervorlagen (Level 1-Level 4). Je höher das Level
desto mehr Funktionen wurden vom Entwickler implementiert.  Weniger **Quelltext** bedeutet **schnellerer upload** in den Speicher 
des µC sowie **geringerer Platzbedarf** im Speicher.    
  
  ***Übersicht AIIT-Programmiervorage, Level 1-4:***  
  

|  Programmiervorlage  |  Inhalt  |
|------------------------|--------------------|
|Level 1 |keine Inhalte, empty |
|Level 2 |Komunikation über UART  |
|Level 3 |Level 2, Tasks |
|Level 4 |level 3, Debug möglichkeiten |

---
## 2. Erklärung zum Programm

**Zeilen 1 bis 14:**  
"Defines"; alle Bibliotheken und Dateien die im Programm verwendet wurden, wurden hier eingebunden!

**Zeile 24 bis 34:**   
Funktion "app_init", mit dem Rückgabedatentyp **"void"**, benötigt man zur Initialisierung des Programms. 
Der Speicher wird geleert. Im **ADMUX Register** wird der **Multiplexer** (Signalweiche) auf den Temperatursensor gestellt. Die interne
Referenzspannung von 1,1V wird ebenso gewählt. Weiters wird die Position (links- oder rechtsbündig) vom Messergebniss im **ADLAR Register**
auf linksbündig festgelegt.  
  
**Zeilen 39 bis 42:**  
Die Funktion **"app_inc16BitCount"** wird verwendet um einen ganzzahligen 16-bit Wert einzulesen und in einer Variable zu speichern. 
Diese VAriable wird überprüft ob sie schon ihre maximale größe von **"0xffff"** angenommen hat, wenn nicht wird sie um "1" erhöht.
Diese Funktion giebt den gespeicherten Wert am Ende wieder aus.

**Zeile 44 bis 52:**  
Die Funktion **"hex2int"** hat einen Parameter mit dem Datentyp **char** und wird verwendet um char-Werte in ganzzahlige 8-bit-Werte
umzuwandeln. Bei einem Fehler wird 0xff zurückgegeben, ansonsten wird der ganzzahlige 8-bit-Wert zurückgegeben.  

**Zeile x bis x:**  

**Zeile x bis x:**  

**Zeile x bis x:**  

**Zeile x bis x:**  

**Zeile x bis x:**  

**Zeile x bis x:**  

**Zeile x bis x:**  

**Zeile x bis x:**  




---
## 3. Das Programm
### App.c
```c 
   1 #include "global.h"
   2 
   3 #include <stdio.h>
   4 #include <string.h>
   5 
   6 #include <avr/io.h>
   7 #include <avr/interrupt.h>
   8 #include <util/delay.h>
   9 
  10 #include "app.h"
  11 #include "sys.h"
  12 
  13 // defines
  14 // ...
  15 
  16 
  17 // declarations and definations
  18 
  19 volatile struct App app;
  20 
  21 
  22 // functions
  23 
  24 void app_init (void) {
  25   memset((void *)&app, 0, sizeof(app));
  26   
  27   ADMUX = 8; // Multiplexer ADC8 = Temp.
  28   ADMUX |= (1 << REFS1) | (1 << REFS0); // VREF=1.1V
  29   ADMUX |= (1 << ADLAR); // Left Adj, -> Result in ADCH
  30   
  31   ADCSRA = (1 << ADEN) | 7; // fADC=125kHz
  32   ADCSRB = 0;
  33   
  34 }
  35 
  36 
  37 //--------------------------------------------------------
  38 
  39 uint16_t app_inc16BitCount (uint16_t cnt) {
  40     // sys_setLed(1);
  41     return cnt == 0xffff ? cnt : cnt + 1;
  42 }
  43 
  44 uint8_t hex2int (char h) {
  45     if (h >= '0' && h <= '9') {
  46         return h - '0';
  47     }
  48     if (h >= 'A' && h <= 'F') {
  49         return h - 'A' + 10;
  50     }
  51     return 0xff; // Fehler
  52 }
  53 
  54 uint8_t app_handleModbusRequest () {
  55     // printf("Request eingetroffen: %s", app.modbusBuffer);
  56     
  57     // Check if valid modbus frame
  58     char *b = app.modbusBuffer;
  59     uint8_t size = app.bufIndex;
  60     
  61     if (size < 9) { return 1; }
  62     if (b[0] != ':') { return 2; }
  63     if (b[size - 1] != '\n') { return 3; }
  64     if (b[size - 2] != '\r') { return 4; }
  65     if ( (size - 3) % 2 != 0) { return 5; }
  66     for (uint8_t i = 1; i < (size - 2); i++) {
  67         char c = b[i];
  68         if (! ((c >= '0' && c <= '9') || 
  69                (c >= 'A' && c <= 'F')))
  70             return 6;
  71     }
  72     
  73     uint8_t lrc = 0;
  74     for (uint8_t i = 1; i < (size - 4); i++) {
  75         lrc += b[i];
  76     }
  77     lrc = (uint8_t)( -(int8_t)lrc);
  78     char s[3];
  79     snprintf(s, sizeof s, "%02X", lrc);
  80     if (b[size - 4] != s[0]) { return 7; }
  81     if (b[size - 3] != s[1]) { return 7; }
  82     
  83     // printf("Request richtig\n");
  84     uint8_t i, j;
  85     for (i = 1, j = 0; i < (size - 4); i += 2, j++ ) {
  86         uint8_t hn = hex2int(b[i]);
  87         uint8_t ln = hex2int(b[i+1]);
  88         if (hn == 0xff || ln == 0xff) {
  89             return 8;
  90         }
  91         uint8_t value = hn * 16 + ln;
  92         b[j] = value;
  93     }
  94     size = j;
  95     
  96     uint8_t deviceAddress = b[0];
  97     if (deviceAddress != 1) {
  98         return 0;
  99     }
 100     
 101     uint8_t funcCode = b[1];
 102     switch (funcCode) {
 103        case 0x04: {
 104            uint16_t startAddress = b[2] << 8 | b[3];
 105            uint16_t quantity = b[4] << 8 | b[5];
 106            if (quantity < 1 || quantity > 0x7d) {
 107                 b[1] = 0x80 | b[1]; // error
 108                 b[2] = 0x03; // quantity out of range
 109                 size = 3;
 110            
 111            } else if (startAddress != 1 || quantity != 1) {
 112                 b[1] = 0x80 | b[1]; // error
 113                 b[2] = 0x02; // wrong start address
 114                 size = 3;
 115            
 116            } else {
 117                 b[2] = 2;
 118                 b[3] = app.mbInputReg01 >> 8;
 119                 b[4] = app.mbInputReg01 & 0xff;
 120                 size = 5;
 121            }
 122            break;
 123        }
 124        
 125        default: {
 126            b[1] = 0x80 | b[1]; // error
 127            b[2] = 0x01; // function code not supported
 128            size = 3;
 129        } 
 130     }
 131     
 132     lrc = 0;
 133     printf(":");
 134     for (i = 0; i < size; i++) {
 135         printf("%02X", (uint8_t)b[i]);
 136         lrc += b[i];
 137     }
 138     lrc = (uint8_t)(-(int8_t)lrc);
 139     printf("%02X", lrc);
 140     printf("\r\n");
 141     return 0;
 142 }
 143 
 144 void app_handleUartByte (char c) {
 145     if (c == ':') {
 146         if (app.bufIndex > 0) {
 147             app.errCnt = app_inc16BitCount(app.errCnt);
 148         }
 149         app.modbusBuffer[0] = c;
 150         app.bufIndex = 1;
 151     
 152     } else if (app.bufIndex == 0) {
 153         app.errCnt = app_inc16BitCount(app.errCnt);
 154  
 155     } else if (app.bufIndex >= (sizeof(app.modbusBuffer))) {
 156         app.errCnt = app_inc16BitCount(app.errCnt);
 157     
 158     } else {
 159         app.modbusBuffer[app.bufIndex++] = c;
 160         if (c == '\n') {
 161             uint8_t errCode = app_handleModbusRequest();
 162             if (errCode > 0) {
 163                 // printf("Fehler %u\n\r", errCode);
 164                 app.errCnt = app_inc16BitCount(app.errCnt);
 165             }
 166             app.bufIndex = 0;
 167         }
 168     } 
 169 }
 170 
 171 void app_main (void) {
 172     
 173     ADCSRA |= (1 << ADSC);
 174     // _delay_ms(1);
 175     // printf("ADCH=%u  ", ADCH);
 176     
 177     // Gerade: ModbusRegister = k * ADCH + d
 178     // aus Datenblatt:
 179     //     -45°C -> 242mV -> ADCH=56.79 -> MBR = -45*256 = -11520
 180     //      25°C -> 314mV -> ADCH=73.08 -> MBR =  25*256 =   6400
 181     //      85°C -> 380mV -> ADCH=88.40 -> MBR =  85*256 =  21760
 182     // daraus ergibt sich für:
 183     //      <=25°C -> MBR = k1 * ADCH + d1
 184     //      >25°C  -> MBR = k2 * ADCH + d2
 185     float k1 = 1100.061, d1 = -73992.464;
 186     float k2 = 1002.611, d2 = -66870.812;
 187 
 188     // reale Messung bei 22°C -> ADCH=88
 189     // interpoliert:
 190     //      22°C -> ?  -> ADCH=72.38 -> MBR = 22*256 = 5632
 191     // Offsetkorrektur um bei ADCH=88 auf 5632 zu kommen
 192     //     ADCH<=88 -> MBR = k1 * ADCH + d1 + o1
 193     //     ADCH>88  -> MBR = k2 * ADCH + d2 + o2
 194     float o1 = -17180.9;
 195     float o2 = -15726.956;
 196     
 197     float mbr;
 198     uint8_t adch = ADCH;
 199     // adch = 88;
 200     if (adch <= 88) {
 201         mbr = k1 * adch + d1 + o1;
 202     } else {
 203         mbr = k2 * adch + d2 + o2;
 204     }
 205     int16_t mbInputReg01 = (int16_t)mbr;
 206     int8_t vk = mbInputReg01 / 256;
 207     uint8_t nk = ((mbInputReg01 & 0xff) * 100) / 256;
 208          
 209     app.mbInputReg01 = (uint16_t)mbInputReg01;
 210     
 211     // printf("T=%.1f = %d.%02d MBR = %d = 0x%04x  \r", mbr/256.0, vk, (int)nk, mbInputReg01, (uint16_t)mbInputReg01);
 212        
 213     int c = fgetc(stdin);
 214     if (c != EOF) {
 215         // printf("\r\n %02x\r\n", (uint8_t)c);
 216         app_handleUartByte((char) c);
 217     }
 218     
 219 }
 220 
 221 //--------------------------------------------------------
 222 
 223 void app_task_1ms (void) {
 224     static uint16_t oldErrCnt = 0;
 225     static uint16_t timer = 0;
 226     if (app.errCnt != oldErrCnt) {
 227         oldErrCnt = app.errCnt;
 228         timer = 2000;
 229         sys_setLed(1);
 230     }
 231     if (timer > 0) {
 232         timer--;
 233         if (timer == 0) {
 234             sys_setLed(0);
 235         }
 236     }
 237 }
 238 
 239 
 240 void app_task_2ms (void) {}
 241 void app_task_4ms (void) {}
 242 void app_task_8ms (void) {}
 243 void app_task_16ms (void) {}
 244 void app_task_32ms (void) {}
 245 void app_task_64ms (void) {}
 246 void app_task_128ms (void) {}

```

### App.h

```c
App.h

   1 #ifndef APP_H_INCLUDED
   2 #define APP_H_INCLUDED
   3 
   4 // declarations
   5 
   6 struct App
   7 {
   8   uint8_t flags_u8;
   9   char modbusBuffer[32];
  10   uint8_t bufIndex;
  11   uint16_t errCnt;
  12   uint16_t mbInputReg01;
  13 };
  14 
  15 extern volatile struct App app;
  16 
  17 
  18 // defines
  19 
  20 #define APP_EVENT_0   0x01
  21 #define APP_EVENT_1   0x02
  22 #define APP_EVENT_2   0x04
  23 #define APP_EVENT_3   0x08
  24 #define APP_EVENT_4   0x10
  25 #define APP_EVENT_5   0x20
  26 #define APP_EVENT_6   0x40
  27 #define APP_EVENT_7   0x80
  28 
  29 
  30 // functions
  31 
  32 void app_init (void);
  33 void app_main (void);
  34 
  35 void app_task_1ms   (void);
  36 void app_task_2ms   (void);
  37 void app_task_4ms   (void);
  38 void app_task_8ms   (void);
  39 void app_task_16ms  (void);
  40 void app_task_32ms  (void);
  41 void app_task_64ms  (void);
  42 void app_task_128ms (void);
  43 
  44 #endif // APP_H_INCLUDED


```
