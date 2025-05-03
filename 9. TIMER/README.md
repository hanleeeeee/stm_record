인터럽트 원리는 사진에서 볼 수 있듯이 기존의 TCNT 레지스터값이 정해진다.
내가 사용하고 있는 STM32F407의 경우 168MHZ이고 이걸 넘어서게 되면 OverFlow되어 다시 0으로 떨어지게 된다.
우리는 특정한 타임을 정하고 싶기 때문에 AutoReload Register Period를 설정하게 된다. 
다만 앞으로 사용하게 될 APB1 TIM CLK=84MHZ, APB2 TIM CLK=168MHz로 상당히 빠르기 때문에 시스템 과부화를 필연적으로 야기하게 된다.
따라서 우리는 Prescler를 활용해서 해당 흐름을 낮춰주게 할 것이다.

TIMCLK=84MHz/10000으로 해서 (1/8400)s으로 clk이 작동하게 되고 register period를 8400으로 설정하게 하면 1초를 달성하는 식으로 하게 되는 것이다.
