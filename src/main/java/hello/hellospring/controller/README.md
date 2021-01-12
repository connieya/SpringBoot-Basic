## 회원가입

회원가입 페이지 url

```java
@GetMapping("/members/new")
    public String createForm(){

        return  "members/createMemberForm";
    }

```



회원가입 등록 url
```java
  @PostMapping("/members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }

```


회원목록 페이지 호출 url

```java
 @GetMapping("/members")
    public String list(Model model){
        List<Member> members = memberService.findMember();
        model.addAttribute("members",members);

        return "members/memberList";
    }

```


home.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
   <div class="container">
       <h1>Hello Spring</h1>
       <p>회원 기능</p>
       <p>
           <a href="/members/new">회원 가입</a>
           <a href="/members">회원 목록</a>
       </p>

   </div>  <!--/container-->
</body>
</html>

```

회원목록 페이지

memberList.html

```html
<!DOCTYPE html>
<html xmlns:th="http:/www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div class="container">
    <div>
        <table>
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"> </td>
            </tr>
            </tbody>
        </table>

    </div>

</div>

</body>
</html>
```
