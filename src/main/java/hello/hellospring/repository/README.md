# DB 접근 기술

- 순수 jdbc
- jdbc template
- JPA



### 순수 jdbc

`JdbcMemberRepository.java`


```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.jdbc.datasource.DataSourceUtils;

import javax.sql.DataSource;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class JdbcMemberRepository implements MemberRepository {

    private final DataSource dataSource;


    public JdbcMemberRepository(DataSource dataSource){
        this.dataSource = dataSource;
    }


    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            conn = dataSource.getConnection();
            pstmt = conn.prepareStatement(sql , Statement.RETURN_GENERATED_KEYS);

            pstmt.setString(1, member.getName());

            pstmt.executeUpdate();

            rs = pstmt.getGeneratedKeys();

            if(rs.next()){
                member.setId(rs.getLong(1));
            }else{
                throw new SQLException("id 조회 실패");
            }

            return  member;

        }catch (Exception e){
            throw new IllegalStateException(e);
        }finally {
            try {
                if(rs !=null) rs.close();
                if(pstmt != null) pstmt.close();
                if(conn != null) conn.close();
            }catch(Exception e1) {
                e1.printStackTrace();
            }

        }

    }

    @Override
    public Optional<Member> findById(Long id) {

        String sql = "select * from member where id =?";

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            conn = dataSource.getConnection();
            pstmt = conn.prepareStatement(sql);

            pstmt.setLong(1, id);

            rs = pstmt.executeQuery();

            if(rs.next()){
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));

                return Optional.of(member);
            }else{
                return Optional.empty();
            }

        }catch (Exception e){
            throw new IllegalStateException(e);
        }finally {
            try {
                if(rs !=null) rs.close();
                if(pstmt != null) pstmt.close();
                if(conn != null) conn.close();
            }catch(Exception e1) {
                e1.printStackTrace();
            }

        }



    }

    @Override
    public Optional<Member> findByName(String name) {

        String sql ="select * from member where name = ?";

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, name);

            rs = pstmt.executeQuery();

            if(rs.next()){
                Member member = new Member();

                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));

                return Optional.of(member);
            }

            return Optional.empty();

        }catch (Exception e){
            throw new IllegalStateException(e);
        }finally {
            close(conn, pstmt, rs);
        }


    }

    @Override
    public List<Member> findAll() {

        String sql ="select * from member";

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);

            rs = pstmt.executeQuery();

            List<Member> members = new ArrayList<>();


            while (rs.next()){
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));

                members.add(member);


            }
                return  members;

        }catch (Exception e){
            throw new IllegalStateException(e);
        }finally {
            try {
                if(rs !=null) rs.close();
                if(pstmt != null) pstmt.close();
                if(conn != null) conn.close();
            }catch(Exception e1) {
                e1.printStackTrace();
            }

        }


    }

    private void close(Connection conn, PreparedStatement pstmt, ResultSet rs){
        try {
            if(rs !=null){
                rs.close();
            }

        }catch (SQLException e){
            e.printStackTrace();
        }
        try {
            if(pstmt !=null){
                pstmt.close();
            }

        }catch (SQLException e){
            e.printStackTrace();
        }
        try {
            if(conn !=null){
                conn.close();
            }

        }catch (SQLException e){
            e.printStackTrace();
        }
    }

    private void close(Connection conn) throws SQLException{
        DataSourceUtils.releaseConnection(conn,dataSource);
    }

    private Connection getConnection(){
        return DataSourceUtils.getConnection(dataSource);
    }
}


```


#### 스프링 변경 설정

이제 스프링 변경 설정을 해야한다.
`SpringConfig.java`

```java
package hello.hellospring;
import hello.hellospring.repository.JdbcMemberRepository;
import hello.hellospring.repository.JdbcTemplateMemberRepository;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;


@Configuration
public class SpringConfig {
 private final DataSource dataSource;
 public SpringConfig(DataSource dataSource) {
 this.dataSource = dataSource;
 }
 
 
 @Bean
 public MemberService memberService() {
 return new MemberService(memberRepository());
 }
 
 
 
 @Bean
 public MemberRepository memberRepository() {
// return new MemoryMemberRepository();
return new JJdbcMemberRepository(dataSource);
 }
}
```
- DataSource는 데이터베이스 커넥션을 획득할 때 사용하는 객체다. 스프링 부트는 데이터베이스 커넥션
정보를 바탕으로 DataSource를 생성하고 스프링 빈으로 만들어둔다. 그래서 DI를 받을 수 있다.

<br/>
스프링 변경 설정 한 뒤의 모습을 이미지로 보면!

![image](https://user-images.githubusercontent.com/66653324/104412173-197a1100-55af-11eb-9aaf-588e2d238527.png)
![image](https://user-images.githubusercontent.com/66653324/104412222-2eef3b00-55af-11eb-88d3-f0a17e4a4284.png)

### jdbc template

`JdbcTemplateMemberRepository.java`

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class JdbcTemplateMemberRepository implements MemberRepository {

   private final JdbcTemplate jdbcTemplate;

   public JdbcTemplateMemberRepository(DataSource dataSource){
       jdbcTemplate = new JdbcTemplate(dataSource);
   }

    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

        Map<String,Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());

        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters)
        );

        member.setId(key.longValue());
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {

       List<Member> result = jdbcTemplate.query("select * from member where id =?", memberRowMapper(),id);

        return result.stream().findAny();
    }

    @Override
    public Optional<Member> findByName(String name) {
       List<Member> result = jdbcTemplate.query("select * from member where name =?", memberRowMapper(), name);

        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }

    private RowMapper<Member> memberRowMapper(){
       return (rs, rowNum) -> {
           Member member = new Member();
           member.setId(rs.getLong("id"));
           member.setName(rs.getString("name"));
           return member;
       };
    }
}


```



### JPA

`JpaMemberRepository.java`
```java
package hello.hellospring.domain;

import hello.hellospring.repository.MemberRepository;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em){
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        
        Member member = em.find(Member.class,id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        
        List<Member> result = em.createQuery("select m from Member m where m.name = :name" , Member.class)
                .setParameter("name",name)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        
        
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}



```

#### 서비스 계층에 트랜잭션 추가

```java
import org.springframework.transaction.annotation.Transactional
@Transactional
public class MemberService {}
```
- org.springframework.transaction.annotation.Transactional 를 사용하자.
- 스프링은 해당 클래스의 메서드를 실행할 때 트랜잭션을 시작하고, 메서드가 정상 종료되면 트랜잭션을
  커밋한다. 만약 런타임 예외가 발생하면 롤백한다.
- JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.


#### JPA를 사용하도록 스프링 설정 변경

`SpringConfing.java`
```java
package hello.hellospring;

import hello.hellospring.domain.JpaMemberRepository;
import hello.hellospring.repository.JdbcMemberRepository;
import hello.hellospring.repository.JdbcTemplateMemberRepository;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

@Configuration
public class SpringConfig {

    private final DataSource dataSource;
    private final EntityManager em;

    public SpringConfig(DataSource dataSource ,EntityManager em){
        this.dataSource = dataSource;
        this.em = em;
    }



    @Bean
    public MemberService memberService(){

        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){


        return new JpaMemberRepository(em);
     //   return new JdbcTemplateMemberRepository(dataSource);
    //    return new JdbcMemberRepository(dataSource);
    //    return new MemoryMemberRepository();

    }
}


```


-------------------------------------------------------------
## 스프링 데이터 JPA

스프링 부트와 JPA만 사용해도 개발 생산성이 정말 많이 증가하고, 개발해야할 코드도 확연히 줄어듭니다.
여기에 스프링 데이터 JPA를 사용하면, 기존의 한계를 넘어 마치 마법처럼, 리포지토리에 구현 클래스 없이
인터페이스 만으로 개발을 완료할 수 있습니다. 그리고 반복 개발해온 기본 CRUD 기능도 스프링 데이터
JPA가 모두 제공합니다.
스프링 부트와 JPA라는 기반 위에, 스프링 데이터 JPA라는 환상적인 프레임워크를 더하면 개발이 정말
즐거워집니다. 지금까지 조금이라도 단순하고 반복이라 생각했던 개발 코드들이 확연하게 줄어듭니다.
따라서 개발자는 핵심 비즈니스 로직을 개발하는데, 집중할 수 있습니다.
실무에서 관계형 데이터베이스를 사용한다면 스프링 데이터 JPA는 이제 선택이 아니라 필수 입니다.

**주의**
 스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 기술입니다. 따라서 JPA를 먼저 학습한
후에 스프링 데이터 JPA를 학습해야 합니


`SpringDataJpaMemberRepository.java`

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long> , MemberRepository{

    Optional<Member> findByName(String name);
}


```

#### 스프링 데이터 JPA 회원 리포지토리를 사용하도록 스프링 설정 변경

`SpringConfig.java`

```java
package hello.hellospring;

import hello.hellospring.domain.JpaMemberRepository;
import hello.hellospring.repository.JdbcMemberRepository;
import hello.hellospring.repository.JdbcTemplateMemberRepository;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

@Configuration
public class SpringConfig {

   private final MemberRepository memberRepository;
   
   public SpringConfig(MemberRepository memberRepository){
       this.memberRepository = memberRepository;
   }



    @Bean
    public MemberService memberService(){

        return new MemberService(memberRepository);
    }

    
}

```

- 스프링 데이터 JPA가 SpringDataJpaMemberRepository 를 스프링 빈으로 자동 등록해준다
