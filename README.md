# Mini-Gaming-Console
This repository consist of code for my Mini Gaming Console Project.
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define BTN 2
#define CELL 4
#define MAX_LEN 50

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// -------- MENU --------
int menuIndex = 0;
bool inGame = false;

// -------- SNAKE --------
int snakeX[MAX_LEN], snakeY[MAX_LEN];
int length = 5;
int foodX, foodY;
int dir = 1;
unsigned long lastMove = 0;
int snakeScore = 0;

// -------- PONG --------
int paddleY = 25;
int ballX = 64, ballY = 32;
int dx = 2, dy = 2;
int pongScore = 0;

// -------- CATCH --------
int basketX = 50;
int fallX = 30, fallY = 0;
int catchScore = 0;

// -------- INIT SNAKE --------
void initSnake() {
  length = 5;
  dir = 1;
  snakeScore = 0;

  for (int i = 0; i < length; i++) {
    snakeX[i] = 40 - i * CELL;
    snakeY[i] = 20;
  }

  foodX = random(0, 30) * CELL;
  foodY = random(0, 15) * CELL;
}

// -------- MENU --------
void drawMenu() {
  display.clearDisplay();
  display.setCursor(20, 10);
  display.print(menuIndex == 0 ? "> Snake" : "  Snake");

  display.setCursor(20, 25);
  display.print(menuIndex == 1 ? "> Pong" : "  Pong");

  display.setCursor(20, 40);
  display.print(menuIndex == 2 ? "> Catch" : "  Catch");

  display.display();
}

// -------- SNAKE GAME --------
void snakeGame() {
  int xVal = analogRead(A0);
  int yVal = analogRead(A1);

  if (xVal < 300 && dir != 1) dir = 3;
  else if (xVal > 700 && dir != 3) dir = 1;
  else if (yVal < 300 && dir != 2) dir = 0;
  else if (yVal > 700 && dir != 0) dir = 2;

  if (millis() - lastMove > 120) {
    lastMove = millis();

    for (int i = length - 1; i > 0; i--) {
      snakeX[i] = snakeX[i - 1];
      snakeY[i] = snakeY[i - 1];
    }

    if (dir == 0) snakeY[0] -= CELL;
    if (dir == 1) snakeX[0] += CELL;
    if (dir == 2) snakeY[0] += CELL;
    if (dir == 3) snakeX[0] -= CELL;

    if (snakeX[0] < 0) snakeX[0] = 124;
    if (snakeX[0] > 124) snakeX[0] = 0;
    if (snakeY[0] < 0) snakeY[0] = 60;
    if (snakeY[0] > 60) snakeY[0] = 0;

    if (snakeX[0] == foodX && snakeY[0] == foodY) {
      if (length < MAX_LEN) length++;
      snakeScore++;
      foodX = random(0, 30) * CELL;
      foodY = random(0, 15) * CELL;
    }
  }

  display.clearDisplay();

  for (int i = 0; i < length; i++) {
    display.fillRect(snakeX[i], snakeY[i], CELL, CELL, WHITE);
  }

  display.fillRect(foodX, foodY, CELL, CELL, WHITE);

  // Score
  display.setCursor(0, 0);
  display.print("Score:");
  display.print(snakeScore);

  display.display();
}

// -------- PONG GAME --------
void pongGame() {
  int yVal = analogRead(A1);
  paddleY = map(yVal, 0, 1023, 0, 54);

  ballX += dx;
  ballY += dy;

  if (ballY <= 0 || ballY >= 63) dy *= -1;

  if (ballX <= 5 && ballY >= paddleY && ballY <= paddleY + 10) {
    dx = 2;
    pongScore++;
  }

  if (ballX < 0) {
    ballX = 64;
    ballY = 32;
    pongScore = 0;
  }

  if (ballX >= 127) dx = -2;

  display.clearDisplay();

  display.fillRect(0, paddleY, 3, 10, WHITE);
  display.fillRect(ballX, ballY, 3, 3, WHITE);

  // Score
  display.setCursor(0, 0);
  display.print("Score:");
  display.print(pongScore);

  display.display();
}

// -------- CATCH GAME --------
void catchGame() {
  int xVal = analogRead(A0);

  if (xVal < 400) basketX -= 3;
  if (xVal > 600) basketX += 3;

  if (basketX < 0) basketX = 0;
  if (basketX > 108) basketX = 108;

  fallY += 3;

  if (fallY > 64) {
    if (fallX >= basketX && fallX <= basketX + 20) {
      catchScore++;
    } else {
      catchScore = 0;
    }
    fallY = 0;
    fallX = random(0, 120);
  }

  display.clearDisplay();

  display.fillRect(basketX, 60, 20, 3, WHITE);
  display.fillCircle(fallX, fallY, 2, WHITE);

  // Score
  display.setCursor(0, 0);
  display.print("Score:");
  display.print(catchScore);

  display.display();
}

void setup() {
  pinMode(BTN, INPUT_PULLUP);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.setTextColor(WHITE);
  display.setTextSize(1);
  randomSeed(analogRead(A3));

  initSnake();
}

void loop() {

  if (!inGame) {
    int yVal = analogRead(A1);

    if (yVal < 300) { menuIndex--; delay(200); }
    else if (yVal > 700) { menuIndex++; delay(200); }

    if (menuIndex < 0) menuIndex = 2;
    if (menuIndex > 2) menuIndex = 0;

    drawMenu();

    if (digitalRead(BTN) == LOW) {
      inGame = true;

      if (menuIndex == 0) initSnake();
      if (menuIndex == 1) pongScore = 0;
      if (menuIndex == 2) catchScore = 0;

      delay(300);
    }
  }
  else {
    if (menuIndex == 0) snakeGame();
    if (menuIndex == 1) pongGame();
    if (menuIndex == 2) catchGame();

    if (digitalRead(BTN) == LOW) {
      inGame = false;
      delay(300);
    }
  }

  delay(30);
}
