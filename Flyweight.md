# FlyWeight 
> Flyweight is a structural design pattern that lets you fit more objects into the available amount of RAM by sharing common parts of state between multiple objects instead of keeping all of the data in each object.

## 🗽Why ? 
유사한 데이터를 다수의 객체로 관리할 때, 내부의 상태를 공유하여 메모리 RAM 사용과 성능을 최적화하기 위해서 사용한다.

캐시와 유사하다.

## ✏️ When ? 

말 그대로 많은 객체가 비슷하거나 동일한 데이터를 가지고 있어서 중복 생성이 비효율적일 때 사용한다.

ex) 게임 개발 -> 같은 오브젝트 (총알)

ex) 텍스트 렌더링 및 문서 편집기 -> 폰트,서체 동일

## 😜 What ?

Flyweight 패턴에서는 변하지 않고 공유되는 속성인 Intrinsic State와 자주 변경되는 외부 속성인 Extrinsic State 속성 두 가지로 구분하여 공유되는 속성을 캐싱한다.

> Intrinsic State : 공유 속성 (본질적인 속성)
> 
> Extrinsic State : 공유되지 않는 속성 (외부적 속성)

### 🥸**언제 쓰고 왜 쓰는 지는 알겠는 데 어떻게 쓸까?**

예시로 총알피하기 게임이 있다고 해보자 우리가 흔히 아는 1942 아케이드 게임을 기억하면 쉽다.

![alt text](image-1.png) 

이 게임 속 총알을 만약 계속해서 new로 인스턴스를 생성하면 어떻게 될까? 


![alt text](image.png)

위의 사진과 같이 총알을 Particle 클래스로 만들면 백만개의 총알을 생성시 21GB나 잡아먹어서 시스템이 터지게 될 것이다. 

여기서 우리는 공유되는 속성과 공유되지 않는 외부 속성을 나누어 분리함으로써 메모리 효율적으로 사용할 수 있는 것이다.

여기서 총알의 색을 나타내는 Color와 총알의 이미지인 Sprite 변수가 메모리를 많이 차지하는 것을 볼 수 있다. 이 두가지 속성은 모든 총알이 똑같이 가지고 겹치는 속성이기에 이 속성을 분리한다.

그렇게 되면 그 외의 총알의 속도나 방향성 같이 각 총알마다 다른 속성만 새로 생성하고, 겹치는 속성인 색과 이미지는 공유하게 되어 메모리가 많이 세이브된다.

![alt text](image-3.png)

위의 그림과 같이 말도 안되게 세이브 되는 모습을 볼 수 있다.

## ✨구조 

*ConcreteFlyweight : 공유 객체*

*UnsahredConcreteFlyweight : 비공유 객체*

*FlyweightFactory : 객체 만드는 팩토리 및 캐시 풀 관리*


### 3가지 구조로 이루어짐 

에시 코드로 해당 구조를 설몀한다.

나무를 백만개를 랜더링해야되는 상황에서 공유되는 속성을 분리하고 공유되지 않는 속성을 분리하여 랜더링하는 코드.



## UnsahredConcreteFlyweight

비공유 객체로서 나무의 위치를 필드값으로 저장하고 있다. 그리고 공유객체인 TreeType을 상속 받는 것이 아닌 컴포지션하고 있다.

*컴포지션과 상속의 차이는 무엇일까?*


```java
package refactoring_guru.flyweight.example.trees;

import java.awt.*;

public class Tree {
    private int x;
    private int y;
    private TreeType type;

    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    public void draw(Graphics g) {
        type.draw(g, x, y);
    }
}
```

## ConcreteFlyweight

공유 객체로서 트리의 공통적인 속성인 나무 종의 이름과 색 등을 포함하고 있다.


```java
package refactoring_guru.flyweight.example.trees;

import java.awt.*;

public class TreeType {
    private String name;
    private Color color;
    private String otherTreeData;

    public TreeType(String name, Color color, String otherTreeData) {
        this.name = name;
        this.color = color;
        this.otherTreeData = otherTreeData;
    }

    public void draw(Graphics g, int x, int y) {
        g.setColor(Color.BLACK);
        g.fillRect(x - 1, y, 3, 5);
        g.setColor(color);
        g.fillOval(x - 5, y - 10, 10, 10);
    }
}
```
## FlyweightFactory

static한 HashMap인 treeTypes를 필드로 가짐으로서 캐싱해서 사용하는 것.
클래스가 메모리에 로드 될 때 한번만 생성되고 프로그램 전체에 공유된다.

### getTreeType

해당 메소드를 호출시 HashMap의 키가 존재하는 지 확인한다. 없을 경우 생성하여 저장하고 있을 경우 이미 존재하는 인스턴스를 리턴해준다. 이렇게 함으로써 공유하는 것.

```java
package refactoring_guru.flyweight.example.trees;

import java.awt.*;
import java.util.HashMap;
import java.util.Map;

public class TreeFactory {
    static Map<String, TreeType> treeTypes = new HashMap<>();

    public static TreeType getTreeType(String name, Color color, String otherTreeData) {
        TreeType result = treeTypes.get(name);
        if (result == null) {
            result = new TreeType(name, color, otherTreeData);
            treeTypes.put(name, result);
        }
        return result;
    }
}
```
## 구현 코드

해당 platTree 메소드를 호출 시 Extrisic 객체는 새롭게 생성하고, 공유 객체를 사용할 수 있는 지 확인 후 사용할 수 있으면 사용.

```java
package refactoring_guru.flyweight.example.forest;

import refactoring_guru.flyweight.example.trees.Tree;
import refactoring_guru.flyweight.example.trees.TreeFactory;
import refactoring_guru.flyweight.example.trees.TreeType;

import javax.swing.*;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class Forest extends JFrame {
    private List<Tree> trees = new ArrayList<>();

    public void plantTree(int x, int y, String name, Color color, String otherTreeData) {
        TreeType type = TreeFactory.getTreeType(name, color, otherTreeData);
        Tree tree = new Tree(x, y, type);
        trees.add(tree);
    }

    @Override
    public void paint(Graphics graphics) {
        for (Tree tree : trees) {
            tree.draw(graphics);
        }
    }
}
```

## Client 코드 

```java
package refactoring_guru.flyweight.example;

import refactoring_guru.flyweight.example.forest.Forest;

import java.awt.*;

public class Demo {
    static int CANVAS_SIZE = 500;
    static int TREES_TO_DRAW = 1000000;
    static int TREE_TYPES = 2;

    public static void main(String[] args) {
        Forest forest = new Forest();
        for (int i = 0; i < Math.floor(TREES_TO_DRAW / TREE_TYPES); i++) {
            forest.plantTree(random(0, CANVAS_SIZE), random(0, CANVAS_SIZE),
                    "Summer Oak", Color.GREEN, "Oak texture stub");
            forest.plantTree(random(0, CANVAS_SIZE), random(0, CANVAS_SIZE),
                    "Autumn Oak", Color.ORANGE, "Autumn Oak texture stub");
        }
        forest.setSize(CANVAS_SIZE, CANVAS_SIZE);
        forest.setVisible(true);

        System.out.println(TREES_TO_DRAW + " trees drawn");
        System.out.println("---------------------");
        System.out.println("Memory usage:");
        System.out.println("Tree size (8 bytes) * " + TREES_TO_DRAW);
        System.out.println("+ TreeTypes size (~30 bytes) * " + TREE_TYPES + "");
        System.out.println("---------------------");
        System.out.println("Total: " + ((TREES_TO_DRAW * 8 + TREE_TYPES * 30) / 1024 / 1024) +
                "MB (instead of " + ((TREES_TO_DRAW * 38) / 1024 / 1024) + "MB)");
    }

    private static int random(int min, int max) {
        return min + (int) (Math.random() * ((max - min) + 1));
    }
}
```

# 결과 

```
1000000 trees drawn
---------------------
Memory usage:
Tree size (8 bytes) * 1000000
+ TreeTypes size (~30 bytes) * 2
---------------------
Total: 7MB (instead of 36MB)
```



~~~~
출처: https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-Flyweight-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90

https://refactoring.guru/design-patterns/flyweight