# 목차
1장 오브젝트와 의존관계

# 들어가며
- 스프링은 자바 기반 기술이고, 객체지향 프로그래밍에 가치를 두고 만들어졌습니다.
- 스프링이 가장 많은 관심을 두는 대상은 오브젝트입니다.
- 스프링은 오브젝트의 설계, 구현, 사용, 개선에 대한 기준을 제공합니다.
# 1.1 초난감 DAO
- 이 책에서는 객체지향 원칙과 거리가 먼 코드를 '초난감'이라고 재치있게 표현했습니다.

## 1.1.1 User
- 다음의 코드를 사용합니다. 
- DB에 저장할 User 모델입니다. 

```java
package springbook.user.domain;

public class User {

    String id;
    String name;
    String password;

    public User() {}

    public User(String id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## 1.1.2 UserDao
- 다음의 코드를 사용합니다.

```java
package springbook.user.domain;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {

        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql://localhost/springbook", "spring", "book"
        );

        PreparedStatement ps = c.prepareStatement(
                "INSERT INTO users(id, name, password) VALUES(?, ?, ?)"
        );

        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {

        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql://localhost/springbook", "spring", "book"
        );

        PreparedStatement ps = c.prepareStatement(
                "SELECT * FROM users WHERE id = ?"
        );

        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();

        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```