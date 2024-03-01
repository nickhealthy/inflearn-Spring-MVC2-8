# 인프런 강의

해당 저장소의 `README.md`는 인프런 김영한님의 SpringBoot 강의 시리즈를 듣고 Spring 프레임워크의 방대한 기술들을 복기하고자 공부한 내용을 가볍게 정리한 것입니다.

문제가 될 시 삭제하겠습니다.



## 해당 프로젝트에서 배우는 내용

- 섹션 11 | 파일 업로드



# 파일 업로드

HTML Form을 통한 파일 업로드는 다음과 같은 두 가지 방식이 있다.

* `application/x-www-form-urlencoded`
* `multipart/form-data`



### application/x-www-form-urlencoded

* HTML 폼 데이터를 서버로 전송하는 가장 기본적인 방법
* Form 태그에 별도의 `enctype` 옵션이 없으면 웹 브라우저는 요청 HTTP 메시지의 헤더에 다음 내용을 추가한다.
  * `Content-Type: application/x-www-form-urlencoded`
* <u>파일을 업로드 하려면 파일은 문자가 아니라 바이너리 데이터를 전송해야 하기 때문에 해당 방식은 문제가 된다.</u>
* <u>또한 폼을 전송할 때 파일만 전송하지 않고, 부가적인 정보도 같이 보내기 때문에 다른 방식이 필요하다.</u>

![스크린샷 2024-03-01 오후 4 16 49](https://github.com/nickhealthy/inflearn-Spring-MVC2-8/assets/66216102/854ee6b4-e256-458f-b7b1-f1dae283d569)



### multipart/form-data

* 이 방식을 사용하려면 Form 태그에 별도의 `enctype="multipart/form-data"` 를 지정해야 한다.
  * 해당 방식은 다른 종류의 여러 파일과 폼의 내용을 함께 전송할 수 있다.
* 폼의 입력 결과로 생성된 HTTP 메시지를 보면 각각의 전송 항목이 구분이 되어있다. 
  * `Content-Disposition`이라는 항목별 헤더가 추가되어 있고 여기에 부가 정보가 있다.
  * 폼의 일반 데이터는 각 항목별로 문자가 전송되고, 파일의 경우 Content-Type이 추가되며 바이너리 데이터가 전송됨

![스크린샷 2024-03-01 오후 4 16 55](https://github.com/nickhealthy/inflearn-Spring-MVC2-8/assets/66216102/4d39a9ee-26de-4804-ae64-ac783b8db3b7)

## 서블릿과 파일 업로드1

먼저 서블릿을 통한 파일 업로드를 실습해보자



### 예제 - 멀티파트 데이터 받기

[`ServletUploadControllerV1`]

```java
package hello.upload.controller;

import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.Part;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import java.io.IOException;
import java.util.Collection;

@Slf4j
@Controller
@RequestMapping("/servlet/v1")
public class ServletUploadControllerV1 {

    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
    }

    @PostMapping("/upload")
    public String saveFileV1(HttpServletRequest request) throws ServletException, IOException {
        log.info("request = {}", request);


        String itemName = request.getParameter("itemName");
        log.info("itemName = {}", itemName);

        // request.getParts(): multipart/form-data 전송 방식에서 각각 나누어진 부분을 받아서 확인 가능하다.
        Collection<Part> parts = request.getParts();
        log.info("parts = {}", parts);

        return "upload-form";
    }
}
```





[`upload-form.html`]

* form 태그에 `enctype="multipart/form-data"` 옵션을 추가해주어야 한다.

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
</head>
<body>

<div class="container">

    <div class="py-5 text-center">
        <h2>상품 등록 폼</h2>
    </div>

    <h4 class="mb-3">상품 입력</h4>

    <form th:action method="post" enctype="multipart/form-data">
        <ul>
            <li>상품명 <input type="text" name="itemName"></li>
            <li>파일<input type="file" name="file" ></li>
        </ul>
        <input type="submit"/>
    </form>

</div> <!-- /container -->
</body>
</html>
```



#### 실행 결과

![스크린샷 2024-03-01 오후 4 29 50](https://github.com/nickhealthy/inflearn-Spring-MVC2-8/assets/66216102/8c037adb-8606-42ed-8e54-6ccec260b63d)



### 멀티파트 사용 옵션

업로드 사이즈 제한

* `max-file-size` : 파일 하나의 최대 사이즈, 기본 1MB
* `max-request-size` : 멀티파트 요청 하나에 여러 파일을 업로드 할 수 있는데, 그 전체 합이다. 기본 10MB

```properties
spring.servlet.multipart.max-file-size=1MB
spring.servlet.multipart.max-request-size=10MB
```



