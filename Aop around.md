# AOP

## @Around annotation

When using AOP @Around, if the advised method has the return declaration, the advising method(AOP) should **declare return and write the return statement for the advised method**. Because, **at last, it's the advising method who return the method to caller method, instead of the advised method**.

```java
public boolean addMember() {

    System.out.println(getClass() + ": DOING STUFF: ADDING A MEMBERSHIP ACCOUNT");   
    return true;
}

@Around("execution(*..addMember(..))")
public boolean performApiAnalytics2(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{

    System.out.println("\n ====> Executing @Before advice on dao using @Around <=================");

    boolean bl =  (Boolean) proceedingJoinPoint.proceed();
    System.out.println("====> Executing @After advice on dao using @Around <=================\n");

    return bl;
}
```

Otherwise, it throws Exception, because there is no return:

```txt
Exception in thread "main" org.springframework.aop.AopInvocationException: 
Null return value from advice does not match primitive return type for: public boolean com.code...
```

<br>
The execution flow without AOP:

```mermaid

sequenceDiagram

  Caller->>addMember(): call
  Note right of addMember(): do something, return true
  addMember()->>Caller: true

```

<br><br>
The execution flow with AOP @Around annotation:

```mermaid

sequenceDiagram

  Caller-->>$Proxy: call
  $Proxy->>$Proxy: do something before
  $Proxy->>addMember():call jointpoint proceed
  Note right of addMember(): do something, return true
  addMember()->>$Proxy: true
  $Proxy->>$Proxy: do something after
  $Proxy-->>Caller: return

```

<div style="text-align: right"><a href="https://stackoverflow.com/questions/33742074/execution-of-spring-aop-around-advice-in-dao-returns-null-in-service" title="StackOverFlow">Reference</a>, 11/20/2022</div>

---
