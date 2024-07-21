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

# 2024.09.20 스터디 내용
0. 2024.07.19 스터디 내용 5. 6. 을 디스코드로 멘토님께 질문하여 답변을 받음

1. mcount_init() 함수는 a.out의 동적 라이브러리 init()에서 plt hooking을 으로 실행된다.
![alt text](image-4.png)
2. uftrace a.out의 실행 순서를 기본 기능 원리 중심으로 생각해보면 아래와 같다
    1. uftrace에서 setenv함수를 실행 LD_PRELOAD 값의 path를 libmocunt폴더로 변경하여 a.out실행시 glibc.so대신 libmcount.so가 실행되도록 설정한다.
    2. a.out가 실행되고 main()함수가 실행되기 전에 libmcount.c의 mocount_init()함수가 실행되고 이 안에서 mcount_return()의 주소를 저장함
    3. a.out main()이 실행되고 그 이후에는 각 함수(main()을 포함)가 실행될때마다 mcount()함수가 실행되어 mcount 함수에서는 입력값, 부모함수주소, 자식함수주소(본인), 반환값 주소를 확인함.
    4. 또한 자식함수주소에 리턴이후 돌아가는 함수의 주소를 mcount_retune()주소 변경하여 함수가 끝난후 mcount_return()이 실행되도록함
    5. mcount_return에서는 mocunt()가 실행된 시점의 시간과 mcount_return()실행된 시간을 차이를 구하여 자식함수의 실행시간을 측정할수 있게됨
    6. 위 과정들을 모두 메모리에 저장하고 a.out실행 종료후 저장된 메모리를 잘 가공하여 상황에 맞게 uftrace에서 출력해서 terminal로 출력해줌

[so파일link]: https://snowjeon2.tistory.com/18
[프로파일링링크]: https://ypangtrouble.tistory.com/entry/%ED%94%84%EB%A1%9C%ED%8C%8C%EC%9D%BC%EB%A7%81
[ftracelink]: https://www.bhral.com/post/linux-kernel-ftrace-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%9B%90%EB%A6%AC
[glibclink]: https://www.gnu.org/savannah-checkouts/gnu/libc/index.html
[gmonlink]: https://www.gnu.org/software/libc/manual/html_mono/libc.html
[uftrace강민철link]: https://www.youtube.com/watch?v=mLhZz0Ibpno&t=405s
[preloadlink]: https://ar9ang3.tistory.com/8
[동적라이브러리link]: https://jjang-joon.tistory.com/28


