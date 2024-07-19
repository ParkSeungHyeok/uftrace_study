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

![alt text](image-2.png)

6. limcount폴더의 코드들은 uftrace코드와 메모리를 공유하여 있어 .out파일을 정보를 가져올수있게 한다.

![alt text](image.png)


# 2024.07.19 스터디 내용
1. execv(opts->exename, argv)는 recode와 live에서 실행된다.
2. mcount()함수에서 call trace 관게를 알수 있는 방법은 실행(추적?)중인 함수에서 mcount()가 호출 되었을때 stack에 쌓인 인자값, 부모함수 주소, 자식함수 주소, 리턴값 주소 정보를 확인할수 있다
3. 위 과정은 [ftrace][ftracelink] 설명에 자세히 나와있다.
4. 시간값을 구하기 위해서는 실행중인 함수가 return할때 한번더 함수를 실행해줘야 하는데 이를 위해 return후 돌아가는 주소가 담기 stack에 mcount_return()주소로 바꿔치기하고 mcount_return() 돌아가는 주소를 실행함수의 돌아가는 주소 바꿔치기하여 끝나는 시점에 부모와 자식 함수 사이에 함수 호출이 가능하게 한다.

![alt text](image-3.png)

5. mcount_return()함수의 주소를 구하기 mcount_init()호출되어 mcount.S의 mcount_return 함수를 mocount_return_fn으로 넘겨준다. 이 작업 왜 필요한지 동적 라이브러리와 관련이 있는지 않을까 추정하고 있다.
6. mcount_init() 어디서 호출되는지 아직 파악이 필요한데 a.out에 __monstartup으로 점프하는 명령어가 아닐까 추정하고있다.

[so파일link]: https://snowjeon2.tistory.com/18
[프로파일링링크]: https://ypangtrouble.tistory.com/entry/%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81
[ftracelink]: https://www.bhral.com/post/linux-kernel-ftrace-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%9B%90%EB%A6%AC
[glibclink]: https://www.gnu.org/savannah-checkouts/gnu/libc/index.html
[gmonlink]: https://www.gnu.org/software/libc/manual/html_mono/libc.html
[uftrace강민철link]: https://www.youtube.com/watch?v=mLhZz0Ibpno&t=405s
[preloadlink]: https://ar9ang3.tistory.com/8
[동적라이브러리link]: https://jjang-joon.tistory.com/28


