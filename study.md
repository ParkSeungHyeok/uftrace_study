# 2024.07.18 스터디 내용
1. gcc에 option으로 mcount같은 trace 관련 함수를 호출하도록 컴파일하는 것 같음
2. mcount 같은 함수를 함수프로파일러라고 하는 것 같음
3. mcount는 glibc.so(GNU  C Library)의 gmon이라는 component에서 프로파일러로 동작하는 것 같다. 
4. 여기서 .so파일이란 so 파일은 공유 라이브러리로, 여러 개의 오브젝트 파일을 하나로 묶은 동적 라이브러리이다. 이 라이브러리는 프로그램과 별도로 존재하며, 실행 시에 메모리에 로드되어 공유된다.
5. .so 파일 설명 https://snowjeon2.tistory.com/18, 동적 라이브러리 설명 https://ledpear.tistory.com/62
6. 프로파일링에 대한 설명이 https://ypangtrouble.tistory.com/entry/%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81 잘나와있다.
7. mcount 같은 함수는 ftrace라는 리눅스 커널의 내장되어있는 기능으로도 활용되는것 같다.(uftrace가 이걸보고 기능을 확장한것같아 보이기도 함)
8. 이건 preloaded라는 방식으로 lib우선순위를 높여서 사용하는것 같다.
9. ftrace에 대해서는 https://www.bhral.com/post/linux-kernel-ftrace-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%9B%90%EB%A6%AC 잘 설명되어 있음
10. uftrace를 빌드하면 libmcount 폴더에 libmcount.so파일이 생긴다.
11. makefile을 잘 분석하면 libmcount에 어떤 c파일이 들어가있는지 확인가능할듯 싶다.
