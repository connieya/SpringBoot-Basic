## 컴포넌트 스캔과 자동 의존관계 설정

**MemberController.java**
```java
@Controller
public class MemberController {

}
```

아직 아무 기능이 없지만
@Controller 어노테이션을 생성하면
스프링 컨테이너에 MemberController 객체를 넣어 둔다.<br/>
그러면 이제 스프링이 이 객체를 관리한다.


**MemberController.java**
```java
@Controller
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService){
        this.memberService = memberService;
    }

}

```

스프링이 컨테이너가 뜰 때 MemberController가 생성되고
이때 생성자를 호출한다.

**@Autowired** 어노테이션
스프링이 MemberService를 가져다 연결 시켜준다.

하지만 실행 해보면

![image](https://user-images.githubusercontent.com/66653324/103364258-a11c4600-4b00-11eb-8525-391e32dff47a.png)


MemberService를 찾을 수 없다 라는 오류가 뜬다.

왜??<br/>
*MemberService 가 스프링 빈으로 등록되어있지 않기 때문에*  <br/><br/><br/>

**MemberService.java**
```java
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
    
    }

```
그냥 일반적인 평범한 java 클래스이다. <br/>

![image](https://user-images.githubusercontent.com/66653324/104458307-8a8ce900-55ee-11eb-9ad1-4cca11911c9d.png)


@Autowired을 사용하여 두 클래스를 연결하려면
빈으로 등록하기 위한 어노테이션이 필요하다.<br/>

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
    
    }

```
![image](https://user-images.githubusercontent.com/66653324/103364666-bfcf0c80-4b01-11eb-826b-dafe80d8ef2f.png)

@Service @Repository 등등 어노테이션을 사용하여 MemberController가 생성이 될 때 <br/>
스프링 빈에 등록되어 있는 MemberService 객체를 가져다 넣어준다. <br/>

이것이 바로 `dependecy injection (의존성 주입)` 이다. <br/>

의존 관계를 주입해주는 것 (스프링이 넣어준다.)<br/><br/>

이것이 _스프링 빈을 등록하는 2가지 방법_ 중 
- 컴포넌트 스캔과 자동 의존관계 설정 이다.

컴포넌트 스켄
@Component 어노테이션을 붙이는 것!!

@Controller ,@Service , @Repository 안에 다 @Component 어노테이션이 있다.

예시 한개 만 보면 

@Controller 어노테이션 안에 보면
![image](https://user-images.githubusercontent.com/66653324/103365554-ad55d280-4b03-11eb-8fa1-987a7c209066.png)<br/><br/>
@Componet 어노테이션이 있다.

그래서 스프링이 실행되면 컴포넌트 스캔을 통해 의존성 주입이 가능한 것이다.

<br/>
그림으로 보면


![image](https://user-images.githubusercontent.com/66653324/104458474-cb84fd80-55ee-11eb-8a57-5cd5c17a0cce.png)

![image](https://user-images.githubusercontent.com/66653324/104458573-e7889f00-55ee-11eb-8b0f-acc7abe53289.png)

