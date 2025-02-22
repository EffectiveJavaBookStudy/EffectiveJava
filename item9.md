# Item9. try-finally보다는 try-with-resources를 사용하라

자바에서는 close() 메서드를 호출해야 하는 자원(예: 파일, 네트워크, 데이터베이스 등)이 많습니다.

하지만 놓치기가 쉬워 보통 잘 안닫아서 문제가 생기는데 상당수가 finalizer을 활용하여 닫게 됩니다.

특히, try 블록과 finally 블록에서 모두 예외가 발생하면, finally 블록의 예외가 원래 예외를 덮어버릴 수 있게 됩니다.

즉, 실제 원인(try 블록에서 발생한 예외)이 사라지고, finally 블록에서 발생한 예외만 남게 되는 문제가 발생하죠.

예시

```java
import java.io.*;

public class TryFinallyException {
    public static void main(String[] args) {
        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader("nonexistent.txt"));
            String line = br.readLine();  // ❌ 예외 발생 가능 (파일 없음)
            System.out.println(line);
        } catch (IOException e) {
            System.err.println("예외 발생: " + e.getMessage());
        } finally {
            if (br != null) {
                try {
                    br.close();  // ❌ close()에서도 예외 발생 가능
                } catch (IOException e) {
                    System.err.println("close()에서 예외 발생: " + e.getMessage());
                }
            }
        }
    }
}
```
❌ 문제점
- readLine()에서 예외 발생 가능 (FileNotFoundException 등).
- close()에서도 예외 발생 가능.
- finally 블록에서 발생한 예외가 원래 예외를 덮어버릴 수 있게 되어서 디버깅 어려워집니다.

## ✅ 해결 방법
try-with-resources 사용 ( 자바 7 에서 도입이 되었습니다 )

```java
import java.io.*;

public class TryWithResourcesExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("example.txt"))) {
            String line = br.readLine();  // ✅ 예외 발생 가능하지만 문제없음
            System.out.println(line);
        } catch (IOException e) {
            System.err.println("예외 발생: " + e.getMessage());
        }
    }
}
```

### ✅ 장점
1. 코드가 짧고 가독성이 좋아짐 (finally 블록 불필요).
2. 예외가 발생해도 원래 예외가 유지됨 (finally에서 예외 덮어쓰는 문제 해결).
3. 자동으로 close() 호출 (null 체크 필요 없음).

만약에 그러면 닫아야할 자원이 많아지면 어떻게 되는지 봅시다.

### try-finally (여러 자원 처리 문제)
```java
import java.io.*;

public class TryFinallyMultiple {
    public static void main(String[] args) {
        BufferedReader br = null;
        BufferedWriter bw = null;
        try {
            br = new BufferedReader(new FileReader("input.txt"));
            bw = new BufferedWriter(new FileWriter("output.txt"));
            String line;
            while ((line = br.readLine()) != null) {
                bw.write(line);
                bw.newLine();
            }
        } catch (IOException e) {
            System.err.println("예외 발생: " + e.getMessage());
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    System.err.println("br.close()에서 예외 발생: " + e.getMessage());
                }
            }
            if (bw != null) {
                try {
                    bw.close();
                } catch (IOException e) {
                    System.err.println("bw.close()에서 예외 발생: " + e.getMessage());
                }
            }
        }
    }
}
```

❌ 문제점
1. finally 블록에서 두 개의 자원(br, bw)을 각각 null 체크 후 close() 호출해야 함.
2. close()에서도 예외가 발생할 수 있어 예외 처리가 중첩됨.
3. 가독성이 매우 떨어짐.

### ✅ try-with-resources (여러 자원도 간단하게 해결)
```java
import java.io.*;

public class TryWithResourcesMultiple {
    public static void main(String[] args) {
        try (
            BufferedReader br = new BufferedReader(new FileReader("input.txt"));
            BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"))
        ) {
            String line;
            while ((line = br.readLine()) != null) {
                bw.write(line);
                bw.newLine();
            }
        } catch (IOException e) {
            System.err.println("예외 발생: " + e.getMessage());
        }
    }
}
```
✅ 장점
1. 자원이 여러 개 있어도 깔끔하게 관리 가능.
2. try 블록이 끝나면 close()가 자동 호출됨.
3. 예외가 발생해도 원래 예외가 유지됨 (finally에서 덮어쓰는 문제 없음).

## 결론) try-with-resources가 훨씬 낫다!
