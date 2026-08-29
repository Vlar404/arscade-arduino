#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// === ПРОТОТИПЫ ===
void showStartupAnimation();
void showInsertCoin();
void playGame();
void showGameOver();
void resetBall();
void resetBricks();
int readDistance();

// === ЭКРАН ===
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// === УПРАВЛЕНИЕ ===
const int axisX = A0;
int valX = 0;
const int trigPin = 9;
const int echoPin = 8;

// === РАКЕТКА ===
int paddleX = 54;
const int paddleWidth = 20;
const int paddleHeight = 4;
const int paddleY = 58;

// === МЯЧИК ===
int ballX = 64;
int ballY = 40;
int ballSpeedX = 2;
int ballSpeedY = -2;
const int ballSize = 2;

// === КИРПИЧИКИ ===
const int brickRows = 3;
const int brickCols = 8;
const int brickW = 14;
const int brickH = 6;
const int brickGap = 2;
const int brickStartX = 2;
const int brickStartY = 18;
bool bricks[brickRows][brickCols];

// === СЧЁТ ===
int score = 0;
int highScore = 0;
int finalScore = 0;
int lives = 3;
bool ballHitPaddle = false;

// === СОСТОЯНИЯ АВТОМАТА ===
// 0 = ждём монету, 1 = играем, 2 = game over
int gameState = 0;
unsigned long stateTime = 0;
bool sensorWasFree = true;
bool blinkOn = false;
unsigned long blinkTime = 0;
bool startupShown = false;

void setup() {
  Serial.begin(9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    for(;;);
  }
  display.clearDisplay();
  resetBricks();
  showStartupAnimation();
  startupShown = true;
  gameState = 0;
}

void loop() {
  if (!startupShown) return;
  if (gameState == 0)      showInsertCoin();
  else if (gameState == 1) playGame();
  else                     showGameOver();
}

// === УЛЬТРАЗВУК ===
int readDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH, 30000);
  return (int)(duration * 0.034 / 2);
}

// === ЗАСТАВКА ===
void showStartupAnimation() {
  for(int progress = 0; progress <= 100; progress += 5) {
    display.clearDisplay();
    display.drawRect(0, 0, 128, 64, WHITE);
    display.setTextSize(1);
    display.setTextColor(WHITE);
    display.setCursor(10, 20);
    display.print("SYSTEM STATUS:");
    display.drawRect(10, 32, 108, 14, WHITE);
    int barWidth = map(progress, 0, 100, 0, 104);
    display.fillRect(12, 34, barWidth, 10, WHITE);
    display.setCursor(10, 50);
    display.print("Progress: ");
    display.print(progress);
    display.print("%");
    display.display();
    delay(150);
  }
  delay(1000);
  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(12, 20);
  display.print("Hello");
  display.setCursor(12, 42);
  display.print("World!");
  display.display();
  delay(3000);
}

// === ЭКРАН "ВСТАВЬ МОНЕТУ" ===
void showInsertCoin() {
  int dist = readDistance();

  if (dist > 25) sensorWasFree = true;

  if (sensorWasFree && dist > 2 && dist < 15) {
    sensorWasFree = false;
    Serial.println("COIN ACCEPTED!");
    score = 0;
    lives = 3;
    resetBricks();
    resetBall();
    paddleX = 54;
    gameState = 1;
    return;
  }

  if (millis() - blinkTime > 500) {
    blinkOn = !blinkOn;
    blinkTime = millis();
  }

  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(16, 8);
  display.print("BREAKOUT");

  display.setTextSize(1);
  display.setCursor(2, 30);
  display.print("HI-SCORE: ");
  display.print(highScore);

  if (blinkOn) {
    display.setCursor(26, 44);
    display.print("INSERT COIN");
  }

  display.setCursor(14, 56);
  display.print("(wave hand)");
  display.display();
}

// === ИГРА ===
void playGame() {
  valX = analogRead(axisX);
  if (valX > 600 && paddleX < SCREEN_WIDTH - paddleWidth) {
    paddleX += 4;
  }
  else if (valX < 400 && paddleX > 0) {
    paddleX -= 4;
  }

  ballX += ballSpeedX;
  ballY += ballSpeedY;

  if (ballX <= ballSize) { ballX = ballSize; ballSpeedX = -ballSpeedX; }
  if (ballX >= SCREEN_WIDTH - ballSize - 1) { ballX = SCREEN_WIDTH - ballSize - 1; ballSpeedX = -ballSpeedX; }
  if (ballY <= ballSize) { ballY = ballSize; ballSpeedY = -ballSpeedY; }

  if (ballSpeedY > 0 &&
      ballY + ballSize >= paddleY &&
      ballY - ballSize <= paddleY + paddleHeight &&
      ballX >= paddleX &&
      ballX <= paddleX + paddleWidth &&
      !ballHitPaddle) {
    ballSpeedY = -ballSpeedY;
    int hitPos = ballX - (paddleX + paddleWidth / 2);
    ballSpeedX = hitPos / 4;
    ballHitPaddle = true;
  }
  if (ballY < paddleY) ballHitPaddle = false;

  for(int row = 0; row < brickRows; row++) {
    for(int col = 0; col < brickCols; col++) {
      if(bricks[row][col]) {
        int bx = brickStartX + col * (brickW + brickGap);
        int by = brickStartY + row * (brickH + brickGap);
        if(ballX + ballSize >= bx && ballX - ballSize <= bx + brickW &&
           ballY + ballSize >= by && ballY - ballSize <= by + brickH) {
          bricks[row][col] = false;
          ballSpeedY = -ballSpeedY;
          score += 10;
          if(score > highScore) highScore = score;
        }
      }
    }
  }

  bool allDestroyed = true;
  for(int row = 0; row < brickRows; row++)
    for(int col = 0; col < brickCols; col++)
      if(bricks[row][col]) allDestroyed = false;
  if(allDestroyed) {
    score += 100;
    if(score > highScore) highScore = score;
    resetBricks();
    resetBall();
  }

  if (ballY > SCREEN_HEIGHT) {
    finalScore = score;
    score = 0;
    lives--;
    if (lives <= 0) {
      gameState = 2;
      stateTime = millis();
    } else {
      resetBall();
    }
  }

  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(2, 2);
  display.print("SCORE:");
  display.print(score);
  display.setCursor(70, 2);
  display.print("HI:");
  display.print(highScore);
  display.drawLine(0, 16, 127, 16, WHITE);
  display.drawRect(0, 17, SCREEN_WIDTH, SCREEN_HEIGHT - 17, WHITE);

  for(int row = 0; row < brickRows; row++) {
    for(int col = 0; col < brickCols; col++) {
      if(bricks[row][col]) {
        int bx = brickStartX + col * (brickW + brickGap);
        int by = brickStartY + row * (brickH + brickGap);
        display.fillRect(bx, by, brickW, brickH, WHITE);
      }
    }
  }

  display.fillRect(paddleX, paddleY, paddleWidth, paddleHeight, WHITE);
  display.fillCircle(ballX, ballY, ballSize, WHITE);

  display.setCursor(2, 55);
  for(int i = 0; i < lives; i++) display.write(3);

  display.display();
  delay(15);
}

// === GAME OVER ===
void showGameOver() {
  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(15, 10);
  display.print("GAME");
  display.setCursor(15, 30);
  display.print("OVER");
  display.setTextSize(1);
  display.setCursor(10, 50);
  display.print("Score: ");
  display.print(finalScore);
  display.display();

  if (millis() - stateTime > 3000) {
    gameState = 0;
    sensorWasFree = true;
  }
}

void resetBall() {
  ballX = 64;
  ballY = 40;
  ballSpeedX = 2;
  ballSpeedY = -2;
  ballHitPaddle = false;
}

void resetBricks() {
  for(int row = 0; row < brickRows; row++)
    for(int col = 0; col < brickCols; col++)
      bricks[row][col] = true;
}
