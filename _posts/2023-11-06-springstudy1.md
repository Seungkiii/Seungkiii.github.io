---
layout: single 

title:  "1주차 스프링 스터디 미션 정리"

---

# inteliJ 및 mysql 연결

1주차 미션은 inteliJ를 설치하고 mysql을 연동하는 부분까지 하는 것이다. 굉장히 간단한 미션이지만 나는 mysql root 비밀버호가 기억이 나질 않아 많이 힘들었다.

먼저 서비스에 들어가서 mysql을 중지시키고 txt파일을 두개 만든다

하나는 

ALTER USER 'root'@'localhost' IDENTIFIED BY '1234';

를 저장해두고 이 파일의 경로를 복사해준다



다른 하나는

"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld.exe" --defaults-file="C:\ProgramData\MySQL\MySQL Server 8.0\my.ini" --init-file="C:\Temp\init.txt"

인데 --init-file="C:\Temp\init.txt" 이부분이 아까 복사해둔 경로이다.

이제 관리자 명령프롬프트에서 두번째 txt 파일을 복붙해서 실행하면 비밀번호가 바뀌어 있을 것이다. 



이제 inteliJ로 들어가서 DB navigator를 설치하고 mysql을 연동하면 끝난다.

쉬운 미션이었지만 예상치 못한 고난으로 좀 힘들었다. 앞으로  이런일이 발생하지 않도록 비밀번호는 꼭 기억해두고 문제가 생겨도 해결방법을 찾아가는 자세와 노하우가 중요한 것 같다.

