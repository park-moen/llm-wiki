# MVC와 템플릿 엔진

> Source: Inflearn 김영한 「스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술」 강의 스크립트
> Collected: 2026-08-12
> Published: Unknown

네 이번 시간에는 MVC와 템플릿 엔진에 대해서 알아보겠습니다. 앞서서 이미 MVC와 템플릿 엔진 쪽을 살짝 맛을 봤죠. 이번에는 조금 더 내용을 몇 가지 더하고 깊이 설명을 좀 더 드릴게요.

이 MVC라는 건 모델, 뷰, 컨트롤러라는 겁니다. 과거에는 컨트롤러랑 뷰라는 게 따로 분리되어 있지 않았어요. 컨트롤러랑 뷰로 되어 있지 않고 뷰에 모든 걸 다 했어요. JSP를 가지고 예전에는 그렇게 개발을 많이 했는데, 그거를 소위 Model 1 방식이라고 하고요. 지금은 MVC style로 많이 합니다.

왜냐하면 프로그램을 개발할 때 관심사를 분리해야 된다는 말을 많이 들어보셨을 거예요. 역할과 책임이라는 이야기도 많이 들어보셨을 거예요. View는 화면을 그리는 데 모든 역량을 집중해야 돼요. Controller나 Model과 관련된 부분들은 business logic과 관련이 있거나 내부적인 것을 처리하는 데 집중해야 돼요. 그래서 Model, View, Controller를 이렇게 쪼갠 거예요.

과거에 제가 처음 취업했던 14년 전 이야기인가 15년 전 이야기인가, 회사에서 내부 운영하는 프로젝트가 있었는데 유지보수할 때 항상 한 분을 불러왔어요. 옆에서 source를 봤는데 JSP file 하나가 수천 줄이 넘어갔던 것 같아요. View 안에서 DB도 접근하고 Controller logic도 다 있고 business logic도 화면에 다 있었던 거예요. 모든 것이 View file 하나 안에 있었고, 개발하셨던 분도 한참 보면서 운영을 하시더라고요.

요즘에는 Controller와 View를 쪼개는 게 기본이에요. View는 화면과 관련된 일만 하고, business logic과 server 뒷단에 관련된 것은 Controller나 business logic에서 처리합니다. Model이라는 데 화면에서 필요한 것을 담아 화면 쪽에 넘겨주는 pattern을 많이 사용합니다.

이번에는 조금 더 내용 있는 Controller를 하나 만들어 볼게요. `helloController`에 `@GetMapping`으로 `helloMvc`를 만들고 외부에서 parameter를 받을 거예요. Web에서 이때는 `@RequestParam`이라고 적고 name이라고 해볼게요. 이전에는 Spring이라고 직접 받았지만 이번에는 이름을 URL parameter로 바꿔볼게요.

Model도 넘겨줘야 됩니다. Model에 담으면 View에서 render할 때 쓰는 거죠. `model.addAttribute`에서 parameter로 넘어온 name을 넘겨볼게요. key가 `name`이고 value도 name입니다. 그리고 `return "hello-template"`이라고 해볼게요.

그러면 `hello-template`으로 간다고 했죠. `templates`에서 `hello-template.html`을 만들고 강의안의 내용을 복사 붙이기 할게요. 이제 Thymeleaf template engine을 써야 돼요. HTML을 바꾸는 겁니다.

Thymeleaf template의 장점은 HTML을 그대로 쓰고 그 file을 server 없이 바로 열어봐도 껍데기를 볼 수 있다는 점이에요. 여기 보면 `hello! empty`라고 나오죠. Template engine으로 동작하면 여기 있는 값으로 내용이 치환됩니다.

내용을 넣어놓는 것은 server 없이 HTML을 만들어서 볼 때 markup하시는 분들이 무언가 적어 놓고 볼 수 있게 하기 위한 것이고요. 실제 server를 타서 돌면 여기 있는 값이 Model의 값으로 바뀌게 됩니다.

이렇게 해 놓고 `localhost:8080/hello-mvc`로 실행하면 처음에 error가 나요. Error가 나면 우선 log를 봐야 됩니다. 원인이 떴죠. `Required String parameter 'name' is not present`라고 name parameter가 없다고 합니다.

여기에서 option을 볼 수 있어요. 이 `Command + P` option이 되게 좋거든요. Windows는 강의의 참고 내용을 확인해 주세요. Parameter information은 `Command + P`입니다. 굉장히 많이 쓰죠.

여기에 `required`라는 option이 있습니다. `required`의 default가 `true`예요. 한마디로 무조건 넣어야 되는 거예요. `false`로 하면 넘기지 않아도 돼요. `required`가 기본으로 `true`이기 때문에 기본으로 값을 넘겨야 됩니다.

물음표 `?name=`은 HTTP GET 방식에서 parameter를 넘기는 방법입니다. `Spring!`을 넣어보면 `Hello Spring!`이 나옵니다.

이 동작 방식을 설명해 드리겠습니다. `name`은 `spring`으로 넘어오고 Controller의 name도 spring으로 바뀝니다. 그리고 Model에 담깁니다. Template으로 넘어가면 `${name}`은 Model에서 key가 name인 값을 꺼내서 치환합니다. 그래서 `hello spring`으로 나가는 거겠죠.

그림으로 설명하면 이전에 봤던 그림과 parameter가 추가된 것 빼고 똑같습니다. Web browser에서 `localhost:8080/hello-mvc`로 요청하면 Spring Boot가 띄울 때 같이 띄우는 내장 Tomcat server를 먼저 거칩니다. 내장 Tomcat server는 `hello-mvc` 요청을 Spring에 전달합니다.

Spring은 `helloController`의 method에 mapping되어 있는 것을 찾아 호출합니다. Method가 return할 때 이름은 `hello-template`이고 Model에는 key가 name, value가 spring으로 담깁니다.

Spring의 View Resolver가 동작합니다. View Resolver는 View를 찾아주고 template engine을 연결해 줍니다. View Resolver가 `templates`에서 return string name과 같은 `hello-template`을 찾아 Thymeleaf template engine에 처리를 넘깁니다. Template engine이 rendering해서 변환한 HTML을 Web browser에 반환합니다.

정적 콘텐츠일 때는 변환하지 않고 그대로 반환했지만 template engine에서는 변환해서 Web browser에 넘겨줍니다. Source를 보면 이 부분이 변환되어 넘어간 것을 볼 수 있습니다.

이렇게 MVC와 템플릿 엔진에 대한 가장 기초적이고 기본적인 내용을 알아보았습니다. 다음 시간에는 API에 대해서 알아보겠습니다. 감사합니다.
