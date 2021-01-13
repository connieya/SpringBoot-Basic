## 스프링 빈과 의존관계

- [컴포넌트 스캔과 자동 의존관계 설정](https://github.com/gunny6026/SpringBoot-Basic/tree/master/src/main/java/hello/hellospring)


---------------------------------

- 자바코드로 직접 스프링 빈 등록하기

회원 서비스와 회원 리포지토리의 @Service , @Repository , @Autowired 어노테이션을 제거하고 진행한다.
<br/>

그리고 SpringConfig.java 를 만든다.

```
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService(){

        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){

        return new MemoryMemberRepository();

    }
}

```
`참고` : XML로 설정하는 방식도 있지만 최근에는 잘 사용하지 않으므로 생략한다.

<br/>
`참고` : DI에는 필드 주입, setter 주입, 생성자 주입 이렇게 3가지 방법이 있다. 의존관계가 실행중에 동적으로 변하는 경우는 거의 없으므로 생성자 주입을 권장한다.

- 필드 주입
```
 @Autowired private  MemberService memberService
```

- setter 주입

```
 @Autowired
    public void setMemberService(MemberService memberService){
        this.memberService = memberService;
    }
```

- 생성자 주입

```
 @Autowired
    public MemberController(MemberService memberService){
        
        this.memberService = memberService;
    }
```

`참고` : 실무에서는 주로 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용한다. 그리고 정형화되지 않거나 상황에 따라 
구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다.

`주의` : *@Autowired* 를 통한 DI는 *helloController* , *MemberService* 등과 같이 스프링이 관리하는 객체에서만 동작한다. 스프링 빈으로 등록하지 않고
내가 직접 생성한 객체에서는 동작하지 않는다.

---------------------------------------------

## 회원관리 : 등록 & 조회

- [회원관리](https://github.com/gunny6026/SpringBoot-Basic/tree/master/src/main/java/hello/hellospring/controller)
- [DB접근 기술](https://github.com/gunny6026/SpringBoot-Basic/tree/master/src/main/java/hello/hellospring/repository)


## AOP
- [AOP](https://github.com/gunny6026/SpringBoot-Basic/tree/master/src/main/java/hello/hellospring/service)
