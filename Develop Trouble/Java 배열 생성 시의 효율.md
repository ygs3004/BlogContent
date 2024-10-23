코딩테스트 문제를 풀 다 시간 초과를 해결했던 경험에 대한 기록이다. 

백준의 텀프로젝트 문제를 풀던 중(https://www.acmicpc.net/problem/9466)
내가 작성한 코드가 충분히 최적화 되었다고 생각했음에도 계속 시간 초과가 발생하였다.
관련하여 문제를 찾던 중 자바의 배열 생성이 시간 초과의 원인이 될 수 있다는 글을 발견하고, 해당 부분을 수정하여 통과하였다.

```java

    private static int solution() throws IOException {
        int studentNum = Integer.parseInt(br.readLine());
        int[] team = new int[studentNum + 1];
        String[] input = br.readLine().split(" ");
        for(int i = 1; i <= studentNum; i++){
            team[i] = Integer.parseInt(input[i-1]);
        }

        checked = new boolean[studentNum + 1];
        result = studentNum;

        for(int i = 1; i <= studentNum; i++){
            if(checked[i]) continue;
            // 배열 초기화
            visited = new int[studentNum + 1];
            findTeam(team, i, 1, visited);
        }

        return result;
    }

    private static void findTeam(int[] team, int student, int seq, int[] visited){
        if(checked[student]) return;
        checked[student] = true;
        visited[student] = seq;

        int next = team[student];
        if(visited[next] != 0){
            result -= (seq - visited[next] + 1);
        }else{
            findTeam(team, next, seq + 1, visited);
        }
    }

```

시간 초과가 나던 시점의 내 코드는 위와 같았으며,  완전 탐색을 위하여 탐색 방문 배열을 new 명령어로 생성하고 있었다. 해당 배열의 크기는 최대 100001의 크기를 갖는 문제이다. 

```java
    private static int solution() throws IOException {
        int studentNum = Integer.parseInt(br.readLine());
        int[] team = new int[studentNum + 1];
        StringTokenizer st = new StringTokenizer(br.readLine());
        for(int i = 1; i <= studentNum; i++){
            team[i] = Integer.parseInt(st.nextToken());
        }

        checked = new boolean[studentNum + 1];
        int[] visited = new int[studentNum + 1];
        result = studentNum;
        for(int i = 1; i <= studentNum; i++){
            if(checked[i]) continue;
            findTeam(team, i, 0, visited);
        }

        return result;
    }

    private static void findTeam(int[] team, int student, int seq, int[] visited){
        if(checked[student]) return;
        seq++;
        checked[student] = true;
        visited[student] = seq;

        int next = team[student];
        if(visited[next] != 0){
            result -= (seq - visited[next] + 1);
        }else{
            findTeam(team, next, seq, visited);
        }

		// dfs 내부에서 사용후 값 원상복구
        visited[student] = 0;
    }

}
```

시간 초과를 해결한 코드는 위와 같다. dfs를 반복하기 이전에 생성한 배열의 값을 new 가 아닌 직접 초기화 하여 배열을 사용하였다. 참고한 글(https://okky.kr/questions/1450047)에 따르면 배열을 생성한다는 것은 새로운 객체의 메모리에 할당 받는 부분, java의 경우 해당 배열의 초기 값을  초기화하는 부분 등으로 인하여 런타임 실행 시간이 늘어날 수 있다고 한다. 단순한 코드의 차이였지만 객체 생성의 효율에 대해 고민할 수 있었다.