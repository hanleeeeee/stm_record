DMA는 Direct Memory Access로 memory에 넣을때 코드를 작성할 필요 없이 자동으로 메모리에 값이 복사될 수 있도록 하는 기능이다.
기존에는 ad 변환이 완료되면 adc 결과를 하나씩 저장해야 했는데, DMA 방식을 쓰면 AD 변환이 완료되면 ADC 결과를 DMA  컨트롤러가 자동으로 원하는 변수에 저장하게 해준다. 
사진 참고

ADC->DR을 하게 되면 자동으로 ADCVal에 넣어준다. 
dma는  cpu의 클럭 소모 없이 복사할 수 있는 기능이 있다. 

그리고 ADC Configuration에서 이제 설정해야 하는 것이 굉장히 많다.

1. clock prescaler의 경우 가장 작게 해주는 4를 선택하고
Resolution의 경우 12bits
Scan conversion Mode의 경우, adc채널이 여러개 있을때, 자동으로 여러개가 따다닥 된다는 것
continuous conversion mode 4개 끝나면 다시 새로운 4개를 할거냐
dma continuous requests: dma 쓰면 해야함
number of conversion: adc 몇개 쓸거냐