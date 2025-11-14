# Quick Reference Guide

Quick lookup for common tasks, endpoints, and code snippets.

## Table of Contents

- [Common Endpoints](#common-endpoints)
- [Code Snippets](#code-snippets)
- [Database Queries](#database-queries)
- [Configuration Snippets](#configuration-snippets)
- [Maven Commands](#maven-commands)
- [Troubleshooting](#troubleshooting)

---

## Common Endpoints

### Public Endpoints

```
GET  /                              → Home page
GET  /index                         → Home page
GET  /login                         → Login page
GET  /registro/nuevo                → Registration form
POST /registro/crearUsuario         → Create user
GET  /contacto/listado              → Contact form
POST /contacto/enviar-correo        → Send contact email
GET  /servicios/listado            → Services page
GET  /desarrolloSoftware/listado   → Software dev page
GET  /ecommerce/listado            → E-commerce page
GET  /ubicacion/listado            → Location page
```

### Protected Endpoints (Requires Authentication)

```
GET  /proyectos/listado            → List projects (ADMIN, USER)
GET  /proyectos/nuevo              → New project form (ADMIN, USER)
POST /proyectos/guardar            → Save project (ADMIN, USER)
GET  /proyectos/modificar/{id}    → Edit project (ADMIN, USER)
GET  /proyectos/eliminar/{id}      → Delete project (ADMIN, USER)

GET  /usuario/listado             → List users (ADMIN only)
GET  /usuario/nuevo                → New user form (ADMIN, USER)
POST /usuario/guardar              → Save user (ADMIN, USER)
GET  /usuario/modificar/{id}      → Edit user (ADMIN, USER)
GET  /usuario/eliminar/{id}        → Delete user (ADMIN, USER)

GET  /reportes/plantillareportes  → Reports page (ADMIN, USER)
GET  /reportes/usuarios?tipo=Pdf  → Users report (ADMIN, USER)
GET  /reportes/proyectos?tipo=Xls → Projects report (ADMIN, USER)
```

---

## Code Snippets

### Controller Example

```java
@Controller
@Slf4j
@RequestMapping("/example")
public class ExampleController {
    
    @Autowired
    private ExampleService exampleService;
    
    @GetMapping("/listado")
    public String listado(Model model) {
        var items = exampleService.getAll();
        model.addAttribute("items", items);
        return "/example/listado";
    }
    
    @PostMapping("/guardar")
    public String guardar(@Valid Example item, BindingResult result) {
        if (result.hasErrors()) {
            return "/example/modifica";
        }
        exampleService.save(item);
        return "redirect:/example/listado";
    }
}
```

### Service Example

```java
@Service
public class ExampleServiceImpl implements ExampleService {
    
    @Autowired
    private ExampleDao exampleDao;
    
    @Override
    @Transactional(readOnly = true)
    public List<Example> getAll() {
        return exampleDao.findAll();
    }
    
    @Override
    @Transactional
    public void save(Example item) {
        exampleDao.save(item);
    }
}
```

### DAO Example

```java
public interface ExampleDao extends JpaRepository<Example, Long> {
    Example findByNombre(String nombre);
    List<Example> findByActivoTrue();
    boolean existsByNombre(String nombre);
}
```

### Entity Example

```java
@Data
@Entity
@Table(name = "example")
public class Example implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String nombre;
    private boolean activo;
}
```

---

## Database Queries

### Common JPA Queries

```java
// Find by single field
Usuario findByUsername(String username);

// Find by multiple fields
Usuario findByUsernameAndPassword(String username, String password);

// Find by OR condition
Usuario findByUsernameOrCorreo(String username, String correo);

// Check existence
boolean existsByUsername(String username);

// Find with condition
List<Proyectos> findByActivoTrue();
List<Proyectos> findByProgresoGreaterThan(int progreso);

// Custom query
@Query("SELECT u FROM Usuario u WHERE u.activo = true")
List<Usuario> findActiveUsers();
```

### SQL Queries

```sql
-- Get all active projects
SELECT * FROM proyectos WHERE activo = TRUE;

-- Get user with roles
SELECT u.*, r.nombre as rol 
FROM usuario u 
LEFT JOIN rol r ON u.id_usuario = r.id_usuario 
WHERE u.username = 'admin';

-- Count users by role
SELECT r.nombre, COUNT(*) as count 
FROM rol r 
GROUP BY r.nombre;
```

---

## Configuration Snippets

### application.properties

```properties
# Server
server.port=8080

# Database
spring.datasource.url=jdbc:mysql://localhost:3306/trabajofinal
spring.datasource.username=admin01
spring.datasource.password=admin01

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true

# Email
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=app-password

# Thymeleaf
spring.thymeleaf.cache=false
```

### Security Configuration

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) {
    http.authorizeHttpRequests((request) -> 
        request
            .requestMatchers("/public/**").permitAll()
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
    );
    return http.build();
}
```

---

## Maven Commands

```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Package application
mvn package

# Run application
mvn spring-boot:run

# Install to local repository
mvn install

# Skip tests
mvn install -DskipTests

# Update dependencies
mvn clean install -U
```

---

## Common Tasks

### Create New Entity

1. Create domain class:
```java
@Entity
@Table(name = "nuevo")
public class Nuevo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // fields...
}
```

2. Create DAO:
```java
public interface NuevoDao extends JpaRepository<Nuevo, Long> {
}
```

3. Create Service:
```java
public interface NuevoService {
    List<Nuevo> getAll();
    void save(Nuevo item);
}
```

4. Create Service Implementation:
```java
@Service
public class NuevoServiceImpl implements NuevoService {
    @Autowired
    private NuevoDao nuevoDao;
    // implementation...
}
```

5. Create Controller:
```java
@Controller
@RequestMapping("/nuevo")
public class NuevoController {
    @Autowired
    private NuevoService nuevoService;
    // endpoints...
}
```

### Add New Role

```sql
INSERT INTO rol (nombre, id_usuario) 
VALUES ('ROLE_ADMIN', (SELECT id_usuario FROM usuario WHERE username = 'admin'));
```

### Reset User Password

```java
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String encoded = encoder.encode("newPassword");
usuario.setPassword(encoded);
usuarioService.save(usuario, false);
```

### Generate Report

```java
// PDF
ResponseEntity<Resource> pdf = reporteService.generarReporte("usuarios", null, "Pdf");

// Excel
ResponseEntity<Resource> excel = reporteService.generarReporte("proyectos", null, "Xls");

// CSV
ResponseEntity<Resource> csv = reporteService.generarReporte("usuarios", null, "Csv");
```

---

## Troubleshooting

### Application Won't Start

```bash
# Check port
netstat -ano | findstr :8080  # Windows
lsof -i :8080                  # Linux/Mac

# Check database connection
mysql -u admin01 -p trabajofinal

# Check logs
tail -f logs/application.log
```

### Database Connection Error

```properties
# Verify in application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/trabajofinal?useTimezone=true&serverTimezone=UTC
spring.datasource.username=admin01
spring.datasource.password=admin01
```

### Email Not Sending

1. Check Gmail app password
2. Verify SMTP settings
3. Check firewall
4. Test connection:
```java
@Autowired
private JavaMailSender mailSender;

public void testEmail() {
    SimpleMailMessage message = new SimpleMailMessage();
    message.setTo("test@example.com");
    message.setSubject("Test");
    message.setText("Test message");
    mailSender.send(message);
}
```

### Template Not Found

- Check template path: `src/main/resources/templates/`
- Verify controller return value matches template path
- Check Thymeleaf cache: set `spring.thymeleaf.cache=false` in dev

---

## Useful Links

- **Spring Boot Docs**: https://spring.io/projects/spring-boot
- **Spring Data JPA**: https://spring.io/projects/spring-data-jpa
- **Thymeleaf**: https://www.thymeleaf.org/
- **Bootstrap**: https://getbootstrap.com/
- **JasperReports**: https://community.jaspersoft.com/

---

## Keyboard Shortcuts (IntelliJ IDEA)

- `Ctrl + Shift + F10` - Run application
- `Ctrl + F9` - Build project
- `Ctrl + /` - Comment/uncomment
- `Alt + Enter` - Quick fix
- `Ctrl + B` - Go to declaration
- `Ctrl + Alt + L` - Format code
- `Shift + F6` - Rename

---

For detailed information, see:
- [README.md](README.md)
- [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [BEST_PRACTICES.md](BEST_PRACTICES.md)

