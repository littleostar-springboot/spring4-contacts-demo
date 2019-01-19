
---

#### spring in action, fourth edition

- chapter21
    - spring boot, contact demo
        - dependencies: jdbc, thymeleaf, web, h2
        - build: maven
        - package: war
        
---

#### code

Contact
```java
public class Contact {
    private Long id;
    private String firstName;
    private String lastName;
    private String phoneNumber;
    private String emailAddress;
    
    // ...
}
```

ContactRepository
```java
@Repository
public class ContactRepository {
    private JdbcTemplate jdbcTemplate;

    @Autowired
    public ContactRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<Contact> findAll() {
        return jdbcTemplate.query(
                "select id, firstName, lastName, phoneNumber, emailAddress " +
                        "from contacts order by lastName",
                new RowMapper<Contact>() {
                    @Override
                    public Contact mapRow(ResultSet resultSet, int i) throws SQLException {
                        Contact contact = new Contact(resultSet.getLong(1),
                                resultSet.getString(2),
                                resultSet.getString(3),
                                resultSet.getString(4),
                                resultSet.getString(5));
                        return contact;
                    }
                }
        );
    }

    public void save(Contact contact) {
        jdbcTemplate.update("insert into contacts " +
                        "(firstName, lastName, phoneNumber, emailAddress) " +
                        "values (?,?,?,?)",
                contact.getFirstName(), contact.getLastName(), contact.getPhoneNumber(), contact.getEmailAddress());
    }
}

```

ContactController
```java
@Controller
@RequestMapping("/")
public class ContactController {

    private ContactRepository contactRepository;

    @Autowired
    public ContactController(ContactRepository contactRepository) {
        this.contactRepository = contactRepository;
    }

    @RequestMapping(method = RequestMethod.GET)
    public String home(Map<String, Object> model) {
        List<Contact> contacts = contactRepository.findAll();
        model.put("contacts", contacts);
        return "home"; // ---> home view
    }

    @RequestMapping(method = RequestMethod.POST)
    public String submit(Contact contact) {
        contactRepository.save(contact);
        return "redirect:/";
    }
}
```

Application
```java
@ComponentScan
@EnableAutoConfiguration
public class Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
//        return super.configure(builder);
        return application.sources(Application.class);
    }
}
```

home.html
```html
<!DOCTYPE html>
<head xmlns:th="http://www.thymeleaf.org">
  <meta charset="UTF-8">
  <title>spring4 boot contact demo</title>
  <link rel="stylesheet" th:href="@{/style.css}"/>
</head>
<body>
<h2>spring4 boot contacts demo</h2>
<form method="post">
  <label for="firstName">firstName:</label>
  <input type="text" name="firstName"/><br>
  <label for="lastName">lastName:</label>
  <input type="text" name="lastName"/><br>
  <label for="phoneNumber">phoneNumber:</label>
  <input type="text" name="phoneNumber"/><br>
  <label for="emailAddress">emailAddress:</label>
  <input type="text" name="emailAddress"/><br>
  <input type="submit"/>
</form>
<ul th:each="contact : ${contacts}">
  <li>
    <span th:text="${contact.firstName}">firstName</span>
    <span th:text="${contact.lastName}">lastName</span> :
    <span th:text="${contact.phoneNumber}">phoneNumber</span>,
    <span th:text="${contact.emailAddress}">emailAddress</span>
  </li>
</ul>
</body>
</html>
```

schema.sql
```sql
create table contacts (
  id identity,
  firstName varchar(30) not null,
  lastName varchar(50) not null,
  phoneNumber varchar(13),
  emailAddress varchar(30)
);
```

---

https://github.com/littleostar-springboot/spring4-contacts-demo

---
end
