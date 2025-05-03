인터럽트 원리는 사진에서 볼 수 있듯이 기존의 TCNT 레지스터값이 정해진다.
내가 사용하고 있는 STM32F407의 경우 168MHZ이고 이걸 넘어서게 되면 OverFlow되어 다시 0으로 떨어지게 된다.
우리는 특정한 타임을 정하고 싶기 때문에 AutoReload Register Period를 설정하게 된다. 
다만 앞으로 사용하게 될 APB1 TIM CLK=84MHZ, APB2 TIM CLK=168MHz로 상당히 빠르기 때문에 시스템 과부화를 필연적으로 야기하게 된다.
따라서 우리는 Prescler를 활용해서 해당 흐름을 낮춰주게 할 것이다.

TIMCLK=84MHz/10000으로 해서 (1/8400)s으로 clk이 작동하게 되고 register period를 8400으로 설정하게 하면 1초를 달성하는 식으로 하게 되는 것이다.

Interrupt 실행하는 방법은 은근 간단하다. 


1. Pin Configuration을 한다. 이번 건은 external_pin을 사용했다.
2. NVIC가 정의된 함수로 이동=> 활성화 잘 되어있는지 확인한다.
3. it.c 파일로 가서 void EXTI3_IRQHandler(void) 이런 IRQHandler를 찾는다. 거기서 내부 IRQHandler 경로를 타고 들어간다
4. 거기 보면은 callback함수가 있을텐데 그거를 또 타고 들어간다
5. 거기에 정의되어있는 _weak함수를 main함수로 끌고와서 정의할 수 있도록 한다.

6. void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  if(GPIO_Pin==GPIO_PIN_3)
  {
	  HAL_GPIO_TogglePin(GPIOD,GPIO_PIN_13);
  }

  if(GPIO_Pin==GPIO_PIN_10)
   {
	  HAL_GPIO_TogglePin(GPIOD,GPIO_PIN_14);
   }

}
이런 식으로 말이다.
그러면 외부 인터럽트를 구현할 수 있게 된다.
