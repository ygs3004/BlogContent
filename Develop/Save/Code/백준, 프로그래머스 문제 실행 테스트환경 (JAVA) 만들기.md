
오늘 포스트 할 코드는 이전에 올렸던 포스트의 코드의 리팩토링과 함께 프로그래머스의 케이스를 추가한 코드이다. 이전 포스트 내용은 아래와 같다.

[백준 문제 실행 테스트 환경(JAVA) 만들기 :: 꿀잠 (tistory.com)](https://ygs3004.tistory.com/14)

---

코딩테스트의 대표 사이트인 백준과 프로그래머스의 문제에는 형식에 차이가 있다. 
백준의 경우 테스트로 입력되는 파라미터가 System.in으로 입력되고, 정답의 경우 출력을 통하여 문제를 푼다면.
프로그래머스의 경우에는 solution 함수를 만들고, 해당 solution 함수의 파라미터로 테스트 케이스가 입력되고, 정답의 경우 return 값으로 처리 된다.

첫 줄에 남겨둔 링크인 이전 코드의 경우 백준을 고려하여 만들다보니 프로그래머스의 문제풀이 테스트 결과를 확인할 수 없는 문제가 있었다. 프로그래머스의 테스트 케이스를 문자열로 하여 적절하게 타입 변환 해주는 코드들을 작성할 수도 있었겠지만 Jackson 라이브러리 등을 붙이는 것이 아니면 과한 작업 소요라고 생각하였고, HashMap을 통하여 테스트 케이스 및 결과 케이스를 저장하는 형태로 테스트 환경을 구축하였다.
또한 실제 사이트에 제출 전 IDE에서 작성한 부분을 수정하는 작업을 최대한 적게 줄일 수 있도록 코드를 작성하였다.


백준과 프로그래머스의 두 케이스로 나누기 위해 전략패턴을 사용하고자 하였고, 우선 한 일은 문제에 대한 Interface 화이다.

### class Problem

```java
import java.awt.*;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.stream.IntStream;

public interface Problem<P, R>{

    Problem<P, R> setAnswer(Object answer);
    HashMap<String, P> getInputCase();
    HashMap<String, R> getResultCase();
    R solve(P parameter) throws Exception;

    default void test() {

        try {
            // Result 저장
            Map<String, R> resultCase = getResultCase();
            boolean isSuccess = true;

            Map<String, P> inputCase = getInputCase();
            for (String caseKey : inputCase.keySet()) {
                P input = inputCase.get(caseKey);

                System.out.println("테스트 실행 Input");
                println(input);

                long startTime = System.nanoTime();
                R testResult = solve(input);
                long endTime = System.nanoTime();

                println("실행시간: " + (endTime-startTime)/1_000_000.00 +"ms");
                System.out.println();

                R expectResult = resultCase.get(caseKey);
                boolean isCaseSuccess = expectResult.equals(testResult);
                isSuccess = isSuccess && isCaseSuccess;

                println("정답");
                println(expectResult);
                System.out.println();

                println("결과");
                println(testResult);
                System.out.println();

                if(isCaseSuccess){
                    passPrintln("Case 통과");
                }else{
                    failPrintln("Case 실패");
                }

                System.out.println();
                println("=======================================================================================");
            }

            if(isSuccess){
                passPrintln("모든 테스트가 통과하였습니다.");
                System.out.println();
            }else{
                failPrintln("************** 실패한 테스트가 있습니다. 결과를 확인해주세요 **************");
                // 실패시 소리
                Toolkit.getDefaultToolkit().beep();
                System.out.println();
            }

        } catch (Exception e){
            e.printStackTrace();
        }
    };

    private void passPrintln(String str){
        // console 색상
        String reset = "\u001B[0m";
        String green = "\u001B[32m";
        println(green + str + reset);
    }

    private void failPrintln(String str){
        // console 색상
        String red = "\u001B[31m";
        String reset = "\u001B[0m";
        println(red + str + reset);
    }

    default void println(Object input){
        if(input instanceof File) {
            try{
                FileInputStream inputFileStream = new FileInputStream((File)input);
                BufferedReader br = new BufferedReader(new InputStreamReader(inputFileStream));
                StringBuilder fileString = new StringBuilder();
                String line = "";
                while ((line = br.readLine()) != null) {
                    fileString.append(line).append(System.lineSeparator());
                }
                System.out.print(fileString);
            }catch (Exception e){
                e.printStackTrace();
                throw new RuntimeException(e);
            }
        }else if (input.getClass().isArray()) {
            printArray(input);
        } else {
            System.out.print(input);
        }

        System.out.println();
    }

    default void printArray(Object input){
        int length = Array.getLength(input);
        Object[] array = new Object[length];
        IntStream.range(0, length).forEach(i ->
                array[i] = Array.get(input, i)
        );

        System.out.print("[");
        for(int i = 0; i < array.length; i++){
            Object value = array[i];
            if(value.getClass().isArray()){
                printArray(value);
            }else{
                System.out.print(value);
            }
            if(i != array.length - 1) System.out.print(", ");
        }

        System.out.print("]");
    }

}
```

백준과 프로그래머스에서 달라지는 케이스인 InputCase 와 ResultCase 를 구현 체에서 작성하게 하였고 test() 자체는 동일하게 구성하였다. 틀렸을 경우 확인하기 쉽도록 GPT의 도움을 받아 console 창에 색상과, 소리가 나는 코드를 추가하였다. 또한 케이스를 프린트 해볼 경우 Array 안의 다른 Object 가 있는 경우가 있어 print 관련하여서도 메서드를 추가하였다.
프로그래머스의 경우 input 과 return 의 타입이 변경될 수 있어서 제네릭을 이용하였다.

아래는 BaekJoon과 Programmers에서 Interface 메서드들을 구현한 내용들이다.
### class BaekJoon

```java
import java.io.*;
import java.lang.reflect.Constructor;
import java.net.URL;
import java.util.Arrays;
import java.util.HashMap;

public class BaekJoon implements Problem<File, String>{

    Object answer;
    File[] testFiles;

    public Problem<File, String> setAnswer(Object answer) {
        this.answer = answer;
        URL classDir = answer.getClass().getResource("");
        try{
            testFiles = new File(classDir.toURI()).listFiles();
        }catch (Exception e){
            e.printStackTrace();
            throw new RuntimeException(e);
        }

        return this;
    }

    @Override
    public HashMap<String, File> getInputCase() {
        HashMap<String, File> testCase = new HashMap<>();
        Arrays.stream(testFiles)
                .filter((file) -> file.getName().startsWith("input"))
                .forEach(file -> {
                    String fileName = file.getName();
                    String testSeq = fileName.substring(fileName.indexOf("input") + "input".length(), fileName.lastIndexOf("."));
                    testCase.put(testSeq, file);
                });
        return testCase;
    }

    @Override
    public HashMap<String, String> getResultCase() {
        HashMap<String, String> result = new HashMap<>();
        try {
            Arrays.stream(testFiles)
                    .filter((file) -> file.getName().startsWith("result"))
                    .forEach(file -> {
                        String fileName = file.getName();
                        String testSeq = fileName.substring(fileName.indexOf("result") + "result".length(), fileName.lastIndexOf("."));
                        StringBuilder resultSb = new StringBuilder();
                        try {
                            BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(file)));
                            String line = "";
                            while ((line = br.readLine()) != null) {
                                resultSb.append(line).append(System.lineSeparator());
                            }

                            result.put(testSeq, resultSb.toString().trim());
                        } catch (IOException e) {
                            throw new RuntimeException(e);
                        }
                    });
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return result;
    }

    @Override
    public String solve(File file) throws Exception {
        InputStream parameter = new FileInputStream(file);
        System.setIn(parameter);
        ByteArrayOutputStream resultOutputStream = new ByteArrayOutputStream();
        PrintStream resultSave = new PrintStream(resultOutputStream);
        PrintStream resultConsole = System.out;

        System.setOut(resultSave);

        // Solution class 변수 초기화를 위해 solve 마다 새로운 instance 생성
        Constructor constructor = answer.getClass().getConstructor();
        Object instance = constructor.newInstance();
        instance.getClass()
                .getDeclaredMethod("main", String[].class)
                .invoke(instance, (Object) null);
        System.out.flush();

        String testResult = resultOutputStream.toString().trim(); // System.out.println() 으로 정답 입력시 개행문자 제거
        resultOutputStream.close();
        parameter.close();
        System.setOut(resultConsole);
        return testResult;
    }

}
```

### class Programmers

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.util.HashMap;

public class Programmers<R> implements Problem<Object[], R>{

    Object answer;

    public Problem<Object[], R> setAnswer(Object answer) {
        this.answer = answer;
        return this;
    }

    @Override
    public HashMap<String, Object[]> getInputCase() {
        HashMap<String, Object[]> inputCase = null;

        try {
            String testClassName = answer.getClass().getPackage().toString().split(" ")[1] + ".TestCase";
            Class<?> inputClass = Class.forName(testClassName);

            Constructor<?> constructor = inputClass.getConstructor();
            Object instance = constructor.newInstance();

            inputCase = (HashMap<String, Object[]>) inputClass
                    .getMethod("getInput")
                    .invoke(instance);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return inputCase;
    }

    @Override
    public HashMap<String, R> getResultCase() {

        HashMap<String, R> resultCase = null;

        try {
            String testClassName = answer.getClass().getPackage().toString().split(" ")[1] + ".TestCase";
            Class<?> testClass = Class.forName(testClassName);

            Constructor<?> constructor = testClass.getConstructor();
            Object instance = constructor.newInstance();

            resultCase = (HashMap<String, R>) testClass
                    .getMethod("getResult")
                    .invoke(instance);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        return resultCase;
    }

    @Override
    public R solve(Object[] parameter) throws Exception {
        Method[] methods = answer.getClass().getDeclaredMethods();

        // Solution class 변수 초기화를 위해 solve 마다 새로운 instance 생성
        Constructor constructor = answer.getClass().getConstructor();
        Object instance = constructor.newInstance();

        R result = null;
        for(Method method : methods){
            if(method.getName().equals("solution")){
                result = (R) method.invoke(instance, parameter);
            }
        }
        return result;
    }

}
```

input 되는 파라미터가 여러 개인 경우도 있기 때문에 Object 배열을 사용하였다.

아래는 테스트 실행 방법 예시이다.

### class TestMain

```java 
import programmers.LV2_12899.Solution;

public class TestMain {

    public static void main(String[] args) throws Exception {

        BaekJoon baekJoon = new BaekJoon();
        Programmers<String> programmersReturnString = new Programmers<>();
        Programmers<Integer> programmersReturnInt = new Programmers<>();

        // 소수 찾기 (Level 2)
        // https://school.programmers.co.kr/learn/courses/30/lessons/42839
        programmersReturnInt.setAnswer(new programmers.LV2_42839.Solution()).test();

        // 어린왕자 (Silver 3)
        // https://www.acmicpc.net/problem/1004
        baekJoon.setAnswer(new baekjoon.S3_2606.Main()).test();

        // 바이러스 (Level 3)
        // https://www.acmicpc.net/problem/2606
        baekJoon.setAnswer(new baekjoon.S3_1004.Main()).test();
    }

}
```

### TestCase - 프로그래머스(class로 작성)

```java
package programmers.LV2_42839;

import java.util.HashMap;

public class TestCase {

    public HashMap<String, Object[]> getInput(){
        HashMap<String, Object[]> testCase = new HashMap<>();
        String numbers1 = "17";
        testCase.put("case1", new Object[]{numbers1});

        String numbers2 = "011";
        testCase.put("case2", new Object[]{numbers2});

        String numbers3 = "143";
        testCase.put("case3", new Object[]{numbers3});
        return testCase;
    }

    public HashMap<String, Object> getResult(){
        HashMap<String, Object> resultCase = new HashMap<>();
        resultCase.put("case1", 3);
        resultCase.put("case2", 2);
        resultCase.put("case3", 6);
        return resultCase;
    }

}
```

### TestCase - 백준
input: (input + 케이스 숫자.txt 파일에 케이스 복사 및 붙여넣기)
result: (result + 케이스 숫자.txt 파일에 케이스 복사 및 붙여넣기)

### input1.txt
```
2
-5 1 12 1
7
1 1 8
-3 -1 1
2 2 2
5 5 1
-4 5 1
12 1 1
12 1 2
-5 1 5 1
1
0 0 2
```

### result1.txt
```
3
0
```


정답 class의 경우 실제 제출할 때와 똑같이 작성하면 된다. 단 IDE에서 작성할 경우 현재 패키지가 import 된 구문이 있으므로, 문제를 해결한 이 후 패키지 명만 제거한 후 제출하면 된다. 

[Algorithm-GroupStudy](https://github.com/ygs3004/Algorithm-GroupStudy/tree/fork/%EC%9C%A4%EA%B1%B4%EC%88%98)
에서 실제로 본인이 사용한 코드를 확인 가능하다.