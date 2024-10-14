# 4a.m-Community1

## 진행상황


### 🔍 미리 보기
- [회원가입](#회원가입)
- [OAuth로 로그인](#OAuth로_로그인)
- [구글 로그인하기](#구글_로그인하기)
- [카카오톡 로그인하기](#카카오톡_로그인하기)
- [페이징](#페이징)
- [문제상황](#문제상황)
- [해결상황](#해결상황)


### 회원가입

![image](https://github.com/user-attachments/assets/1ff10135-f2af-4b2b-8818-a75f9d3f00ad)
- http://localhost:8080/signup 회원가입 필요
- blogdb의 users테이블에 OAuth 이메일과 비밀번호가 있어야 다음이 진행됨-> 디비에 레코드가 없어도 돌아가는 방법이 없나?

### OAuth로 로그인

![image](https://github.com/user-attachments/assets/7fc39bfb-c75a-4f7c-a49b-c28c54cdd980)
- http://localhost:8080/login

### 구글 로그인하기

![image](https://github.com/user-attachments/assets/f45e0aa3-1d51-43c2-a7fc-c76fc928e749)
- users테이블에 있는 구글 이메일로 로그인

****
![image](https://github.com/user-attachments/assets/e4a93f67-35ab-40d6-9049-b1e3d4d5dfb7)

- 비밀번호 입력
- 이때 db에서 비밀번호는 암호화되어 있음

****
![image](https://github.com/user-attachments/assets/5fe73090-f741-4033-9b17-2e1fc2e92b15)
- 계속

****
![image](https://github.com/user-attachments/assets/a9bd5ca9-48f8-43ab-82be-c05b9d1d9751)

- 글 목록

****
![image](https://github.com/user-attachments/assets/5bbd54c8-b274-48d0-b6fa-6bb40df700e2)
- 글 등록 가능

****
![image](https://github.com/user-attachments/assets/15b2df53-181b-43ec-9a38-960f48df71c8)
- 자신이 작성한(같은 이메일) 글이면 수정, 삭제 가능

****
![image](https://github.com/user-attachments/assets/d65b9408-e81a-4c90-9598-7398fd346e16)
- 다른 사용자가 작성한 글은 수정, 삭제 불가 ->다른 사용자와 본인을 구분해서 다른 사용자의 글은 수정, 삭제 버튼이 안보이게 할 순 없을까

****
### 카카오톡 로그인하기

![image](https://github.com/user-attachments/assets/9fe28100-ea9e-4fd4-9404-c4d7897698c6)
- users테이블에 있는 카카오톡 이메일로 로그인
- 이후 구글 로그인과 똑같이 됨
****

### 페이징

![image](https://github.com/user-attachments/assets/4b24a95c-2639-424e-8606-e06a401b95bf)
- 최대 10페이지가 보이게 만듦 (예시를 위해 게시글을 1개씩 보이게 했으나 실제로는 최대 10개가 보임)
- '다음' 버튼은 아직 표시되지 않은 다음 페이지 그룹이 있을 때만 보임(활성화)
- '이전' 버튼은 이전 페이지 그룹이 10 이상일 때만 보임(활성화)

![image](https://github.com/user-attachments/assets/6d3a7409-318c-4729-bdda-cb5dcf454762)
- '다음' 버튼을 클릭면 11페이지로 이동함

![image](https://github.com/user-attachments/assets/9efb2499-5096-49c8-b834-713e478207e9)
- 11페이지에서 '다음' 버튼을클릭하면 21페이지로 이동함
- 표시되지 않은 다음 페이지 그룹이 없으므로 '다음' 버튼이 보이지 않음

![image](https://github.com/user-attachments/assets/2a154142-17b9-45b0-8f74-43c54f345c9c)
- 24페이지에서 '이전' 버튼 클릭하면 20페이지로 이동

![image](https://github.com/user-attachments/assets/49802d70-c6ad-41f7-8747-7151d0d3fa27)
- 20페이지에서 '이전' 보튼 클릭하면 10페이지로 이동
- 표시되지 않은 이전 페이지 그룹이 없으므로 '이전' 버튼이 보이지 않음


****
### 문제상황

1. nickname 컬럼명이 겹치면 안됨
   - 만약에 사용자가 구글, 카카오톡 로그인을 시도할때 이름(ex. 홍길동)이 같으므로 nickname에 본인 이름이 들어간다.
 ```
  //OAuth관련키 저장
    @Column(name="nickname",unique = true)
    private String nickname;

    //생성자에 nickname 추가
    @Builder
    public User(String email, String password,String nickname) {
        this.email = email;
        this.password = password;
        this.nickname = nickname;
    }

    //사용자 이름 변경
    public User update(String nickname) {
        this.nickname = nickname;
        return this;
    }
 ```
  - 위 코드는 User 클래스 일부분
```
 private User savedOrUpdate(OAuth2User oAuth2User) {
        Map<String, Object> attributes = oAuth2User.getAttributes(); //OAuth2 사용자 정보(email, name 등)를 속성 맵으로 가져온다.

        // OAuth2User의 속성 출력
        String email;
        String name;

        if (attributes.containsKey("kakao_account")) {
            email = (String) ((Map<String, Object>) attributes.get("kakao_account")).get("email");
            System.out.println("Email from Kakao: " + email);
            name = (String) ((Map<String, Object>) attributes.get("properties")).get("nickname");

        } else {
            // 구글 사용자 정보 처리
            email = (String) attributes.get("email");
            name = (String) attributes.get("name");
        }

        User user = userRepository.findByEmail(email)
                .map(entity -> {
                    // 로그 추가
                    System.out.println("User found, updating: " + entity);
                    return entity.update(name);
                })
                ...
;
        return userRepository.save(user); //새로운 사용자 정보를 저장하거나 업데이트된 사용자 정보를 저장한다.
    }
```
- 위 코드는 Oauth2UserCustomService 클래스 일부분


![image](https://github.com/user-attachments/assets/7a1f11e9-aaa9-4fd0-94e6-950f15b47f33)
- 같은 nickname으로 로그인시 에러
  
- 결론: 같은 사용자가 구글계정에서 카카오톡 계정으로 로그인하기 위해 구글 계정 로그아웃할때, nickname이 본인 이름이 아닌 null로 업데이트되게 만들고 싶음(실패)

****
2.  blogdb의 users테이블에 OAuth 이메일과 비밀번호가 있어야함
   - 디비에 없는 계정으로 구글 로그인 시,이메일은 삽입되나 비밀번호가 디비에 삽입이 안됨
   - 이거 어짜피 구글 로그인만 성공해야 글을 작성,수정,삭제할 수 있으니까 신경 안써도 될까? (몰라)

****     
3. 로그아웃하면 세션 및 쿠키 삭제, 토큰 무효화되어야 함
   - 로그아웃되도 직전 로그인방식으로 들어가면 토큰이 살아있음...
```
        http.logout(logout -> logout
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout", "GET"))
                .addLogoutHandler(new CustomLogoutHandler(oAuth2AuthorizationRequestBasedOnCookieRepository()))
                .logoutSuccessUrl("/login"));
```
- 위 코드는 WebOAuthSecurityConfig클래스 일부분

  
```
@Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate(); // 세션 무효화
            System.out.println("Session invalidated.");
        }
        authorizationRequestRepository.removeAuthorizationRequestCookies(request, response);
        System.out.println("Authorization request cookies removed.");
        if (authentication != null) {
            SecurityContextHolder.clearContext();
            System.out.println("Security context cleared.");
        }
    }
```
- 위 코드는 CustomLogoutHandler 클래스 일부분


- 결론:
- 크롬 설정-> 개인 정보 보호 및 보안 -> 인터넷 사용 기록 삭제를 해야지만 토큰이 사라짐
- 로그아웃만으로  토큰 무효화, 쿠키 및 세션 삭제하고 싶다....

****
4. 일반 로그인으로 로그인하면 글 등록,수정,삭제가 안됨
- OAuthAndSessionSecurityConfig 클래스를 주석 해제하고 WebOAuthSecurityConfig 주석처리 후
```
<div class = "mb-4">
                    <form action="/login" method="POST">
                        <input type="hidden" th:name="${_csrf?.parameterName}" th:value="${_csrf?.token}" />
                        <div class="mb-3">
                            <label class="form-label text-white">Email address</label>
                            <input type="email" class="form-control" name="username">
                        </div>
                        <div class="mb-3">
                            <label class="form-label text-white">Password</label>
                            <input type="password" class="form-control" name="password">
                        </div>
                        <button type="submit" class="btn btn-primary">Submit</button>
                    </form>

                    <button type="button" class="btn btn-secondary mt-3" onclick="location.href='/signup'">회원가입</button>
                </div>
```
- oauth.html에 위 코드 주석 해제
![image](https://github.com/user-attachments/assets/933bd86b-91f0-4772-a93f-6dd8e84564c4)


- 일반 로그인을 하면 글 목록이 보이나 글 등록, 수정,삭제를 할 수 없음

- 결론: 일반 로그인도 액세스 토큰, 리프레시 토큰을 발급하게 만들고 싶음(실패)
****

### 해결상황

#### 페이징 넘버
![image](https://github.com/user-attachments/assets/ad1686d7-3353-49cc-837b-fcf1aa07f9b3)
- 2페이지를 클릭해도 http://localhost:8080/articles?page=1으로 로딩됨
![image](https://github.com/user-attachments/assets/561057d1-7d0d-4187-8145-06f8a40fe5e4)
- 1페이지를 클릭하면 에러 발생

  
- 초기 articleList.html 일부 코드
   ```
   <ul class="pagination justify-content-center">
       <!-- 이전 버튼 -->
       <li class="page-item" th:classappend="${page.hasPrevious()} ? '' : 'disabled'">
           <a class="page-link" th:href="@{/articles(page=${page.number > 0 ? page.number - 1 : 0}, size=${page.size})}" tabindex="-1">이전</a>
       </li>
   
       <!-- 페이지 번호 반복 -->
       <li class="page-item" th:each="i : ${#numbers.sequence(1, page.totalPages)}"
           th:classappend="${page.number + 1 == i} ? 'active' : ''">
           <a class="page-link" th:href="@{/articles(page=${i - 1}, size=${page.size})}"
              th:text="${i}"></a>
       </li>
   
       <!-- 다음 버튼 -->
       <li class="page-item" th:classappend="${page.hasNext()} ? '' : 'disabled'">
           <a class="page-link" th:href="@{/articles(page=${page.number + 1}, size=${page.size})}">다음</a>
       </li>
   </ul>
   ```

1. Spring Data JPA의 페이지 처리
    - Spring Data JPA에서 PageRequest.of(page, size) 메서드를 통해 페이지 요청을 생성할 때, 첫 번째 페이지를 0으로 처리한다.
    -  따라서 사용자가 입력하는 페이지 번호와 실제 페이지 번호 간의 불일치가 발생함
2. Thymeleaf 템플릿 코드의 오류
    - 페이지 번호 반복 처리에서 인덱스가 잘못 계산되어 페이지 번호가 올바르게 표시되지 않는다.
3. URL 파라미터 전달
    - 링크에서 페이지 번호를 0으로 설정하는 경우(예: 이전 버튼 클릭 시) page=-1 또는 page=0으로 잘못 전달되어 오류를 유발했다.
4.  첫번째 페이지 그룹에서의 '이전'버튼과 마지막 페이지 그룹에서의 '다음' 버튼은 비활성화가 됐지만 계속 보인다.


- 수정된 코드
  ![image](https://github.com/user-attachments/assets/eb359c69-2558-4bae-b02b-19e7ae02afd1)
  ```
   <nav aria-label="Page navigation">
        <ul class="pagination justify-content-center">

            <!-- 이전 그룹 버튼 -->
            <li class="page-item" th:classappend="${page.number >= 10} ? '' : 'd-none'">
                <a class="page-link" th:href="@{/articles(page=${(page.number/10)*10 }, size=${page.size})}" tabindex="-1">이전</a>
            </li>

            <!-- 페이지 번호 반복 -->
            <li class="page-item" th:each="i : ${#numbers.sequence((page.number/10)*10, ((page.number/10)*10 + 9) < page.totalPages ? (page.number/10)*10 + 9 : page.totalPages - 1)}"
                th:classappend="${page.number == i} ? 'active' : ''">
                <a class="page-link" th:href="@{/articles(page=${i+1}, size=${page.size})}" th:text="${i+1}"></a> <!-- 1-based 페이지 표시 -->
            </li>

            <!-- 다음 그룹 버튼 -->
            <li class="page-item" th:classappend="${(page.number/10)*10 + 10 < page.totalPages} ? '' : 'd-none'">
                <a class="page-link" th:href="@{/articles(page=${(page.number/10)*10 + 11}, size=${page.size})}">다음</a>
            </li>

        </ul>
    </nav>
  ```

- n페이지 http://localhost:8080/articles?page=n 
- 페이지네이션 로직을 수정하여 페이지 번호를 올바르게 계산하도록 함
- 이전, 다음 버튼 클릭 시 올바른 페이지로 이동하도록 th:classappend와 href를 수정함
- 'd-none' 사용하여 첫번째 페이지 그룹에서의 '이전'버튼과 마지막 페이지 그룹에서의 '다음' 버튼 보이지 않게 함(비활성화)
  
##### 참고
    
```
<script>
    // 페이지 번호를 로그에 출력
    document.addEventListener('DOMContentLoaded', function () {
        // Thymeleaf 변수를 JavaScript로 전달
        var currentPage = [[${page.number}]]; // 0-based 페이지 번호
        var totalPages = [[${page.totalPages}]];

        // 콘솔에 출력
        console.log("Current page number: " + currentPage);
        console.log("Total number of pages: " + totalPages);

    });
</script>
```
- 콘솔 로그 찍으면서 pageNumber 확인
