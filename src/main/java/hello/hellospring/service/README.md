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
