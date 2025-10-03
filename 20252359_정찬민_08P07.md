// Arduino pin assignment
#define PIN_LED  9
#define PIN_TRIG 12 
#define PIN_ECHO 13

// configurable parameters
#define SND_VEL 346.0     
#define INTERVAL 25      
#define PULSE_DURATION 10  

// 실측 범위 반영 (100mm ~ 300mm)
#define _DIST_MIN 100.0   // 최소 거리 (mm)
#define _DIST_MAX 300.0  // 최대 거리 (mm)

#define SCALE (0.001 * 0.5 * SND_VEL) 

unsigned long last_sampling_time = 0;  
void setup() {
  pinMode(PIN_LED, OUTPUT);
  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  digitalWrite(PIN_TRIG, LOW);  
  Serial.begin(57600);
}

void loop() { 
  if (millis() < (last_sampling_time + INTERVAL))
    return;

  float distance = USS_measure(PIN_TRIG, PIN_ECHO); // read distance
  int duty = 0;

  if (distance <= _DIST_MIN) {
    duty = 255;  
  } 
  else if (distance >= _DIST_MAX) {
    duty = 255;
  }
  else if (distance > _DIST_MIN && distance < 200) {
    duty = map(distance, 100, 200, 255, 0);
  } 
  else if (distance > 200 && distance < 300) {
    duty = map(distance, 200, 300, 0, 255);
  } 
  else {
    duty = 255; // 안전장치
  }

  analogWrite(PIN_LED, duty);

  // Debug 출력
  Serial.print("Distance(mm): ");
  Serial.print(distance);
  Serial.print(", Duty: ");
  Serial.println(duty);

  last_sampling_time += INTERVAL;
}

float USS_measure(int TRIG, int ECHO)
{
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(PULSE_DURATION);
  digitalWrite(TRIG, LOW);
 
  unsigned long duration = pulseIn(ECHO, HIGH, 10000); 
  if (duration == 0) {
    return _DIST_MAX + 10; 
  }
  return duration * SCALE; // mm
}
