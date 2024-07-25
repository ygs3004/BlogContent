
프로젝트 진행중에 NAS서버와 FTP 로 연결하여 이미지 파일을 저장, 보여주는 화면을 개발했을때 생겼던 문제이다.

FTP 서버 연결을 위해 구글 검색 후 Apache Commons Net( [FTPClient (Apache Commons Net 3.10.0 API)](https://commons.apache.org/proper/commons-net/apidocs/org/apache/commons/net/ftp/FTPClient.html) 라이브러리를 이용하였다.

해당 문서에 들어가면 다음과 같은 기본 Example 코드를 제공해준다.

```
    FTPClient ftp = new FTPClient();
    FTPClientConfig config = new FTPClientConfig();
    config.setXXX(YYY); // change required options
    // for example config.setServerTimeZoneId("Pacific/Pitcairn")
    ftp.configure(config );
    boolean error = false;
    try {
      int reply;
      String server = "ftp.example.com";
      ftp.connect(server);
      System.out.println("Connected to " + server + ".");
      System.out.print(ftp.getReplyString());

      // After connection attempt, you should check the reply code to verify
      // success.
      reply = ftp.getReplyCode();

      if (!FTPReply.isPositiveCompletion(reply)) {
        ftp.disconnect();
        System.err.println("FTP server refused connection.");
        System.exit(1);
      }
      ... // transfer files
      ftp.logout(); 
    } catch (IOException e) {
      error = true;
      e.printStackTrace();
    } finally {
      if (ftp.isConnected()) {
        try {
          ftp.disconnect();
        } catch (IOException ioe) {
          // do nothing
        }
      }
      System.exit(error ? 1 : 0);
    }
```

해당 코드와 구글에 다른 개발자들이 남긴 흔적들을 템플릿-메소드 패턴으로 개발을 하였다.

파일을 가져오는 메서드를 반복하여 여러 파일을 불러오려고 시도하였으나 disconnect를 하지 않은상황임에도 FTPConnectionClosedException 가 발생하였기에 한 번에 하나의 파일정보만 가져올수 있는것으로 판단하였다.

따라서 여러 이미지를 불러오기 위해 이미지 태그를 사용하여 이미지별 Get 요청을 받아 이미지들을 Response 하다보니 요청하는 형태로 하였다.

코드를 완성한 후 테스트를 하다보니 한 번의 FTP 요청에 1~2초 정도 걸리는 상황을 발견하였다.

몇몇 화면에서는 이미지를 다수 보여줘야하는 상황이었으며, 이미지가 전부 로딩되는데 상당한 시간이 소모되었다. 

개발 후 테스트할 초기에는 단건씩 연결해야하는 상황과 FTP 서버 자체의 속도 문제인가라는 생각에 어쩔수 없다라고 생각했다. 하지만 뭔가 해결 방법을 찾고 싶어 확인하는 중에 신기한 현상(?)을 발견하였다. 난 정확히 얼마나 걸리나 **파일을 가져오는 메서드 부분**에 시간 로그를 찍어보았는데 **빠르게** 되는것이었다. 

어...째서? 그렇다면 왜 어디서 느리지 ? 라는 궁금증과 함께 **코드 사이사이 시간로그를 모두 찍어보며 테스트**한 결과 위 Example 코드에서 **logout 메서드를 실행하는 곳**에서 상당한 지연시간이 있음을 확인하였다.

```
      ... // transfer files
      
      
      ftp.logout();  <--- 바로 이곳!
      
      
    } catch (IOException e) {
```

아니 어째서 로그아웃이...

이후 해당 로그아웃 함수를 주석처리 후 테스트를 실행한 결과 상당한 속도의 향상이 있었다. 

로그아웃을 하지 않으면 문제가 없나 테스트하기 위해 수십장의 이미지를 불러오는 화면을 만들고 새로고침을 통하여 반복 테스트 해본결과 별다른 문제는 발생하지 않았다. 아마도 finally 코드 부분에서 결국 disconnect 하기때문에 문제가 발생하지 않는것으로 예상된다. Document 에서도 해당 메서드(logout)에 대해 단순히 QUIT command를 전송한다고만 되어있으며 IOException이 발생할수 있다 정도로만 설명이 적혀있었다.

```
public boolean logout() throws IOException

Logout of the FTP server by sending the QUIT command.
Returns:
True if successfully completed, false if not.
Throws:
FTPConnectionClosedException - If the FTP server prematurely closes the connection as a result of the client being idle or some other reason causing the server to send FTP reply code 421. This exception may be caught either as an IOException or independently as itself.
IOException - If an I/O error occurs while either sending a command to the server or receiving a reply from the server.
```

코드 라인라인마다 시간로그를 찍어보는 무식한(?) 방법이지만 속도향상을 할 수 있어서 만족했다.

혹시 FTPClient 를 사용하다 느려져서 검색하다 오신분을 위한 

**요약: logout() 를 주석처리해보세요...**