사진에서 볼 수 있듯이 주기를 바꾸는 방법에는 auto를 바꾸거나 prescale을 바꿀 수 있다.
다만, auto를 바꾸면 주기를 조절할 수 있는 폭이 줄어들기 때문에 prescale을 바꾸는 것이 강력히 권장된다. (prescale 늘리면 auto reload register값이 줄어듬)
그리고 현재 사용할 timer의 경우 channel을 4개 까지 설정할 수 있는데, 이는 duty cycle을 한 scale안에 4개까지 설정할 수 있다고 보면 된다.
따라서 그 경우 Counter Period를 조절하며 duty cycle을 조절할 수 있는 것이다.

