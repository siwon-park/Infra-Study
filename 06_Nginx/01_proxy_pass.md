# 01_proxy_pass

> Nginx Proxy Pass

Nginx가 제공하는 Reverse Proxy 기능으로 클라이언트의 요청에 대해서 특정 서비스(서버)로 접근할 수 있도록 프록시 패스를 지정할 수 있다.

<br>

## 1. Forward Proxy와 Reverse Proxy

Forward Proxy

Reverse Proxy

<br>

## 2. location 블록

server 블록 안에 있으며 location 블록에서 특정 uri에 대한 요청을 처리할 수 있다.

<br>

## 3. regex

### set

> 변수를 초기화하는 데 사용

```nginx
set $[변수이름] [값];
set var1 "server";
```

<br>

### 패턴 매칭 우선 순위

#### 1순위: `=`

정규식이 정확하게 일치하는 경우(exactly)

`location = [패턴] {...}`형태로 사용

```nginx
# /admin, /admin?param은 가능하지만, /admin/, /admins 등은 매칭 불가능함
location = /admin {
    
}
```

#### 2순위: `^~`

우선순위를 부여하고, 정규식 앞부분이 일치하는 경우

`location ^ ~ [패턴] {...}` 형태로 사용

※ `^ ~`와 다른 개념임

```nginx
location ^~/admin {
    
}
```

#### 3순위: `~`

정규식 대소문자가 일치하는 경우

`location ~ [패턴] {...}`형태로 사용

```nginx
# /admin, /admin?param에 대해서는 매칭되지만, /ADMIN, /admin/, /admind 등에 대해서는 매칭되지 않음
location ~^/admin$ {
    
}
```

#### 4순위: `~*`

대소문자를 구분하지 않고 일치

`location ~*[패턴]`형태로 사용

```nginx
location ~*.(jpg|jpeg|png|gif) {
    
}

# /admin, /ADMIN, /admin?param 등에 대해서는 일치하지만, /admin/, /ADMIN/, /admind 등에 대해서는 매칭되지 않음
location ~* ^/admin$ {
    
}
```

#### 5순위: `/`

`location /[패턴] {...}`형태로 사용

```nginx
# admin, admin?param, admin/, admind 등에 대해서 매칭
location /admin {
    
}
```

<br>

### ^

> 시작

`^[문자열]`형태로 사용하여 특정 문자열로 시작하는 패턴으로 사용

```
예시: ^a
- 패턴과 일치하는 경우: apple, air
- 패턴과 불일치하는 경우: siwon, park
```

<br>

### $

> 끝

`$[문자열]`형태로 사용하여 특정 문자로 끝나는 패턴으로 사용

```
예시: n$
- 패턴과 일치하는 경우: sun, siwon
- 패턴과 불일치하는 경우: apple, air
```

<br>

### .

> 모든 문자

`.`단독으로 쓰여 모든 문자를 의미함

```
예시: nginx.
- 패턴과 일치하는 경우: nginx, nginxw, nginx1
- 패턴과 불일치하는 경우: ngin, ngix
```

<br>

### []

> 집합

`[특정 문자 - 문자]`나 `[숫자-숫자]`형태로 사용하여 해당 집합 내의 패턴과 일치 유무를 판별하기 위해 사용

```
예시: nginx[a-c123-]
- 패턴과 일치하는 경우: nginxa, nginx1, nginx2
- 패턴과 불일치하는 경우: nginxz, nginx5 nginxo
```

<br>

### [^ ]

> 부정집합

집합의 반대 개념. 특정 메타문자에 포함되지 않는 패턴으로 사용

<br>

### |

> 또는(OR)

메타문자별로 OR

<br>

### ()

> 그룹

여러 패턴 요소를 한 단위로 묶음. 주로 `|` 과 함께 사용됨

```nginx
(jpg|jpeg|png|gif)
```

<br>

### \

> 이스케이프

`\`뒤 문자열에 대해 특수문자를 인식할 수 있도록 처리함(=문자열 그 자체로 처리)

<br>

### *

> 없거나 1회 이상의 문자열이 `*` 자리에 존재

```nginx
# 예시
ng*inx # nginx, ngxxx..inx와 매칭
.* # 모든 문자가 최소 0개 이상
```

<br>

### +

> `+`자리에 최소 1개 이상의 문자열이 존재

```nginx
# 예시
ng+inx # ngxxx..inx와 매칭 # nginx와는 불일치
```

<br>

### ?

> `?`자리에 없거나 1개의 문자열이 존재

```nginx
# 예시
ng?inx
```

### {n}

> n회

`{n}`자리에 오는 패턴 요소가 n회 있어야 함

### {n,}

> n회 이상

`{n,}`자리에 오는 패턴 요소가 n회 이상 있어야 함

### {n1, n2}

> n1회에서 n2회까지

`{n1, n2}`자리에 오는 패턴 요소가 n1회 이상, n2회 이하 있어야 함

<br>

## 4. 지시어(directive)

### return

> 문자나 URL을 지정하여 HTTP 상태 코드에 따라 요청한 처리를 반환하기 위해 사용

```nginx
# 예시
location /swagger-ui {
    return 301 http://localhost:8080/swagger-ui.html;
}
```

<br>

### redirect

>  정규 표현식 및 패턴 매칭으로 요청받은 URL을 redirect로 지정한 주소로 바꿔서 요청함

특정 uri로 요청하더라도 해당 uri를 유지하지 않고 redirect로 맵핑되어있는 주소로 요청함

<br>

### rewrite

> 정규 표현식 및 패턴 매칭으로 요청받은 URL을 조작하고 경로를 재지정함

redirect와는 달리 내부적으로 요청을 처리하고 uri는 기존에 호출했던 uri를 유지함

`rewrite [정규식] [rewrite할 내용]` 과 같이 사용

```nginx
location /api {
    rewrite ^/api(.*)$ $1?$args break;
    error_log /var/log/nginx/rewrite_debug.log debug;
    proxy_pass   http://localhost:8080;
    ...
}
```

`$args`를 사용하면 query string으로 보낸 argument에 대해서도 접근이 가능하다.

`[패턴]$`부분이 `$1`에 해당하는 부분이 된다. 이 때, `$`은 정규식 매칭에서 특정 문자열로 끝나는 의미가 아니라 capturing group의 의미로 사용된다.

즉, 위 예시에서 `/api/post?id=1`이라는 요청이 들어왔을 때, 내부적으로 `/post?id=1`로 rewrite되어 8080서버에 전달된다. 

<br>