## AOP가 필요한 상황

- 모든 메소드의 호출 시간을 측정하고 싶다면?
- 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern)
- 회원 가입 시간, 회원 조회 시간을 측정하고 싶다면?

![image](https://user-images.githubusercontent.com/66653324/104429525-0d9b4880-55c9-11eb-9142-0f752fa6e078.png)


### MemberService 회원 조회 시간 측정 추가

`MemberService.java`

```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Transactional
public class MemberService {

    private final MemberRepository memberRepository;


    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }


//    회원가입
    public Long join(Member member){

        long start = System.currentTimeMillis();

        try {
            //같은 이름이 있는 중복 회원은 X
            validateDuplicateMember(member); //중복회원 검증

            memberRepository.save(member);
            return member.getId();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;

            System.out.println("join =" +timeMs +"ms");


        }


    }

    public void validateDuplicateMember(Member member){

        memberRepository.findByName(member.getName()).ifPresent(m -> {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        });


    }

    public List<Member> findMember(){
        long start = System.currentTimeMillis();

        try {
            return memberRepository.findAll();

        }finally {

            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("findMembers" + timeMs +"ms");

        }



    }

    public Optional<Member> findOne(Long memberId){

        return memberRepository.findById(memberId);
    }

}


```

![image](https://user-images.githubusercontent.com/66653324/104459286-e441e300-55ef-11eb-890c-79d47c9c8590.png)


-----------------------------------

## AOP 적용
![image](https://user-images.githubusercontent.com/66653324/104462479-f9b90c00-55f3-11eb-965e-538df01fed24.png)


`TimeTraceAop.java`

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..)) && !target(hello.hellospring.SpringConfig)")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.currentTimeMillis();

        System.out.println("START" + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END" + joinPoint.toString() +"" + timeMs +"ms");
        }

    }
}


```

#### 실행
![image](https://user-images.githubusercontent.com/66653324/104462754-4ef51d80-55f4-11eb-9830-259c264a19ee.png)

회원목록 부분을 누르고

콘솔을 보면

![image](https://user-images.githubusercontent.com/66653324/104462902-764bea80-55f4-11eb-9f13-109238c57c6a.png)

![image](https://user-images.githubusercontent.com/66653324/104462663-3553d600-55f4-11eb-91c9-be1fa8994a09.png)



#### AOP 적용 전 의존관계
![image](https://user-images.githubusercontent.com/66653324/104463040-9f6c7b00-55f4-11eb-902b-bb23f3b5c778.png)

#### AOP 적용 후 의존관계

![image](https://user-images.githubusercontent.com/66653324/104463095-adba9700-55f4-11eb-80fb-1a749fcb5b88.png)


#### AOP 적용 전 전체그림

![image](https://user-images.githubusercontent.com/66653324/104463157-c1fe9400-55f4-11eb-9dee-2704a203027d.png)

#### AOP 적용 후 전체그림

![image](https://user-images.githubusercontent.com/66653324/104463202-d347a080-55f4-11eb-957f-e2468d9689b8.png)






