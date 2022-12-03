# record

- a special class type since Java 16

---

In traditional way, we create a class like (supposed that the fields are final):

```java
public class UserClass {

    private final int id;
    private final String name;
    private final LocalDate birthDate;

    public int getId() {

        return id;
    }
    public String getName() {

        return name;
    }
    public LocalDate getBirthDate() {

        return birthDate;
    }
    @Override
    public String toString() {

        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", birthDate=" + birthDate +
                '}';
    }
    
    @Override
    public boolean equals(Object o) {

        if (this == o) {return true;}
        if (o == null || getClass() != o.getClass()) {return false;}
        User user = (User) o;
        return id == user.id && name.equals(user.name) && birthDate.equals(user.birthDate);
    }
    
    @Override
    public int hashCode() {

        return Objects.hash(id, name, birthDate);
    }
}
```

or you cloud use lombok:

```java
@Getter
@RequiredArgsConstructor
@ToString
@EqualsAndHashCode
public class UserClass {

    private final int id;
    private final String name;
    private final LocalDate birthDate;
}
```

---

Now, since Java 16, we can create it using **record**,

```Java
public record UserRecord(int id, String name, LocalDate birthDate) {}
```

1. All the fields listed in the record will be set as **final decorated**, so the record is immutable. While the record has be instanced, the value can't be modified any more.
2. **record can't extend any other class.** all record extends java.lang.Record class, like enum extends java.lang.Enum class.
3. **record can't be extended by any other class.** record is implicitly decorated by final keyword.
4. For the field, you're **not allowed to create a non-static field**.

    ```java
    public record UserRecord(int id, String name, LocalDate birthDate) {
        public static final String DEFAULT_NAME="Tim";
        public String DEFAULT default2; // compile error
    }
    ```

5. You can implement any interface like normal class.
6. You can write method like other class, except for modifying the value of record instance.

   ```java
    public record UserRecord(int id, String name, LocalDate birthDate) {
        public String nameToUpperCase(){
            return this.name.toUpperCase();
        }
    }
    ```

7. Instead of using getXXX() to get the value, record instance use XXX() to get the value.

    ```java
        UserRecord userRecord = new UserRecord(1, "Jan", LocalDate.now());
        userRecord.name();  //instead of userRecord.getName()
    ```

8. You can create your own constructor,

   ```java
    public record UserRecord(int id, String name, LocalDate birthDate) {

        public UserRecord(int id, String name, LocalDate birthDate){
            if(id<0){
                throw new IllegalArgumentException("id is not allowed.");
            }
            this.id = id;   // must assign the value to the instance field
            this.name = name;
            this.birthDate = birthDate;
        }
    }
    ```

9. You can create **compact constructor**(unique in the record),

    ```java
    public record UserRecord(int id, String name, LocalDate birthDate) {

        public UserRecord{
            if(id<0){
                throw new IllegalArgumentException("id is not allowed.");
            }
        }
    }
    ```
