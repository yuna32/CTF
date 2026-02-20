Phantom_Signal
======

제공된 pcap파일을 와이어샤크로 열어보면 https와 ssdp 패킷만이 있다. 이 중 단 세개 있는 ssdp 패킷에서 정보를 얻어내야 할 것으로 보인다. 


<img width="862" height="363" alt="스크린샷 2026-02-07 164623" src="https://github.com/user-attachments/assets/04ff1275-cbf4-4a59-a1a9-5edfa8bb6ba6" />


<img width="801" height="373" alt="스크린샷 2026-02-07 164652" src="https://github.com/user-attachments/assets/4dfffc5b-33c7-46b7-b952-d198c42cb752" />


<img width="726" height="354" alt="스크린샷 2026-02-07 164708" src="https://github.com/user-attachments/assets/06b527b3-2602-416d-b083-f29670134abd" />


각 패킷의 raw data를 살펴보면 rv: 필드에 데이터가 숨겨져 있다. 이걸 이어붙여 base64로 디코딩하면 플래그가 나온다. 
