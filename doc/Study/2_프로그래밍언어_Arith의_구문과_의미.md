# 프로그래밍 언어 Arith의 구문과 의미

## Arith

### Arith언어의 구문

Arith 프로그램은 세미콜론으로 구분하여 나열한다.

```arith
x = 123; x = x + 1; z = 0; y * z ; y = x
```

각각 산술식(arithmetic expression)또는 할당식(assignment expression)형태이다.

왼쪽은 변수, 오른쪽에는 임의의 식이 온다.

식을 구성하는 가장 기본식(primary expression)은 변수나 숫자이다.

괄호를 사용하여 복잡한 식을 구성할 수 있다.

ex) `1 + (2 - 3) * 4 /5`

### Arith 프로그램의 AST

Arith 프로그래밍언어의 식물 Java로 작성할 수 있다.

`x = 123`

```java
new Assign("x", new Lit(123))
```

`x = x + 1`

```java
 new Assign("x", 
  new  BinOp.ADD, new Var("x"), new List(1)))
```

`y - 1 * 2 / 3`

```java
 new BinOp(BinOp.SUB,
 new Var("y"),
 new BinOp(BinOp.DIV, new BinOp(BinOp.MUL, new Lit(1), new Lit(2)),
 new Lit(3))
```

`x = 123; x = x + 1`

```java
 Expr[] exprs = {
  new Assign("x", new Lit(123)),
  new Assign("x", 
    new BinOp(BinOp.ADD, new Var("x"), new Lit(1)))
  };
 exprSeq = new ArrayList<Expr>(Arrays.asList(exprs));
```

위 코드들을 추상구문트리(Abstract syntax tree)라 한다. 이것은 소스프로그램의 구문을 트리 자료구조(tree data structure)로 요약해서 표현한 것이다.

위 추상구문 트리를 위해 Java클래스들을 준비해야 한다.

- arith.ast 패키지의 클래스들
  - Expr
  - Assign extends Expr
  - BinOp extends Expr
  - Lit extends Expr
  - Var extends Expr

- 세미콜론으로 구분된 식들을 표현할 때 Java의 ArrayList<Expr>클래스를 사용한다.

Q. 다음 식을 Java로 작성한 추상구문트리를 만들어 보시오.

- `z = y`

```java
 new Assign("z", new Var("y"));
```

- `z + 123`

```java
 new BinOp(Bin.ADD, new Var("z"), new Lit(123));
 ```

### 연산자 우선순위

다음 식을 Java로 구성할 때 주의해야 한다. 예를 들어 1 + 2 * 3에 대한 추상 구문 트리를 2가지로 만들 수 있다.

```java
new BinOp(BinOp.MUL, new BinOp(BinOp.ADD, new Lit(1), new Lit(2)), new Lit(3)))

new BinOp(BinOp.ADD, new Lit(1), new BinOp(BinOp.MUL, new Lit(2), new Lit(3)))
```

위와 같이 만들면 위에꺼는 `(1 + 2) * 3`이고, 아래꺼는 `1 + (2 * 3)`이다.

일반적으로 추상구문 트리에서는 아래쪽부터 계산한다.

연산자 우선순위(operator perecedence)규칙을 이용하여 해석을 해야한다.

`x = y = z`의 추상 구문 트리는

`new Assign("x", new Assign("y", Var("z")))`로 정의한다. 즉 x = (y = z)로 해석한다. 오늘쪽에서부터 차례로 변수에 넣도록 해석한다. 연산자 결합(operator associativity)규칙을 이용하여 만든다.

### Arith소스프로그램의 의미(semantics) 정의

해석기(interpreter)라고 하는 함수를 작성하여 프로그래밍 언어의 의미를 정의할 수 있다. 이 합수의 입력은 추상구문트리이고, 출력은 실행 결과이다. 이 함수의 입력은 추상구문트리이고, 출력은 실행결과이다. 예를 들어보자

x = 123; x = x + 1; z = 0; y * z; y = x

위 프로그램은 변수 x는 124, z는 0, y는 124이다.

이렇게 변수가 어느 값을 가지고 있는지 보관하는 자료구조를 환경(environgment)라고 부른다. 환경은 보통 `{ x=124, y=124, z=0 }로 표현된다.

Java의 HashMap<String, Integer>클래스를 사용하면 java로 쉽게 환경을 작성하여 다룰 수 있다.

```java
HashMap<String, Integer> env = new HashMap<String, Integer>()
  env.put("x", 124);
  env.put("y", 124);
  env.put("z", 0);
```

env.get("x")는 124가 될 것이다.

Arith의 해석기를 Interp 클래스의 seq함수와 expr함수로 작성해보자.

- seq함수: 세미콜론으로 구분된 식을 받아 환경을 변경하여 결과로 나타낸다
- expr함수: 하나의 식을 받아 환경을 바꾸고 정수를 실행 결과로 반환한다.

seq함수는 ArrayList<Expr> 객체로 표현된 식들을 받아 각 식마다 expr함수를 호출하여 차례대로 실행하도록 작성한다.

```java
void seq(ArrayList<Expr> exprList, HashMap<String, Integer> env) {
  int index = 0;
  while(index < exprList.size()) {
    Integer retV = expr(exprList.get(index), env)
    index = index + 1;
  }
}
```

expr함수는 다음과 같이 작성한다. 식을 Expr객체로 받아, 이 식이 BinOp이면 산술계산, Assign이면 변수에 값을 대입하여 환경을 변경하고, Lit이면 상수, Var이면 환경 env에서 변수를 읽는 것을 실행한다.

```java
Integer expr(Expr expr, HashMap<String, Integer> env) {
  if (expr instanceof BinOp) {
    BinOp binOpExpr = (BinOp)expr;

    Integer leftV = expr(binOpExpr.getLeft(), env);
    Integer rightV = expr(binOpExpr.getRight(), env);

    switch(binOpExpr.getOpKind()) {
    case BinOp.ADD:
      return leftV + rightV;
    case BinOp.SUB:
      return leftV - rightV;
    case BinOp.MUL:
      return leftV * rightV;
    case BinOp.DIV:
      return leftV / rightV;
    }
  } else if (expr instanceof Assign) {
    Assign assignExpr = (Assign)expr;

    String varName = assignExpr.getVarName();
    Expr rhs = assignExpr.getRhs();

    Integer rhsV = expr(rhs, env);
    env.put(varName, rhsV);

    return rhsV;
  } else if (expr instanceof Lit) {
    Lit litExpr = (Lit)expr;

    Integer intLitV = litExpr.getInteger();

    return intLitV;
  } else if (expr instanceof Var) {
    Var varExpr = (Var)expr;

    String varName = varExpr.getVarName();
    Integer varV = env.get(varName);
    assert varV != null;

    return varV;
  }
}
```

프로그래밍언어 Arith의 해석기를 사용하여 주어진 소스프로그램을 실행하는
방법은 다음과 같다.

```java
ArrayList<Expr> exprSeq =  ... 추상구문트리 ...

HashMap<String,Integer> env = new HashMap<String,Integer>();
Interp.seq(exprSeq, env);
```

exprSeq는 세미콜론으로 구분된 식들의 리스트에 대한 추상구문트리이다.
실행을 위해서 비어있는 환경 env를 만든다.
