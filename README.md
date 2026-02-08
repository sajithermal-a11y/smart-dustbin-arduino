# smart-dustbin-arduino
#The Smart Dustbin is an automatic, touchless waste disposal system designed using Arduino, an ultrasonic sensor, and a servo motor. The dustbin lid opens automatically when a person or object is detected near it and closes after a short delay. This system helps maintain hygiene and reduces the spread of germs.

#include <Servo.h>

Servo dustbinServo;

const int trigPin = 6;
const int echoPin = 7;
const int servoPin = 8;
const int ledPin  = 5;

long duration;
int distance;

int openAngle = 120;        // lid open position
int closeAngle = 180;      // lid close position
int openDistance = 15;     // open if hand is within 15 cm
int closeDistance = 25;    // close when hand goes away (hysteresis)

int currentAngle = closeAngle;
bool lidOpen = false;      // lid status

// Smooth servo move function
void smoothMove(int targetAngle) {
  if (currentAngle < targetAngle) {
    for (int pos = currentAngle; pos <= targetAngle; pos++) {
      dustbinServo.write(pos);
      delay(10);
    }
  } else {
    for (int pos = currentAngle; pos >= targetAngle; pos--) {
      dustbinServo.write(pos);
      delay(10);
    }
  }
  currentAngle = targetAngle;
}

// Function to measure distance
int getDistanceCM() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH, 30000); // timeout 30ms

  if (duration == 0) return 999;  // no object detected
  return duration * 0.034 / 2;
}

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledPin, OUTPUT);

  dustbinServo.attach(servoPin);
  dustbinServo.write(closeAngle);
  currentAngle = closeAngle;

  digitalWrite(ledPin, LOW);

  Serial.begin(9600);
}

void loop() {
  distance = getDistanceCM();

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // OPEN condition
  if (!lidOpen && distance > 0 && distance <= openDistance) {
    digitalWrite(ledPin, HIGH);   // LED ON
    smoothMove(openAngle);        // open smoothly
    lidOpen = true;
  }

  // CLOSE condition (only when hand removed)
  if (lidOpen && distance >= closeDistance) {
    digitalWrite(ledPin, LOW);    // LED OFF
    smoothMove(closeAngle);       // close smoothly
    lidOpen = false;
  }

  delay(150);
}
