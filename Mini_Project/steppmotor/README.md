STM32 타이머의 입력 클럭은 보통 APB 버스 클럭과 연결되는데, HCLK가 50 MHz라면 APB1/APB2 프리스케일러 설정에 따라 타이머에도 50 MHz가 들어간다고 가정할 수 있습니다.
https://controllerstech.com/interface-stepper-motor-with-stm32/
2. Prescaler (PSC)
정의

PSC 레지스터에 쓴 값 N은 실제로 (N + 1)로 나누기를 수행합니다.

즉, 타이머에 공급되는 타이머 카운터 클럭 = 입력 클럭 ÷ (PSC + 1)

예시: 1 µs 분해능

1 µs 분해능(=1 MHz 카운터 클럭)을 얻으려면

PSC + 1
=
50
 
MHz
1
 
MHz
=
50
PSC + 1= 
1MHz
50MHz
​
 =50
⇒
PSC
=
50
−
1
=
49
⇒PSC=50−1=49

이렇게 설정하면 타이머는 1 µs마다 카운터를 1씩 올립니다.

3. Counter Period (ARR)
정의

ARR(Auto-Reload Register)에 쓴 값 M까지 카운터가 올라가면, “한 사이클” 혹은 오버플로우가 발생합니다.

카운터는 0 → 1 → 2 → … → M → 0 로 반복됩니다.

최대 딜레이 한계

만약 M = 0xFFFF (65535)라면, 분해능이 1 µs일 때 최대 약 65 535 µs(≈65 ms)까지 연속 딜레이를 만들 수 있습니다.

더 긴 딜레이가 필요하면 소프트웨어로 “여러 번 호출”하거나 32-bit 타이머를 써야 합니다.

PSC는 “얼마나 분주(divide)할 것인가”를 정하고,

ARR은 “카운터를 어디까지 셀 것인가”를 정하는 값입니다.

3.1 타이머 시작
c
복사
편집
// 초기화 함수들 이후, 메인 진입부에서
HAL_TIM_Base_Start(&htim1);  
HAL_TIM_Base_Start()를 호출해야 카운터가 실제로 증가하기 시작합니다.

2. 하드웨어 연결
모터 → 드라이버: 28BYJ-48 의 5개 출력 선을 ULN2003 보드에 연결

드라이버 → MCU: ULN2003 입력 IN1∼IN4 → STM32 PA1∼PA4 (GPIO_Output)

전원: 드라이버에 5–12V DC (예: 5V 배터리) 공급

타이머 클럭: 외부 8MHz 크리스털로 SYSCLK=72MHz, APB2 타이머 클럭=72MHz

3. 마이크로초 딜레이 준비
TIM1을 APB2(72MHz)에 연결

Prescaler = 72–1 → 72MHz / 72 = 1MHz → 1 µs 분해능

ARR = 0xFFFF → 최대 ≈65 535 µs 연속 딜레이

delay(uint16_t us) 함수:

c
복사
편집
void delay(uint16_t us) {
  __HAL_TIM_SET_COUNTER(&htim1, 0);
  while (__HAL_TIM_GET_COUNTER(&htim1) < us);
}
4. RPM 제어
c
복사
편집
#define STEPS_PER_REV 4096  // Half-drive 모드 전체 스텝 수

void stepper_set_rpm(int rpm) {
  // 1분 = 60,000,000 µs → (60e6 / STEPS_PER_REV / rpm) µs 딜레이
  delay(60000000 / STEPS_PER_REV / rpm);
}
60000000은 1분(µs)

이만큼 딜레이를 넣어야 지정한 rpm 속도로 회전

5. Half-Drive 1 스텝 시퀀스
c
복사
편집
void stepper_half_drive(int step) {
  switch(step) {
    case 0: // IN1=1, 나머지=0
      HAL_GPIO_WritePin(GPIOA, PIN1, SET);
      ...
step 0~7까지 각각 다른 코일 활성화 패턴

8개 패턴을 연속 호출하면 1 시퀀스(≈0.703°) 회전

6. 지정 각도, 방향 제어
c
복사
편집
void stepper_step_angle(float angle, int dir, int rpm) {
  int seqCnt = angle / 0.703125;  // 필요한 시퀀스 수
  for(int s=0; s<seqCnt; s++) {
    if(dir==0) // 시계
      for(int st=7; st>=0; st--) {
        stepper_half_drive(st);
        stepper_set_rpm(rpm);
      }
    else       // 반시계
      for(int st=0; st<8; st++) {
        stepper_half_drive(st);
        stepper_set_rpm(rpm);
      }
  }
}
angle 만큼 회전하기 위해 필요한 시퀀스 수만큼 반복

dir(0=시계,1=반시계) 따라 스텝 순서 역전

7. 현재 위치 추적
c
복사
편집
float currentAngle = 0;

void Stepper_rotate(int targetAngle, int rpm) {
  int delta = targetAngle - currentAngle;
  if(delta > 0.71)
    stepper_step_angle(delta, 0, rpm);
  else if(delta < -0.71)
    stepper_step_angle(-delta, 1, rpm);
  currentAngle = targetAngle;
}
매 호출 시 currentAngle을 업데이트 → 절대 좌표 제어 가능

8. 메인 루프 예시
c
복사
편집
int main() {
  // ... 초기화
  HAL_TIM_Base_Start(&htim1);
  while(1) {
    for(int i=0; i<=360; i++) {
      Stepper_rotate(i, 10);  // 0→360° 천천히
      HAL_Delay(250);         // 250 ms 휴식
    }
    for(int i=360; i>=0; i--) {
      Stepper_rotate(i, 10);  // 360→0° 역회전
      HAL_Delay(250);
    }
  }
}
0°→360°→0° 를 반복하며 천천히 회전

