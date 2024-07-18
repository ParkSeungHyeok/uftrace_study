# 2024.07.17 스터디 내용
1. gcc에 option으로 mcount같은 trace 관련 함수를 호출하도록 컴파일하는 것 같음([youtube 영상으로 학습][uftrace강민철link])
2. mcount 같은 함수를 함수[프로파일러][프로파일링링크]라고 하는 것 같음
3. mcount는 [glibc][glibclink][.so][so파일link](GNU  C Library)의 [gmon][gmonlink]이라는 component에서 프로파일러로 동작하는 것 같다. 
4. mcount 같은 함수는 [ftrace][ftracelink]라는 리눅스 커널의 내장되어있는 기능으로도 활용되는것 같다.(uftrace가 이걸보고 기능을 확장한것같아 보이기도 함)
5. 이건 preload라는 방식으로 lib우선순위를 높여서 사용하는것 같다.
6. uftrace를 빌드하면 libmcount 폴더에 libmcount.so파일이 생긴다.
7. makefile을 잘 분석하면 libmcount에 어떤 c파일이 들어가있는지 확인가능할듯 싶다.


# 2024.07.18 스터디 내용
1. [preload][preloadlink]의 개념을 이해하기 위해서는 [동적라이브러리][동적라이브러리link]의 개념부터 이해가 필요하다
2. 환경 변수 LD_PRELOAD를 변경하여 preload를 하는것 같다
3. getenv, setenv를 환경변수관련 라이브러리함수를 사용하여 설정한다.
![alt text](image-1.png)
4. 환경변수 설정이후 execv(filename)으로 .out파일을 실행시킨다.
5. 여기서 .out파일의 mcount()가 mcount.s로 부터 시작하여 libmcount폴더의 mcount_entry함수로 이어진다.
6. limcount폴더의 코드들은 uftrace코드와 메모리를 공유하여 있어 .out파일을 정보를 가져올수있게 한다.
![alt text](image.png)


[so파일link]: https://snowjeon2.tistory.com/18
[프로파일링링크]: https://ypangtrouble.tistory.com/entry/%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81
[ftracelink]: https://www.bhral.com/post/linux-kernel-ftrace-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%9B%90%EB%A6%AC
[glibclink]: https://www.gnu.org/savannah-checkouts/gnu/libc/index.html
[gmonlink]: https://www.gnu.org/software/libc/manual/html_mono/libc.html
[uftrace강민철link]: https://www.youtube.com/watch?v=mLhZz0Ibpno&t=405s
[preloadlink]: https://ar9ang3.tistory.com/8
[동적라이브러리link]: https://jjang-joon.tistory.com/28