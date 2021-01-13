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