# Event Note

## Java - SpringBoot, AOP

### Proxy Problem - Resolved on 10/25/2022

- AOP doesn't work when the method should be advised by AOP is called by the other method in the class. Instance given below:

```Java

    @ComponentScan
    @EnableAspectJAutoProxy
    class Test{

        public static void main(String[] args){

            Test A = ...; //get bean from spring beans
            A.doTest(); // AOP works
            A.doMultipleTest(); // AOP doesn't work
        }

        public void doTest(){
            // AOP works on this method
            ...
        }

        public void doMultipleTest(){

            for(int i=0; i<10; i++>){
                doTest();
            }
        }
    }
    
    @Aspect
    @Component
    class TestAOP{
        
        @Pointcut("execution(* ..doTest(..))")
        private void pointcut(){}

        @Before("pointcut()")
        public void before(JoinPoint joinPoint){
            System.out.println("Hi");
        }
    }
```

If run `A.doTest();` in the source code, AOP works; however, when the code run `A.doMultipleTest();`, `doTest()` won't be advised by AOP. Here is the reason:

- Because when you run `A.doTest()` **the executer will be the instance `$Proxy`**. It comes to **`$Proxy.doTest()`**, and the execution order will be `System.out.println()` first, then `A.doTest()`.
- On the contrary, when you run `A.doMultipleTest()`, **the executer will be instance A** instead of `$Proxy`, **because `doMultipleTest()` doesn't be declared to be advised**. Further, **the executer of `doTest()` is `doMultipleTest()`** in the instance A. The execution order will be `A.doMultipleTest()` -> `this(instance A).doTest()` and the AOP won't advise `doTest()` because **AOP description does't exist in the `class Test`**.

Solution:

- **Specify instance `$Proxy` as the executer of `doTest()` in `doMultipleTest()`**
  - **`((className) AOPContext.currentProxy()).methodShouldBeAdvised();`**

``` Java
    public void doMultipleTest(){

        for(int i=0; i<10; i++>){
            ((Test)AOPContext.currentProxy()).doTest();
        }
    }
```

reference: <https://blog.csdn.net/u014788227/article/details/90111662>

---

在class內部method呼叫**應該被代理的method**，應該代理的AOP並不會執行，舉例如下:

```Java

    @ComponentScan
    @EnableAspectJAutoProxy
    class Test{

        public static void main(String[] args){

            Test A = ...; //get from spring beans
            A.doTest(); // AOP works
            A.doMultipleTest(); // AOP doesn't work
        }

        public void doTest(){
            // AOP works on this method
            ...
        }

        public void doMultipleTest(){

            for(int i=0; i<10; i++>){
                doTest();
            }
        }
    }
    
    @Aspect
    @Component
    class TestAOP{
        
        @Pointcut("execution(* ..doTest(..))")
        private void pointcut(){}

        @Before("pointcut()")
        public void before(JoinPoint joinPoint){
            System.out.println("Hi");
        }
    }
```

如果run `A.doTest();`，AOP會執行，但如果是 `A.doMultipleTest();`，則AOP不會生效。原因如下:

- 因為`A.doTest()`是由代理 instance `$Proxy`去執行doTest() -> **`$Proxy.doTest()`**，會先 `System.out.println()` 再去執行`instance A.doTest()`
- 而在執行`A.doMultipleTest()`則是由instance A 去執行`doMultipleTest()`-> **`A.doMultipleTest()`**，再由doMultipleTest()內部去call instance A內的doTest() -> `this.doTest()`。因此AOP不會執行，因為AOP並不存在於instance A內部。
\# **doMultipleTest()並沒有宣告被代理**，所以會由instance A去執行而不是代理instance `$Proxy`。

解決方法:

- **在doMultipleTest()要求用代理instance `$Proxy` 去執行 doTest()**

``` Java
    public void doMultipleTest(){

        for(int i=0; i<10; i++>){
            ((Test)AOPContext.currentProxy()).doTest();
        }
    }
```

<https://blog.csdn.net/u014788227/article/details/90111662>
