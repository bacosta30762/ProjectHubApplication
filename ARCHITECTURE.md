# Architecture Documentation

## Overview

This application follows a layered architecture pattern with clear separation of concerns:

```
┌─────────────────────────────────────┐
│         Presentation Layer          │
│      (Controllers + Thymeleaf)      │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│          Business Layer             │
│           (Services)                │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│         Data Access Layer           │
│      (DAOs + Spring Data JPA)      │
└─────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────┐
│          Database Layer             │
│            (MySQL)                  │
└─────────────────────────────────────┘
```

## Architecture Layers

### 1. Presentation Layer (Controllers)

**Location**: `com.TrabajoFinal.controller`

**Responsibilities**:
- Handle HTTP requests and responses
- Validate input data
- Delegate business logic to services
- Return views or redirects

**Pattern**: MVC (Model-View-Controller)

**Key Controllers**:
- `ContactoController` - Contact form handling
- `ProyectosController` - Project CRUD operations
- `UsuarioController` - User management
- `RegistroController` - Registration and activation
- `ReporteController` - Report generation

**Best Practices**:
- Controllers should be thin (minimal logic)
- Use `@Controller` for view-based responses
- Use `@RestController` for JSON APIs (if needed)
- Validate input at controller level
- Handle exceptions appropriately

---

### 2. Business Layer (Services)

**Location**: `com.TrabajoFinal.service` and `com.TrabajoFinal.service.impl`

**Responsibilities**:
- Implement business logic
- Coordinate between multiple DAOs
- Handle transactions
- Validate business rules

**Pattern**: Service Layer Pattern

**Key Services**:
- `UsuarioService` - User business logic
- `ProyectosService` - Project business logic
- `RegistroService` - Registration workflow
- `CorreoService` - Email functionality
- `ReporteService` - Report generation

**Transaction Management**:
```java
@Transactional
public void save(Usuario usuario, boolean crearRolUser) {
    // Database operations are transactional
}
```

**Best Practices**:
- Use `@Service` annotation
- Implement interfaces for testability
- Use `@Transactional` for database operations
- Keep services focused on single responsibility

---

### 3. Data Access Layer (DAOs)

**Location**: `com.TrabajoFinal.dao`

**Responsibilities**:
- Abstract database operations
- Provide CRUD operations
- Define custom queries

**Pattern**: Repository Pattern (Spring Data JPA)

**Key DAOs**:
- `UsuarioDao` - User data access
- `ProyectosDao` - Project data access
- `RolDao` - Role data access

**Example**:
```java
public interface UsuarioDao extends JpaRepository<Usuario, Long> {
    Usuario findByUsername(String username);
    boolean existsByUsernameOrCorreo(String username, String correo);
}
```

**Best Practices**:
- Extend `JpaRepository` for standard operations
- Use method naming conventions for queries
- Use `@Query` for complex queries
- Keep DAOs focused on data access only

---

### 4. Domain Layer (Entities)

**Location**: `com.TrabajoFinal.domain`

**Responsibilities**:
- Represent database entities
- Define relationships
- Provide data structure

**Pattern**: Domain Model Pattern

**Key Entities**:
- `Usuario` - User entity
- `Proyectos` - Project entity
- `Rol` - Role entity
- `Contacto` - Contact form DTO

**JPA Annotations**:
```java
@Entity
@Table(name = "usuario")
public class Usuario {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idUsuario;
    
    @OneToMany
    @JoinColumn(name = "id_usuario")
    List<Rol> roles;
}
```

**Best Practices**:
- Use `@Entity` for database entities
- Use `@Data` (Lombok) for getters/setters
- Define relationships clearly
- Use DTOs for data transfer when needed

---

## Design Patterns

### 1. MVC (Model-View-Controller)

**Implementation**:
- **Model**: Domain entities and service layer
- **View**: Thymeleaf templates
- **Controller**: Spring MVC controllers

**Benefits**:
- Separation of concerns
- Easy to test
- Maintainable code

---

### 2. Service Layer Pattern

**Implementation**:
- Interface defines contract
- Implementation contains logic
- Controllers depend on interfaces

**Benefits**:
- Testability (mock services)
- Flexibility (swap implementations)
- Clear separation of concerns

---

### 3. Repository Pattern

**Implementation**:
- Spring Data JPA repositories
- Custom query methods
- Standard CRUD operations

**Benefits**:
- Reduced boilerplate code
- Type-safe queries
- Easy to extend

---

### 4. Dependency Injection

**Implementation**:
- Constructor injection (preferred)
- Field injection (via `@Autowired`)
- Spring manages dependencies

**Example**:
```java
@Service
public class UsuarioServiceImpl implements UsuarioService {
    private final UsuarioDao usuarioDao;
    
    @Autowired
    public UsuarioServiceImpl(UsuarioDao usuarioDao) {
        this.usuarioDao = usuarioDao;
    }
}
```

---

## Security Architecture

### Spring Security Configuration

**Location**: `ProjectConfig.java`

**Components**:
1. **Authentication**: UserDetailsService
2. **Authorization**: Role-based access control
3. **Password Encoding**: BCrypt
4. **Session Management**: Default Spring Security

**Security Filter Chain**:
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) {
    http.authorizeHttpRequests((request) -> 
        request
            .requestMatchers("/", "/index", "/servicios/**").permitAll()
            .requestMatchers("/proyectos/**").hasAnyRole("ADMIN", "USER")
            .requestMatchers("/usuario/listado").hasRole("ADMIN")
    );
    return http.build();
}
```

**User Details Service**:
- Loads user from database
- Provides roles for authorization
- Handles authentication

---

## Internationalization (i18n)

### Configuration

**Location**: `ProjectConfig.java`

**Components**:
1. **LocaleResolver**: Session-based locale resolution
2. **LocaleChangeInterceptor**: Changes locale via URL parameter
3. **MessageSource**: Loads messages from properties files

**Implementation**:
```java
@Bean
public LocaleResolver localeResolver() {
    SessionLocaleResolver slr = new SessionLocaleResolver();
    slr.setDefaultLocale(Locale.getDefault());
    return slr;
}
```

**Usage**:
- Change language: `?lang=en` or `?lang=es`
- Messages stored in `messages.properties`, `messages_en.properties`, `messages_es.properties`

---

## Email Service Architecture

### Email Flow

```
User Action
    ↓
Controller
    ↓
CorreoService
    ↓
JavaMailSender
    ↓
SMTP Server (Gmail)
```

**Configuration**:
- SMTP settings in `application.properties`
- HTML email support via `MimeMessage`
- Plain text email via `SimpleMailMessage`

---

## Report Generation Architecture

### JasperReports Integration

**Flow**:
```
Request
    ↓
ReporteController
    ↓
ReporteService
    ↓
JasperReports Engine
    ↓
DataSource (MySQL)
    ↓
Formatted Report (PDF/Excel/CSV)
```

**Components**:
- `.jasper` files: Compiled report templates
- `ReporteService`: Generates reports
- `DataSource`: Database connection
- Export formats: PDF, Excel, CSV

---

## Database Schema

### Entity Relationships

```
Usuario (1) ────< (Many) Rol
    │
    └─── id_usuario (Foreign Key in Rol)
```

**Tables**:
1. **usuario**: User information
2. **rol**: User roles (ROLE_USER, ROLE_ADMIN)
3. **proyectos**: Project data

---

## Configuration Management

### Application Properties

**Location**: `src/main/resources/application.properties`

**Categories**:
1. **Server**: Port, logging
2. **Database**: Connection, JPA settings
3. **Email**: SMTP configuration
4. **Thymeleaf**: Cache settings

**Best Practices**:
- Use environment variables for sensitive data
- Separate profiles (dev, prod)
- Never commit passwords to version control

---

## Testing Strategy

### Unit Testing
- Test services in isolation
- Mock DAOs and dependencies
- Use `@SpringBootTest` for integration tests

### Integration Testing
- Test controller endpoints
- Test database operations
- Test security configuration

---

## Deployment Considerations

### Production Checklist

1. **Security**:
   - Change default passwords
   - Use environment variables for secrets
   - Enable HTTPS
   - Configure CORS properly

2. **Performance**:
   - Enable Thymeleaf cache
   - Optimize database queries
   - Use connection pooling
   - Enable JPA second-level cache

3. **Monitoring**:
   - Add logging framework (Logback)
   - Monitor application metrics
   - Set up error tracking

4. **Database**:
   - Use production database
   - Set up backups
   - Optimize indexes
   - Monitor query performance

---

## Best Practices Summary

### Code Organization
- ✅ Follow package structure
- ✅ Use meaningful names
- ✅ Keep classes focused
- ✅ Document complex logic

### Security
- ✅ Always encode passwords
- ✅ Validate user input
- ✅ Use parameterized queries
- ✅ Implement CSRF protection

### Performance
- ✅ Use transactions appropriately
- ✅ Optimize database queries
- ✅ Cache when appropriate
- ✅ Lazy load relationships

### Maintainability
- ✅ Write clean code
- ✅ Use design patterns
- ✅ Document APIs
- ✅ Write tests

---

For API details, see [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
For setup instructions, see [README.md](README.md)

