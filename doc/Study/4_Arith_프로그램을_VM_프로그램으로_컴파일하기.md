# Arith 프로그램을 VM 프로그램으로 컴파일하기

Arith컴파일러는 소스프로그램의 추상구문트리를 입력받아 타겟 프로그램의 추상구문트리를 출력하는 함수로 작성한다.

- 소스프로그램의 추상구문트리: `ArrayList<Expr>`
- 타겟프로그램의 추상구문트리: `ArrayList<Instr>`

오버라이딩 된 동일한 이름의 compile함수를 작성해보자.

- 세미콜론으로 분리된 식 리스트를 받아 명령어 리스트를 출력하는 함수

```java
ArrayList<Instr> compile(ArrayList<Expr> exprSeq) {
  ArrayList<Instr> instrs = new ArrayList<Instr>();

  int index = 0;
  while (index < exprSeq.size()) {
    ArrayList<Instr> subInstrs = compile(exprSeq.get(index));

    instrs.addAll(subInstrs);
    instrs.add(new Pop());

    index = index + 1;
  }

  return instrs;
}
```

- 하나의 식을 입력받아 명령어 리스트를 출력하는 함수

```java
ArrayList<Instr> compile(Expr expr) {
  ArrayList<Instr> instrs = new ArrayList<Instr>();
  if (expr instanceof BinOp) {
    BinOp binOpExpr = (BinOp)expr;

    ArrayList<Instr> leftInstrs = compile(binOpExpr.getLeft());
    ArrayList<Instr> rightInstrs = compile(binOpExpr.getRight());

    instrs.addAll(leftInstrs);
    instrs.addAll(rightInstrs);

    switch(binOpExpr.getOpKind()) {
    case BinOp.ADD:
      instrs.add(new InstrOp(InstrOp.ADD));
      break;
    case BinOp.SUB:
      instrs.add(new InstrOp(InstrOp.SUB));
      break;
    case BinOp.MUL:
      instrs.add(new InstrOp(InstrOp.MUL));
      break;
    case BinOp.DIV:
      instrs.add(new InstrOp(InstrOp.DIV));
      break;
    }
  } else if (expr instanceof Assign) {
    Assign assignExpr = (Assign)expr;

    String varName = assignExpr.getVarName();
    Expr rhs = assignExpr.getRhs();

    ArrayList<Instr> rhsInstrs = compile(rhs);

    instrs.addAll(rhsInstrs);
    instrs.add(new Store(varName));
    instrs.add(new Push(varName));
  } else if (expr instanceof Lit) {
    Lit litExpr = (Lit)expr;

    Integer intLitV = litExpr.getInteger();

    instrs.add(new Push(intLitV));
  } else if (expr instanceof Var) {
    Var varExpr = (Var)expr;

    String varName = varExpr.getVarName();

    instrs.add(new Push(varName));
  }
  return instrs;
}
```

컴파일 과정을 다음과 같이 예를 들어서 볼 수 있다.

- `new Var("x") ===> new Push("x")`
- `new List(123) ==> new Push(!23)`
- `new Assign("x", 123) ===> new Push(123) new Stroe("x") new Push("x")`
- `new BinOp(BinOp.ADD, "x", 123) ===> new Push("x") new Push(123) new InstrOp(InstrOp.ADD)`

다음과 같이 컴파일러를 완성하였다. 이제 추상구문트리를 변환하여 타겟프로그램의 추상구문트리를 만들고 이 타겟프로그램을 가상기계를 통해 실행하는 과정을 쉽게 작성할 수 있다.

```java
ArrayList<Expr> exprSeq = ...소스프로그램 추상구문트리...
Expr.prettyPrint(exprSeq);

ArrayList<Instr> instrs = arith.comp.Compiler.compile(exprSeq);

Instr.prettyPrint(instrs);

HashMap<String, Integer> envVM = new HashMap<String, Integer> ();
VM.run(instrs, envVM);
```

소스프로그램 추상구문트리가 exprSeq에 주어져 있다고 가정할 때, compile함수로 타겟프로그램 추상구문트리 instrs를 만들고, 초기환경 envVM을 만들어 run함수로 이 타겟프로그램을 실행시킬 수 있다.
