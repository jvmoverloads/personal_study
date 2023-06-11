
## 2. String배열
* [2.1 String배열의 선언과 생성](#2.1-string배열의-선언과-생성)
* [2.2 String클래스의 주요 메소드](#2.3-char배열과-string클래스)

### 2.1 String배열의 선언과 생성
```java
String[] name = new String[3]; // 참조형 변수의 기본값은 null이므로 각 요소의 값은 null로 초기화된다.
```

| 자료형   | 기본값 |
|-------|---|
| boolean | false |
| char  | '\u0000' |
| byte, short, int | 0 |
| long  | 0L |
| double | 0.0d 또는 0.0 |
 | 참조형 변수| null |

### 2.2 String클래스의 주요 메소드
| 메소드                                | 설명                                 |
|------------------------------------|------------------------------------|
| char charAt(int index)             | 문자열에서 해당 위치(index)에 있는 문자를 반환한다.   |
| String substring(int from, int to) | 문자열에서 해당범위(from~to)에 있는 문자열을 반환한다) |
| char[] toCharArray()               | 문자열을 문자배열(char[])로 변환해서 반환한다.      |

## 2. 다차원배열
### 2차원배열의 초기화
```java
int[][] arr = new int[][] {{1,2,3}, {4,5,6}};
int[][] arr = {{1,2,3}, {4,5,6}};   // new int[][]생략
```
|     | 국어  | 영어  | 수학  |
|-----|-----|-----|-----|
| 1   | 100 | 100 | 100 |
| 2   | 20  | 20  | 20  |
| 3   | 30  | 30  | 30  |
| 4   | 40  | 40  | 40  |
| 5   | 50  | 50  | 50  |
```java
int [][] score = {
          {100, 100, 100}
        , {20, 20, 20}
        , {30, 30, 30}
        , {40, 40, 40}
        , {50, 50, 50}
        }

```
2차원 배열은 '배열의 배열'. 즉, 여러개의 1차원 배열을 묶어서 하나의 배열로 만든 것.
여기서 score.length의 값은 5. score[0].length는 배열 참조변수 score[0]이 참조하고 있는 배열의 길이이므로 3.

### 가변배열
2차원 이상의 배열을 '배열의 배열'의 형태로 처리한다는 사실을 이용하여    
보다 자유로운 형태의 배열을 구성할 수 있음.

2차원 이상의 다차원 배열을 생성할 때 전체 배열 차수 중 마지막 차수의 길이를 지정하지 않고,
추후에 각기 다른 길이의 배열을 생성함으로써 고정된 형태가 아닌 보다 유동적인 가변 배열을 구성할 수 있다.
만일 다음과 같이 '5 X 3'길이의 2차원 배열 score를 생성하는 코드가 있을 때,
```java
int[][] score = new int[5][3];  // 5행 3열의 2차원 배열 생성
```
위 코드를 다음과 같이 표현할 수 있다.
```java
int[][] score = new int[5][];   // 두번째 차원의 길이는 지정하지 않는다.
score[0] = new int[3];
score[1] = new int[3];
score[2] = new int[3];
score[3] = new int[3];
score[4] = new int[3];
```
첫번째코드와 같이 2차원배열을 생성하면 직사각형 테이블 형태의 고정적인 배열만 생성할 수 있지만,  
두번째 코드와 같이 2차원 배열을 생성하면 다음과 같이 각 행마다 다른 길이의 배열을 생성하는 것이 가능하다.
```java
int[][] score = new int[5][];
score[0] = new int[4];
score[1] = new int[3];
score[2] = new int[2];
score[3] = new int[2];
score[4] = new int[3];
```
score.length의 값은 여전히 5이지만, 일반적인 2차원 배열과 달리 score[0].length의 값은 4이고
score[1].length의 값은 3으로 서로 다르다.
```java
//좌표찍기
public class Test {
//2차원 배열 char배열 board는 입력한 좌표를 표시하기 위한 것이고, 2차원 byte배열 shipBoard에는 상대방의 배의 위치를 저장한다.
//0은 바다이고, 1은 배가 있는 것이다.
    public static void main(String[] args) {
        final int SIZE = 10;
        int x = 0, y = 0;

        char[][] board = new char[SIZE][SIZE];
        byte[][] shipBoard = {
//                1  2  3  4  5  6  7  8  9
                 {0, 0, 0, 0, 0, 0, 1, 0, 0},   // 1
                 {1, 1, 1, 1, 0, 0, 1, 0, 0},   // 2
                 {0, 0, 0, 0, 0, 0, 1, 0, 0},   // 3
                 {0, 0, 0, 0, 0, 0, 1, 0, 0},   // 4
                 {0, 0, 0, 0, 0, 0, 0, 0, 0},   // 5
                 {1, 1, 0, 1, 0, 0, 0, 0, 0},   // 6
                 {0, 0, 0, 1, 0, 0, 0, 0, 0},   // 7
                 {0, 0, 0, 1, 0, 0, 0, 0, 0},   // 8
                 {0, 0, 0, 1, 0, 1, 1, 1, 0},   // 9
        };
        // 1행에 행번호를, 1열에 열번호를 저장한다.
        for(int i=1; i < SIZE; i++) {
            // board는 char배열이므로, 숫자를 문자로 변환하여 저장해야 한다.
            // 그래서 변수 i에 문자 0을 더한다. 
            // 숫자 1에 문자'0'을 더하면 문자 '1'이 된다.
            board[0][i] = board[i][0] = (char) (i+'0');
        }
        
        Scanner scanner = new Scanner(System.in);
        while(true) {
            System.out.print("좌표를 입력하세요. (종료는 00)>");
            String input = scanner.nextLine();

            if(input.length() == 2) {
                x = input.charAt(0) - '0';
                y = input.charAt(1) - '0';

                if(x == 0 && y == 0)
                    break;
            }

            if(input.length() != 2 || x <= 0 || x >= SIZE || y <= 0 || y >= SIZE) {
                System.out.println("잘못된 입력입니다. 다시 입력해주세요");
                continue;
            }

            // shipBoard[x-1][y-1]의 값이 1이면, 'o'을 board[x][y]에 저장한다.
            board[x][y] = shipBoard[x-1][y-1] == 1 ? 'O' : 'X';

            // 배열 board의 내용을 화면에 출력한다.
            for(int i=0; i<SIZE; i++) {
                System.out.println(board[i]);   // board[i]는 1차원 배열
            }
            System.out.println();
        }
    }
}
```
