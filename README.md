### 개발일지

## 1일차

Service는 혼자서 DB 작업을 할 수 없기때문에 Repository가 반드시 필요한다.

그래서 Service 계층에서 객체 생성을 하고 @RequiredArgsConstructor를 사용하여
Service 생성자에 Repository를 param으로 받게 만든다. (this.repository = repository로 되어있음)

Controller에서 이 서비스를 불러올때도 똑같이 @RequiredArgsConstructor를 사용해서 만든다

Spring은 이렇게 생성자를 만들었을때 main을 실행하는 순간
@Repository, @Service @RestController가 붙은 클래스를 찾아가 생성자에 파라미터를 직접 넣어준다.

Board board = boardRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

findById(id)는 값이 있을 수도 없을수도 있는 상자인 Optional<Board로 돌려준다.> 여기서 Board는 Entity
id가 없을때 orElseThrow가 발생되고 예외를 던져준다.

orElseThrow 안에 ()는 매개변수가 없다는 뜻 (람다식)

---

Controller에서 findbyid로 지정한 id의 컬럼들을 보고 싶을땐 그냥 dto 객체만 반환 시키면 
spring이 알아서 json으로 자동 변환 시켜줌.

---

- 서비스
 @Transactional
    public List<Board> findAll(){
        return boardRepository.findAll();
    }
- 컨트롤러
@GetMapping
    public List<Board> findAllView(){
        return boardService.findAll();
    }

이렇게 쓰면 Entity를 리스트로 그대로 반환하는 보안문제가 생긴다
예시로 Entity 데이터베이스 구조에 비밀번호가 있다고 치자 그럴때 이 코드를 쓰면 비밀번호가 유출되는 사고가
일어날 수 있다.

또 @Transactional은 데이터를 변경할때 실패하면 되돌리기 위해 쓰는거여서 안써도 된다
대신 @Transactional(readOnly = true)를 써야함.

이 문제를 해결하기 위해서 서비스 계층에 findAll 메서드를 바꿔야한다
반환값을 Entity 말고 Dto로 바꿔야 하는데 바꾸기 위해선 .stream(), .map(), .toList가 필요했다.
.stream은 list, set 등 데이터를 람다식으로 map, filter, collect 할때 필요하다
.map은 데이터 하나하나 꺼내서 어떤 값으로 바꾸기 가능
.toList는 리스트로 반환하기
map 안에 람다식으로 board -> new BoardResponseDto(board)라고 썼었으나 
람다식에서 파라미터를 생성자에 바로 넘겨주는 기본 람다식은 클래스명 :: new로 축약 가능하다.
(Board :: new)

---

jpa 개념 이해
@Entity가 데이터베이스와 1ㄷ1 매핑할 수 있게 중간에서 관리자 역할을 해주는 거가 jpa이다.
SQL 자바 객체를 통역해주는 역할을함


---
# 2일차 개발 일지
---
## 목적
회원가입 로직에 암호화를 추가 할것임.
---
## 배운것

단방향 암호화에 대한 개념을 배왔다

암호화엔 단방향 암호화, 양방향 암호화가 있다
단방향은 암호화를 하고 복화화는 못하는 방식이고
양방향은 암호화도 되고 복호화도 되는 방식이다

해시함수 SHA 다이제스트 레인보우 테이블 키 스트래칭, 솔트

이때 단방향 암호화의 종류중에는 '해시'가 있다.

해시 함수는 고정되지 않은 길이의 문자열을 고정된 길이의 데이터로 매핑하는 기술이고
SHA란 기술을 쓴다

SHA의 방식을 쉽게 설명해 보자면

1. 입력받은 문자열을 바이너리 코드로 변환한다
2. 바이너리 코드를 512비트의 블락으로 나눈다 블락이 512비트보다 작으면 자동으로 늘려주고
크면 512비트에서 자른다.
3. 그 다음 32비트의 블락으로 또 나눈다
4. 블락마다 압축을 64번정도 한다
5. 그 후 블락별로 해시값을 생성하는데 첫번째 블락의 해시 결과가 두번째 블락에 입력되고
두번째 블락의 해시는 첫번째와 두번째를 합친거와 같다
6. 최대 256비트로 해시 생성됨

이렇게 생성된 해시를 다이제스트라고 함.

하지만 이런 다이제스트를 모아둔 사이트가 있는데 그 사이트가 레인버우 테이블이다

이러한 문제를 보완하기 위해서

키 스트래칭이나, 솔트란 방법을 쓴다.

키 스트래칭은 해시를 n번 반복해서 해시를 만드는것이고

솔트는 기본 입력값에 문자를 추가하여 해시를 하는 방법이다 (사용자마다 솔트값은 다 달라야함)

@Bean은 외부 라이브러리/패키지의 객체를 스프링에 등록하고 구성하는 방법

---
이러한 방식이 스프링부트에는 BcryptPasswordEncoder로 만들어져있음

이처럼 만들어진 외부라이브러리를 가져오려면 Config 과정이 필요하다
Spring 컨테이너에 Bean을 등록하기 위해서 @Configuration이 필요함.
@Bean은 자바객체를 컨테이너에 넣기 위해 사용함

SecurityConfig
스프링 시큐리티를 추가할시 모든 url이 잠금되기 때문에
어떤 url은 잠겨있지 않고 어떤 url은 잠겨있는지 구성하기위해 필요함

---
## 다음에 할 것
회원가입 시스템에 암호화 과정까지 만들었기 때문에
다음엔 로그인 시스템을 만들 것이다.

---
# 3일차 개발 일지

---
## 목적
로그인 기능 추가

---
## 배운 것

쿠키, 세션, (토큰)JWT에 대해 배웠다

http는 로그인을 해도 다음으로 넘어가면 로그인 정보를 기억 못하는 무상태성이란 특징을 가지고있다.

로그인을 하면 세션 서버에 세션이 생성되고 로그인을 하면 세션id가 포함된 쿠키를 웹 브라우저에
전달 함으로써 무상태성을 극복할 수 있다. 

1. 로그인 
2. 비번이 맞는지 확인 후 세션ID 생성  
3. 서버가 쿠키를 웹브라우저에 전달 (쿠키엔 세션ID정보가 저장되어 있음)
4. 다음 요청부터 받은 쿠키를 서버에 보냄
5. 서버가 쿠키에있는 세션ID 정보를 세션DB와 매칭시켜서 누구인지 확인

쿠키, 세션과 달리 토큰은 신분증 같은거다.
로그인을 하면 엄청나게 긴 String(토큰)을 암호화 해 클라이언트에 준다. 토큰엔 사용자 정보가 들어있음.
클라이언트가 요청을 보낼때 토큰을 보내기 때문에 DB를 뒤질 필요가 없음
서버는 토큰이 조작되지 않았는지만 확인한다.

쿠키 & 세션과 토큰 방식의 차이

쿠키&세션 방법은 요청 받을때 DB에서 찾아야하지만
토큰은 DB같은거 필요없고 토큰이 맞는지만 검증

---
## 다음에 할 것

레파지토리 따로 만들어서 로그인 기능 제대로 공부하기

---

## 4일차 개발일지

---

## 목적
회원가입까지 최대한 직접 만들어 보고 JPA버리고 myBatis로 MySQL 연동하기
로그인 구현

---

## 배운것

JPA와 달리 myBatis는 레파지토리 대신 mapper란걸 쓴다.
mapper도 인터페이스인데 mysql과 연결하기 위해선 .xml 파일을 만들거나 어노테이션을 써서 간편하게 쓰는 방법이 있다.
나는 그 중에서 .xml 파일을 만드는 방법을 사용했다.

이 방법에 대해 설명을 하자면
.xml엔 mapper에 있는 메서드 안에 sql문을 작성해야하고 태그 규칙이 담겨있는 링크를 넣어야함
또 mysql에 매핑하기 위해선 환경변수가 담겨있는 application.properties에 나의 mysql 링크와 이름, 비번까지 넣어야함

controller에서 login 부분에 
``` java 
HttpServletRequest request
```
란 파라미터를 넣는다 (요청 받을때 정보를 함께 받아옴)
그리고 
```java 
HttpSession session = request.getSession();
```
을 사용 함으로서 세션을 생성하고 변수에 넣는다

``` java 
session.setAttribute("memberId",memberId);
```
그 다음 로그인한 id를 세션에 연결시켜서 판별 가능하게함 
그리고 getsession 만들때 쿠키도 만들어서 마지막에 웹브라우저로 보내줌

---
## 다음에 할 것

프론트 엔드 연결해보기

---
# 5일차

---

## 목적
세션 체크, 로그아웃 만들고 ai를 통해 프론트 엔드 연결해 볼거임
CRUD 시스템 전부 다 만들기

---
## 배운 것

로그아웃은 세션을 지워서 기억 못하는 상태로 바꾸는 방법이다.

프론트 엔드에 연결하기 위해선 `CORS`란 걸 설정해줘야 한다
브라우저는 다른 출처로 보내는 요청을 함부로 허용하지 않게 되어있는데 
이 정책을 `CORS`라고 부른다
따라서 다른 출처에 요청을 보내기 위해서 허용된 출처를 선언해줘야한다

프론트엔드에서 서버에 요청을 보내면 서버에서 일을 처리해 응답을 하고 그 응답을 받은 브라우저가 CORS여부를 확인한 후 보낼지 말지를 정함

`@ExceptionHandler`에 대해서도 배웠다
이 어노테이션은
new illegalstate()를 쓰거나 new illegalargument를 쓸때 이 어노테이션이 없으면 무조건 500코드만 반환을 하기 때문에
각자 어떤 오류코드를 반환 하게 할지 정할 수 있게 만든다.

---
## 내가 혼자 고친 버그
ai를 통해 프론트 엔드를 만들었을때 url 연동이 잘못되어있던걸 고쳤고
수정 로직을 짤때 비번도 바꿀 수 있게 만들어 놨는데 encode를 까먹고 안했어서
데이터베이스에 비번이 바로 가게 되고 인코드 안되어있어서 match도 안됐던걸 고쳤다

---
## 다음에 할 것
로그인 세션 쿠키 방법을 토큰(JWT) 방식으로 바꿔볼 것이다

---
# 6일차

---
## 목적

jwt(토큰) 방법으로 로그인 만들어보기

---
## 배운 것

토큰은 로그인할때 한번 발급받는다 
그 후 사용자가 토큰을 보내면 서버가 제대로된 토큰이 맞고 사용자가 누구인지 알아냄으로써
HTTP 무상태성을 극복할 수 있다.

세션은 서버가 로그인 정보를 저장하지만
JWT는 서버가 아무것도 저장하지 않고
토큰 자체가 정보를 들고 다닌다.
  
토큰은 Header - Payload - signature로 이루어져 있다

Header는 토큰을 만들때 어떤 알고리즘을 썼는지가 있다.
Payload엔 사용자 정보가 있다. memberId, issuedAt 같은것들
signature엔 header와 payload securityKey가 해시 되어 들어있다

``` java
public String createToken(Long memberId){
        return Jwts.builder()
                .claim("memberId",memberId)
                .issuedAt(new Date(System.currentTimeMillis()))
                .expiration(new Date(System.currentTimeMillis() + expirationTime))
                .signWith(securityKey)
                .compact();
    }
```
header는 자동으로 만들어져 있음
payload는 .claim, .issuedAt, .expiration
signature은 signWith를 쓸 시 만들어짐
이거를 .compact해서 토큰 생성!

jwt 방식은 로그인 할때 딱 한번 토큰을 생성하고
filter를 이용하여 여러번 코드를 안써도 되게 할 수 있다

OncePerRequestFilter를 상속하여 doFilterInternal을 오버라이드 해야 필터 기능을 할 수 있음

HttpRequest 받아서 토큰 받기 -> 토큰 memberId로 바꾸기 -> Spring Security가 이해할 수 있게 정해진 클래스 UsernamePasswordAuthenticationToken을 사용해 인증 받기
-> SecurityContextHolder에 등록해서 Spring Security가 이사람은 로그인된 사람이란걸 알게 해준다.
이런식으로 이루어져 있다

securityconfig에 .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class); 이걸 넣음으로써 필터가 적용되게 했다

## 에러
@RequiredArgsConstructor를 사용해 매개변수를 받는 생성자를 만드는데
실행할때 그 매개변수에 있는게 spring 컨테이너에 들어가있어야한다 
하지만 filter만들때 @component 안써서 컨테이너에 안들어가서 오류생김

---
## 다음에 할 것

로그인 시스템을 다 만들었으므로 스프링부트 새 레파지토리를 만들거다
레파지토리를 너무 많이 만들어서 폴더에 넣을거임

---
# 7일차

---
## 목적 
레파지토리 만들고 스프링 구조 이해 하기

---
## 배운 것

서버를 시작하면 tomcat(was)가 켜지고 Spring이 어노테이션을 스캔함
그리고 url -메서드 매핑표를 만들고 매핑표를 서블릿에 등록
url 요청이 들어올때 tomcat이 요청을 받고 서블렛한테 넘기고 알맞은 메서드로 안내를 해준 후 실행
tomcat은 모든 과정이 돌아가는 엔진 역할을 한다

---
## 다음에 할 것

게시판 만들기

---
# 8일차

---
## 목적
게시판 CRUD 만들고
프론트엔드 조금 공부해보기
리눅스 서버 연결도 해보기

---
## 배운 것

리눅스에 서버 연결을 할때
git clone 레파지토리 주소로 가져온 후
./gradlew build로 빌드를 해야한다.
이 빌드를 하고 java -jar --.jar을 쓰면 서버 실행이됨.

---
## 오류

오늘 서버연결을 하면서 오류가 났었다
문제는 세션 전달 오류였는데
내가 프론트엔드를 인텔리제이에 만들어놨던 html로 켜서 문제가 생겼음
세션이 전달되지 않은 이유는 인텔리제이는 localhost:63342여서 리눅스 서버랑 주소가 아에 다르기 때문에 세션 전달이 안된거임

해결 방법 html을 resource/static에 넣고 url을 "/api/board", "/api/member"로 바꿈으로써
윈도우 브라우저에서 리눅스ip:8080/login.html로 들어가고 로그인할때 세션 연결도 잘 되게 바뀌었다

f12 누르고 network에서 info에서 쿠키 전달 안된게 보였다

집에서도 우분투 서버 가동 해봤는데 오류가 났음
이번엔 집에 네트워크 어뎁터 설정이 nat이 되어있어서 났음 (bridge)로 바꿈
그리고 방화벽 8080포트가 안열려있어서 ufw 8080포트 열게 바꿈

```
nat은 윈도우 안에 우분투가 있다는 뜻
ubuntu는 윈도우 밖으로 나갈 수 있는데 윈도우는 우분트로 못들어감

그래서 bridge로 연결함
bridge는 윈도우랑 우분투랑 나란히 있는 구조여서 아이피 주소만 알면 바로 들어갈 수 있음
```

---
## 다음에 할 것

Docker 해보기

---
# 9일차

---

## 배운 것

docker는 다른 컴퓨터에 환경설정을 그대로 옮겨주는거.
그리고 가상환경을 실행하지 않고 필요한것만 커널에 붙여서 리소스 안잡아먹게 도와주는 역할을 한다

docker는 
docker file, image, container로 구성되어 있는데
docker file엔 어떤 방식으로 환경을 설정할지 방법이 들어있고 image는 그 방밥을 통해 만든 결과물이다
그 image를 통해 container를 만들 수 있다.

그리고 docker는 클라이언트 - 서버 구조이다.

docker run을 실행하면 docker deamon에서 이미지로 컨테이너를 통해 실행한다
docker build는 docker file을 통해 image를 만든다
docker pull은 다른사람이 만든 이미지를 registry에서 가져와 이미지를 얻는다

docker file만 우리가 쓰고
docker run, docker build, docker pull은 다 docker engine의 역할이다.

docker가 가벼운 이유는 리눅스에 있는 chroot, namespace, cgroup을 조합해 하이퍼바이저가 필요 없게 만들어서 그렇다.
하이퍼 자이저란 vmwrae같이 새로운 컴퓨터를 생성할 수 있게 도와주는거

---

그리고 또 다른 네트워크에서 내 사이트 들어오는 방법중
포트 포워딩, ngrok을 배웠다

포트 포워딩은 공인 ip를 불러 특정 포트로 연결 됐을때 어떤 ip로 이동해야하는지 공유기 설정을 바꾸는 방법이였다
ngrok는 임의의 주소를 만들어서 다른 네트워크도 다 들어올 수 있게 만드는 방법 이다

맨 처음엔 포트 포워딩 방법을 쓰려고했으나 공유기 비번이 계속 틀려서 ngrok 방법을 택했다

ngrok 하는 방법은 쉬웠다 토큰 설정하고 포트 연결이 끝이기 때문이다

---

## 오류 고치기

하지만 ngrok으로 사이트를 열고 뜻밖의 문제가 발생을 했는데 바로 cors 오류였다 나는 static에 html을 넣어서 상대경로로 주소를 써서 이런 문제가 발생하지 않을 줄 알았는데 static 상대경로 방법은 브라우저 cors를 방지하는 방법이였고 spring security 즉 서버에서 cors 오류가 난거였다

``` java
config.setAllowedOrigins(List.of(
                "http://localhost:63342",
                "https://hungrily-elf-repossess.ngrok-free.dev"
        ));
        config.setAllowedOriginPatterns(List.of("*"));
```
이렇게 고침

그리고 보통 공유기에는 AP 격리라고 같은 와이파이를 가진 사람들이 마음대로 그 ip에 접근 못하게 막는다는걸 알았다
