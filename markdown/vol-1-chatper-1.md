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

## 1.3.3 main()을 이용한 DAO 테스트 코드
- 책에서는 `public static main()` 메소드를 직접 만드는 내용이 있으나, JUnit5로 대신했습니다.
- 다음의 코드를 사용합니다.

```java
package springbook.user.domain;

import org.junit.jupiter.api.Test;

import java.sql.SQLException;

class UserDaoTest {

    @Test
    void addAndGet() throws SQLException, ClassNotFoundException {

        UserDao dao = new UserDao();

        User user = new User();
        user.setId("whiteship");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);

        System.out.println(user.getId() + " 등록 성공");

        User user2 = dao.get(user.getId());
        System.out.println(user2.getName());
        System.out.println(user2.getPassword());
        System.out.println(user2.getId() + " 조회 성공");
    }
}
```

그러나 다음과 같은 오류가 발생합니다. 

```
java.lang.ClassNotFoundException: com.mysql.jdbc.Driver
```

MySQL8 버전에 맞는 JDBC 드라이버를 다운로드해 프로젝트에 추가해봐도 소용이 없었습니다. 

그러던 중 JDBC 4.0부터는 `Class.forName()`을 사용하지 않아도 된다는 정보를 발견합니다. [출처](https://docs.oracle.com/javadb/10.8.3.0/ref/rrefjdbc4_0summary.html)

```
Autoloading of JDBC drivers. In earlier versions of JDBC, applications had to manually register drivers before requesting Connections. With JDBC 4.0, applications no longer need to issue a Class.forName() on the driver name; instead, the DriverManager will find an appropriate JDBC driver when the application requests a Connection.
```

UserDao로 이동해 `add()`, `get()` 메소드의 `Class.forName()`을 삭제합니다.

이번엔 다음과 같은 오류가 발생합니다.

```
java.sql.SQLException: No suitable driver found for jdbc:mysql://localhost/springbook
```

당연합니다. JDBC 드라이버가 없으니까요. 

`pom.xml`에 의존성을 추가합니다. 제 PC의 MySQL 버전은 8.0.30 입니다. 

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.30</version>
</dependency>
```

다시 테스트를 실행하면 성공합니다.  

```markdown
whiteship 등록 성공
백기선
married
whiteship 조회 성공

Process finished with exit code 0
```

