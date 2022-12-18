# Json infinite recursion in serialization

## Case description

```java
//Controller 
@GetMapping("/{id}/posts")
public List<Post> retrieveAllPostsForUser(@PathVariable int id) {

    log.info("retrieving all posts for the user id:{}", id);
    return userBusinessService.retrieveAllPostsForUser(id);
}

//DTO
// Lombok annotations
public class User {

    private int id;
    private String name;
    private LocalDate birthDate;
    private List<Post> postList;
}

// Lombok annotations
public class Post {

    private int id;

    private String description;

    @ToString.Exclude
    User user;
}

```

There is an annotation `@ToString.Exclude` to avoid infinite recursion during toString(). However, during the serialization work of Json, it will still invoke infinite recursion because of the **relationship(One to Many) of `User` and `Post`**

<br>

## Root cause:

Because the Statemaster model contains the object of Districtmaster model, which itself contains the object of Statemaster model. This causes an infinite json recursion.

## Solution:

1. Create a DTO and include only the fields that you want to display in the response.

2. You can use the @JsonManagedReference and @JsonBackReference annotations.

```java
// Lombok annotations
public class User {

    private int id;
    private String name;
    private LocalDate birthDate;

    @JsonManagedReference 
    private List<Post> postList;
}

// Lombok annotations
public class Post {

    private int id;
    private String description;

    @ToString.Exclude
    @JsonBackReference 
    User user;
}
```

<br>

reference:
1. <https://stackoverflow.com/questions/47693110/could-not-write-json-infinite-recursion-stackoverflowerror-nested-exception>
2. <https://cloud.tencent.com/developer/article/1711806>
