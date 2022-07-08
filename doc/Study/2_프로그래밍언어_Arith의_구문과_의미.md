# 프로그래밍 언어 Arith의 구문과 의미

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

또한 Arith 프로그래밍언어의 식물 Java로 작성할 수 있다.

`x = 123`
> new Assign("x", new Lit(123))

`x = x + 1`
> new Assign("x", new  BinOp.ADD, new Var("x"), new List(1)))

`y - 1 * 2 / 3`
> new Var("y"), new BinOp(BinOp.DIV, new BinOp(BinOp.MUL, new Lit(1), new Lit(2)), new Lit(3))

`x = 123; x = x + 1`
> Expr[] exprs = {new Assign("x", new Lit(123)), new Assign("x", new BinOp(BinOp.ADD, new Var("x"), new Lit(1)))};
> exprSeq = new ArrayList<Expr>(Arrays.asList(exprs));

