#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <avr/wdt.h>

const int pin2 = 2;
const int pin3 = 3;
const int pin4 = 4;
const int pin5 = 5;

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

int stan = 1 ;
bool selected = false ;

// graty systemowe -gra

int faza = 0 ;
int score = 0 ;
int liczba_x = 10;
int liczba_y = 50;
int screenState = 0;
int stara_liczba_x = 0;
int stara_liczba_y = 0;

// kwadrat - ten co goni
struct Box {
  int x, y;
  bool active;
};
Box box = { 64, 40, true};

// serce 

const uint8_t heartBitmap[] PROGMEM = {
  0b00000000,
  0b01100110,
  0b11111111,
  0b11111111,
  0b01111110,
  0b00111100,
  0b00011000,
  0b00000000
};
struct Heart {
  int x, y;
  bool active;
};
Heart heart = { 60, 30, true };

// bramki 

struct Gate {
  int  y_up , y_down;
};

// bramik ustawianie pozycji
  Gate gate1 ;
  Gate gate2 ;
  Gate gate3 ;
//---------------------------------------------------------------------------------------------------------------------

void setup() {

  screenState = 10;   
  pinMode(pin2, INPUT_PULLUP);
  pinMode(pin3, INPUT_PULLUP);
  pinMode(pin4, INPUT_PULLUP);
  pinMode(pin5, INPUT_PULLUP);

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);

  randomSeed(analogRead(A0));
}

void loop() {

if (stan <= 3){
  if (digitalRead(pin2) == LOW) {
    stan--;
    delay(150);
  }

  if (digitalRead(pin5) == LOW) {
    stan++;
    delay(150);
  }
  if (digitalRead(pin3) == LOW) {
    if(selected == true){
    selected = false;
    }else if (selected == false){
    selected = true;
    }
    delay(150);
  }
  
  // ograniczenie menu 
  if (stan < 1) stan = 1;
  if (stan > 3) stan = 3;
}

  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);

  if (selected == false) {

    display.print("GATERUNNER\n");

    if (stan == 1) display.print(">"); else display.print(" ");
    display.print("START\n");

    if (stan == 2) display.print(">"); else display.print(" ");
    display.print("RULES\n");

    if (stan == 3) display.print(">"); else display.print(" ");
    display.print("QUIT\n");

  }
else {
//---------------------------------------------------------------------------------------------------------------------
if (stan == 1){
  stan = 4 ;
while (stan == 4){
// poruszanie się gracza , klatka

  if (digitalRead(pin2) == LOW) {  stara_liczba_x = liczba_x;  stara_liczba_y = liczba_y;
liczba_y--; delay(3);}
  if (digitalRead(pin3) == LOW) {  stara_liczba_x = liczba_x;  stara_liczba_y = liczba_y;
liczba_x++; delay(3);}
  if (digitalRead(pin4) == LOW) {  stara_liczba_x = liczba_x;  stara_liczba_y = liczba_y;
liczba_x--; delay(3);}
  if (digitalRead(pin5) == LOW) {  stara_liczba_x = liczba_x;  stara_liczba_y = liczba_y;
liczba_y++; delay(3);}
  if (liczba_x < 0) liczba_x = 0;
  if (liczba_x > 127) liczba_x = 127;
  if (liczba_y < 20) liczba_y = 20;
  if (liczba_y > 63) liczba_y = 63;



  // punktacja ogólna
  if(faza == 0){
  gate1.y_up = random(16, 46);
  gate1.y_down = (gate1.y_up + 20);
  gate2.y_up = random(16, 46);
  gate2.y_down = (gate2.y_up + 20);
  gate3.y_up = random(16, 46);
  gate3.y_down = (gate3.y_up + 20);
  faza = 1 ;};
  if ((liczba_x > 120) && (faza == 1)) {score = (score + 10);faza = 2;
  gate1.y_up = random(16, 46);
  gate1.y_down = (gate1.y_up + 20);
  gate2.y_up = random(16, 46);
  gate2.y_down = (gate2.y_up + 20);
  gate3.y_up = random(16, 46);
  gate3.y_down = (gate3.y_up + 20);}
  if ((liczba_x < 10) && (faza == 2)) {score = (score + 10);faza = 1;
  gate1.y_up = random(16, 46);
  gate1.y_down = (gate1.y_up + 20);
  gate2.y_up = random(16, 46);
  gate2.y_down = (gate2.y_up + 20);
  gate3.y_up = random(16, 46);
  gate3.y_down = (gate3.y_up + 20);}

  // --- RUCH KWADRATU ---
if (box.active) {
  static int counter = 0;
  counter++;

  if (box.active && counter >= 4) {
  counter = 0;

  if (box.x > liczba_x) box.x--;
  else if (box.x < liczba_x) box.x++;

  if (box.y > liczba_y) box.y--;
  else if (box.y < liczba_y) box.y++;
}
}
// --- kolizja kwadrat 
if (box.active &&
    liczba_x >= box.x && liczba_x < box.x + 8 &&
    liczba_y >= box.y && liczba_y < box.y + 8) {

  selected = false;
  stan = 4;

  while (selected == false){

    if (digitalRead(pin2) == LOW) {
      stan--;
      delay(150);
    }

    if (digitalRead(pin5) == LOW) {
      stan++;
      delay(150);
    }

    if (digitalRead(pin3) == LOW) {
      selected = true;
      delay(150);
    }
    if (stan != 4){
    if (stan < 1) stan = 1;
    if (stan > 2) stan = 2;
    }
    display.clearDisplay();
    display.setTextSize(2);
    display.setCursor(0,0);

    display.print("Game over\nScore:");
    display.print(score);
    display.print("\n");

    if (stan == 1) display.print(">"); else display.print(" ");
    display.print("Restart\n");

    if (stan == 2) display.print(">"); else display.print(" ");
    display.print("Main menu");

    display.display();

    if (stan == 4){
      delay(400);
      stan = 1 ;
    }
  }

if (stan == 1){

  // reset gry
  score = 0;
  faza = 0;

  liczba_x = 10;
  liczba_y = 50;

  box.x = 64;
  box.y = 40;

  heart.x = 60;
  heart.y = 30;

  stan = 4;
}

  if (stan == 2){
    wdt_enable(WDTO_15MS);
    while (1) {}
  }
}
  
  // --- kolizja serce

   if (heart.active &&
    liczba_x >= heart.x && liczba_x < heart.x + 8 &&
    liczba_y >= heart.y && liczba_y < heart.y + 8) {

  score = (score + 5);
  heart.x = random(20, 100);
  heart.y = random(20, 60);
}

 // --- kolizje bramki

if (liczba_x == 30 &&
   (liczba_y < gate1.y_up || liczba_y > gate1.y_down)) {
  liczba_x = stara_liczba_x;
  liczba_y = stara_liczba_y;
}

if (liczba_x == 60 &&
   (liczba_y < gate2.y_up || liczba_y > gate2.y_down)) {
  liczba_x = stara_liczba_x;
  liczba_y = stara_liczba_y;
}

if (liczba_x == 90 &&
   (liczba_y < gate3.y_up || liczba_y > gate3.y_down)) {
  liczba_x = stara_liczba_x;
  liczba_y = stara_liczba_y;
}



//-------------------

  display.clearDisplay();

  // --- TEKST ---
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.print("SCORE ");
  display.print(score);

  display.drawLine(0, 15, 128, 15, SSD1306_WHITE);

  // bramki
  display.drawLine(30, 16, 30, gate1.y_up, SSD1306_WHITE);
  display.drawLine(30, 63, 30, gate1.y_down, SSD1306_WHITE);
  display.drawLine(60, 16, 60, gate2.y_up, SSD1306_WHITE);
  display.drawLine(60, 63, 60, gate2.y_down, SSD1306_WHITE);
  display.drawLine(90, 16, 90, gate3.y_up, SSD1306_WHITE);
  display.drawLine(90, 63, 90, gate3.y_down, SSD1306_WHITE);


  // gracz
  display.drawPixel(liczba_x, liczba_y, SSD1306_WHITE);

  // kwadrat

  if (box.active) {
    display.fillRect(box.x, box.y, 7, 7, SSD1306_WHITE);
  }



  if (heart.active) {
  display.drawBitmap(heart.x, heart.y, heartBitmap, 8, 8, SSD1306_WHITE);
}

  display.display();

}
}
//---------------------------------------------------------------------------------------------------------------------


  if (stan == 2){
    display.setTextSize(1);
    display.print("Go through left to \nright and back.\nAvoid square if you \ndon't want to lose.\nGood luck : )\n");
  }

  if (stan == 3){

    display.clearDisplay();
    display.setTextSize(1);
    display.setCursor(0, 0);
    display.print("Game will turn shut down \nPress any button to  turn it on ");

    display.display();

    delay(3000);

    display.clearDisplay();
    display.display();
    while (digitalRead(pin2) == HIGH &&
           digitalRead(pin3) == HIGH &&
           digitalRead(pin4) == HIGH &&
           digitalRead(pin5) == HIGH) {
    }
    wdt_enable(WDTO_15MS);
    while (1) {}
  }
  display.display();
}

if (stan <= 3){
display.display();
}

}