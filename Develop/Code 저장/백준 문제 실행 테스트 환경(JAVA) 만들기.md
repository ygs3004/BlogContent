
코딩테스트, 알고리즘을 위한 사이트들이 있는데 그 중 내가 이용하는 것은 프로그래머스와 백준 사이트이다.

프로그래머스의 경우 자체적으로 코드 실행을 할 수도 있고, 코드 실행중에 System.out.println 을 이용하여 디버깅도 나름 가능하다. 

하지만 백준의 경우 디버깅을 하기가 힘들다. 또한 문제를 풀다가 왜 틀렸는지 알 수 없어 질문하기 쪽을 보다 보니 input 데이터에 공백이 들어가 있는 것 같다는 대답을 들은 적도 있다. 실제로 공백을 처리하는 로직을 추가하였더니 통과된 경험도 있다.

그래서 공백까지 들어가있는 테스트 데이터를 만들거나, InputStream 으로 input 데이터가 들어오는 문제를 실행 및 테스트를 할 수 있는 코드를 만들었고 이를 공유하고자 이 글을 포스트한다. 해당 코드는 아래와 같다.

``` java
import java.io.*;  
import java.lang.reflect.InvocationTargetException;  
import java.net.URL;  
import java.util.Arrays;  
import java.util.HashMap;  
import java.util.Map;  
import java.util.concurrent.atomic.AtomicBoolean;  
  
public class Main {  
  
    public static void main(String[] args) throws Exception {  
        test(new Form());  
    }  
  
    private static void test(Object problem) throws Exception {  
        URL classDir = problem.getClass().getResource("");  
        File[] files = new File(classDir.toURI()).listFiles();  
  
        // Result 저장  
        Map<String, String> result = new HashMap();  
        Arrays.stream(files).filter((file) -> file.getName().startsWith("result")).forEach(file -> {  
            String fileName = file.getName();  
            String testSeq = fileName.substring(fileName.indexOf("result") + "result".length(), fileName.lastIndexOf("."));  
            StringBuilder resultSb = new StringBuilder();  
            try {  
                BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(file)));  
                String line = "";  
                while((line = br.readLine()) != null){  
                    resultSb.append(line).append(System.lineSeparator());  
                }  
  
                result.put(testSeq, resultSb.toString().trim());  
            } catch (FileNotFoundException e) {  
                throw new RuntimeException(e);  
            } catch (IOException e) {  
                throw new RuntimeException(e);  
            }  
        });  
  
        AtomicBoolean isSuccess = new AtomicBoolean();  
        isSuccess.set(true);  
  
        // Input 실행  
        Arrays.stream(files).forEach(file -> {  
            String fileName = file.getName();  
            boolean isInput = fileName.startsWith("input");  
            if(isInput){  
                String testSeq = fileName.substring(fileName.indexOf("input") + "input".length(), fileName.lastIndexOf("."));  
                System.out.println("** " + testSeq + "번 테스트 실행 **");  
                try {  
                    long startTime = System.nanoTime();  
                    String testResult  
                            = (String) problem.getClass()  
                            .getDeclaredMethod("solution", InputStream.class)  
                            .invoke(problem, new FileInputStream(file));  
                    long endTime = System.nanoTime();  
                    System.out.println("실행시간: " + (endTime-startTime)/1_000_000.00 +"ms");  
                    System.out.println();  
  
                    String expectResult = result.get(testSeq);  
                    System.out.println("정답");  
                    System.out.println(expectResult);  
                    System.out.println();  
  
                    System.out.println("결과");  
                    System.out.println(testResult);  
                    System.out.println();  
  
                    if(testResult.equals(expectResult)){  
                        System.out.println(testSeq + "번 테스트 성공");  
                        isSuccess.set(isSuccess.get());  
                    }else{  
                        System.out.println(testSeq + "번 테스트 실패");  
                        isSuccess.set(false);  
                    }  
  
                } catch (IllegalAccessException | FileNotFoundException e) {  
                    throw new RuntimeException(e);  
                } catch (NoSuchMethodException e) {  
                    System.out.println("solution 메서드가 없습니다");  
                    throw new RuntimeException(e);  
                } catch (InvocationTargetException e) {  
                    e.printStackTrace();  
                    throw new RuntimeException(e);  
                }  
                System.out.println("=======================================================================================");  
            }  
        });  
  
        if(isSuccess.get()){  
            System.out.println("모든 테스트가 통과하였습니다.");  
        }else{  
            System.out.println("************** 실패한 테스트가 있습니다. 결과를 확인해주세요 **************");  
        }  
    }  
  
}



```

해당 코드를 활용하기 위해서는 우선 문제 풀이를 작성할 Form 클래스(클래스 이름은 문제마다 또는 원하는 형태)를 하나 만든다.

``` java
import java.io.BufferedReader;  
import java.io.InputStream;  
import java.io.InputStreamReader;  
  
public class Form {  
  
    public String solution(InputStream systemIn) throws Exception{  
        BufferedReader br = new BufferedReader(new InputStreamReader(systemIn));  
        StringBuilder result = new StringBuilder();  

		// 문제풀이코드
  
        System.out.println(result);  
        return result.toString();  
    }  
  
}
```

문제 풀이 코드는 위와 같은 형태로 작성한다. 테스트 데이터는 input1.txt, result1.txt 형태로(input/result + 케이스 번호) 텍스트 파일을 만들어 문제 풀이 클래스와 동일한 디렉토리에 저장한다. 아래와 같은 형태면 된다.

프로젝트 루트
├── package1
│   ├── 문제1 Class
│   └── input1.txt
│   └── input2.txt
│   └── input3.txt
│   └── result1.txt
│   └── result2.txt
│   └── result33.txt
├── package2
│   ├── 문제2 Class
│   └── input1.txt
│   └── result1.txt
└── Main

``` java
        if(isSuccess.get()){  
            System.out.println("모든 테스트가 통과하였습니다.");  
        }else{  
            System.out.println("************** 실패한 테스트가 있습니다. 결과를 확인해주세요 **************");  
        }  

```

테스트 코드의 성공 여부에 따라 위와같은 멘트가 콘솔창에 출력된다.
폴더 구조 및 출력멘트는 Main 클래스에서 입맛에 맞게 변경하면 될 것이다.

백준에 제출하기 전에 내 PC 에서 실행하던 클래스를 적절히 변경 후 제출하면 된다.


``` java
// 실행환경 패키지 이름 제거

import java.io.BufferedReader;  
import java.io.InputStream;  
import java.io.InputStreamReader;  
  
// public class Form {  임의의 클래스명 -> Main
public class Main {  

	// public String solution(InputStream systemIn) throws Exception{ 
    public static void main(String[] args) throws Exception{  
    
        // systemIn => System.in
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));  
        StringBuilder result = new StringBuilder();  

		// 문제풀이코드
  
        System.out.println(result);  
        // return result.toString();  
    }  
  
}
```

1. package 이름 제거
2. 클래스 이름 Main 으로 변경,
3. main 메서드 이름 변경
4. return 제거 후 정답 코드 출력(System.out.println 이 아니라 BufferWriter 등을 사용해도 된다.)

이후 런타임 오류가 없는데 문제가 틀린다면 문제를 다시 풀어보도록 하면된다.
열공하는 사람 모두 화이팅!