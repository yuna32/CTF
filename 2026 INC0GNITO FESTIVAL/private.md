web3: private
========

제공된 sol 파일은

```sol
pragma solidity 0.8.13;

contract FlagContract {

    string private flag = "INCOGNITO{REDACTED}";

    constructor() {
        flag = "YOU_SHALL_NOT_PASS";
    }
    function getFlag() public view returns (string memory) {
    return flag;
    }
}
```

이후 Sepolia Etherscan에 접속해서 트랜잭션을 추적한다. 


<img width="636" height="166" alt="image" src="https://github.com/user-attachments/assets/f7ef9ea1-123b-4988-8084-6a3a7ac0c9c5" />

이렇게 힌트가 나와있으므로 address 를 따라서 찾아가보면   



<img width="1742" height="718" alt="image" src="https://github.com/user-attachments/assets/96cf1abe-d429-498c-9450-30f6f66c2cea" />

이렇게 나온다. contact creator를 따라가보면   

<img width="1755" height="340" alt="스크린샷 2026-02-07 145357" src="https://github.com/user-attachments/assets/0c71f7d8-03ed-42bb-a361-1889560d6122" />

이렇게 나온다. 여기서 위쪽에 있는 hash를 클릭 


<img width="1633" height="342" alt="image" src="https://github.com/user-attachments/assets/33baf67f-92b5-4cb9-be2f-645c857adabe" />

그러면 input data 맨 마지막에 플래그 형태로 존재하고 base64돌려주면 플래그가 나온다. 
