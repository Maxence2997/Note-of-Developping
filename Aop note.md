# Note

## AOP

### @Around annotation

When use AOP @Around, the method will be advised should **declare return type with Object type instead of primitive type**.

```java
///public boolean addSillyMember() {
    public Boolean addSillyMember() {
        System.out.println(getClass() + ": DOING STUFF: ADDING A MEMBERSHIP ACCOUNT");
        
        return true;
    }
```

Otherwise, it throws Exception:

```txt
Exception in thread "main" org.springframework.aop.AopInvocationException: 
Null return value from advice does not match primitive return type for: public boolean com.code...
```

<div style="text-align: right">10/30/2022</div>
<br>
