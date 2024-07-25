
이 내용은 Spring AOP를 이용하여 API 요청정보를 DB에 저장하는 기능을 구현할 때 알게된 정보이다.

당시 구현하고자 했던 기능은 어노테이션을 이용한 Advisor 클래스를 등록한 이후 API Controller 단에 해당 어노테이션을 적용하여 Request 의 Body 그리고 결과값으로 return 되는 값을 DB History 테이블에 저장하는 기능이었다.

해당 기능을 Request Body를 DB에 저장하기 위해 Advisor 클래스 @Before에서 읽어야 했고 구현하였고, Controller 에서 또한 요청 정보를 확인하기 위해 Request Body를 읽어야 했다. 하지만 코드를 작성한 후 테스트한 결과 Body를 읽을 수 없다는 Exception을 만나게 되었다.

구글링을 한 결과 애초에 Request Body 는 한 번만 읽어진다는 사실을 확인하였다. 해당 부분을 해결하기 위해선 HttpRequest 객체 자체를 Wrapper 객체로 감싸서 HttpRequest 객체 자체를 새로 등록하는 형태로 해결하는 방법이 있다는 내용들이었다. Wrapper 객체에 읽었던 Body 내용을 캐싱 할 수 있게 저장해서 사용하는 방법이었다.

해당 방법으로 해결하고자 하였으나 이사님과 상의한 결과 Request 객체 자체를 새로 등록하는 방법은 시스템 전체에 변경을 주는 방법이다 보니 위험하다는 의견을 주셨다. 고민 끝에 결국 내가 택한 방법은 Request 자체에 Body 읽은 값을 다시 setParameter 하여 저장하는 방법이었고, 문제 없이 해당 기능 구현을 완료할 수 있었다.

