# Best Practices Guide

This document outlines coding standards, best practices, and conventions for the project.

## Table of Contents

- [Code Style](#code-style)
- [Security Practices](#security-practices)
- [Database Practices](#database-practices)
- [API Design](#api-design)
- [Error Handling](#error-handling)
- [Testing](#testing)
- [Performance](#performance)
- [Documentation](#documentation)

---

## Code Style

### Naming Conventions

#### Classes
- Use PascalCase: `UsuarioController`, `ProyectosService`
- Be descriptive: `UsuarioServiceImpl` not `UserSvcImpl`

#### Methods
- Use camelCase: `getUsuario()`, `crearUsuario()`
- Use verbs: `save()`, `delete()`, `findByUsername()`

#### Variables
- Use camelCase: `usuario`, `proyectoList`
- Be meaningful: `usuario` not `u`, `proyectoList` not `list`

#### Constants
- Use UPPER_SNAKE_CASE: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`

### Code Organization

```java
// ✅ Good: Organized imports
package com.TrabajoFinal.controller;

import com.TrabajoFinal.domain.Usuario;
import com.TrabajoFinal.service.UsuarioService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class UsuarioController {
    // Fields first
    @Autowired
    private UsuarioService usuarioService;
    
    // Then methods
    public String listado(Model model) {
        // Implementation
    }
}
```

### Annotations Order

```java
// ✅ Good: Logical annotation order
@Controller
@Slf4j
@RequestMapping("/usuario")
public class UsuarioController {
    
    @Autowired
    private UsuarioService usuarioService;
    
    @GetMapping("/listado")
    @PreAuthorize("hasRole('ADMIN')")
    public String listado(Model model) {
        // Implementation
    }
}
```

---

## Security Practices

### 1. Password Handling

```java
// ✅ Good: Always encode passwords
@Autowired
private BCryptPasswordEncoder passwordEncoder;

public void save(Usuario usuario) {
    String encodedPassword = passwordEncoder.encode(usuario.getPassword());
    usuario.setPassword(encodedPassword);
    usuarioDao.save(usuario);
}

// ❌ Bad: Never store plain text passwords
public void save(Usuario usuario) {
    // Password is already in plain text - DANGEROUS!
    usuarioDao.save(usuario);
}
```

### 2. Input Validation

```java
// ✅ Good: Validate input
@PostMapping("/guardar")
public String guardar(@Valid Usuario usuario, BindingResult result) {
    if (result.hasErrors()) {
        return "/usuario/modifica";
    }
    usuarioService.save(usuario, true);
    return "redirect:/usuario/listado";
}

// ❌ Bad: No validation
@PostMapping("/guardar")
public String guardar(Usuario usuario) {
    usuarioService.save(usuario, true); // Could have invalid data!
    return "redirect:/usuario/listado";
}
```

### 3. SQL Injection Prevention

```java
// ✅ Good: Use JPA methods (parameterized queries)
Usuario findByUsername(String username);

// ❌ Bad: Never use string concatenation
@Query("SELECT u FROM Usuario u WHERE u.username = '" + username + "'")
Usuario findByUsername(String username); // SQL Injection risk!
```

### 4. Authentication Checks

```java
// ✅ Good: Use Spring Security annotations
@GetMapping("/admin")
@PreAuthorize("hasRole('ADMIN')")
public String adminPage() {
    return "admin";
}

// ✅ Good: Check in code if needed
@GetMapping("/profile")
public String profile(Model model, Authentication auth) {
    String username = auth.getName();
    Usuario usuario = usuarioService.getUsuarioPorUsername(username);
    model.addAttribute("usuario", usuario);
    return "profile";
}
```

### 5. Sensitive Data

```java
// ✅ Good: Use environment variables
@Value("${spring.mail.password}")
private String mailPassword;

// ❌ Bad: Hardcoded credentials
private String mailPassword = "myPassword123"; // Never do this!
```

---

## Database Practices

### 1. Transaction Management

```java
// ✅ Good: Use @Transactional for write operations
@Transactional
public void save(Usuario usuario, boolean crearRolUser) {
    usuario = usuarioDao.save(usuario);
    if (crearRolUser) {
        Rol rol = new Rol();
        rol.setNombre("ROLE_USER");
        rol.setIdUsuario(usuario.getIdUsuario());
        rolDao.save(rol);
    }
}

// ✅ Good: Use @Transactional(readOnly = true) for reads
@Transactional(readOnly = true)
public List<Usuario> getUsuarios() {
    return usuarioDao.findAll();
}
```

### 2. Query Optimization

```java
// ✅ Good: Fetch only needed data
@Query("SELECT u FROM Usuario u WHERE u.activo = true")
List<Usuario> findActiveUsuarios();

// ✅ Good: Use pagination for large datasets
Page<Usuario> findAll(Pageable pageable);

// ❌ Bad: Fetch all data when not needed
List<Usuario> findAll(); // Could be thousands of records!
```

### 3. Entity Relationships

```java
// ✅ Good: Use appropriate fetch types
@OneToMany(fetch = FetchType.LAZY)
@JoinColumn(name = "id_usuario")
List<Rol> roles;

// ✅ Good: Use cascade appropriately
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
List<Rol> roles;
```

### 4. Error Handling

```java
// ✅ Good: Handle database errors gracefully
@Transactional
public void delete(Usuario usuario) {
    try {
        usuarioDao.delete(usuario);
    } catch (DataIntegrityViolationException e) {
        log.error("Cannot delete user with ID: {}", usuario.getIdUsuario(), e);
        throw new ServiceException("Cannot delete user: has related records");
    }
}
```

---

## API Design

### 1. RESTful Endpoints

```java
// ✅ Good: RESTful naming
GET    /proyectos          → List all
GET    /proyectos/{id}     → Get one
POST   /proyectos          → Create
PUT    /proyectos/{id}     → Update
DELETE /proyectos/{id}     → Delete

// ❌ Bad: Non-RESTful
GET /getAllProyectos
POST /createProyecto
GET /deleteProyecto?id=1
```

### 2. HTTP Status Codes

```java
// ✅ Good: Appropriate status codes
@PostMapping("/guardar")
public ResponseEntity<Usuario> guardar(@Valid @RequestBody Usuario usuario) {
    Usuario saved = usuarioService.save(usuario, true);
    return ResponseEntity.status(HttpStatus.CREATED).body(saved);
}

@GetMapping("/{id}")
public ResponseEntity<Usuario> getUsuario(@PathVariable Long id) {
    Usuario usuario = usuarioService.getUsuario(id);
    if (usuario == null) {
        return ResponseEntity.notFound().build();
    }
    return ResponseEntity.ok(usuario);
}
```

### 3. Request/Response Validation

```java
// ✅ Good: Validate request body
@PostMapping("/guardar")
public String guardar(@Valid Usuario usuario, BindingResult result) {
    if (result.hasErrors()) {
        return "/usuario/modifica";
    }
    usuarioService.save(usuario, true);
    return "redirect:/usuario/listado";
}
```

---

## Error Handling

### 1. Exception Handling

```java
// ✅ Good: Centralized exception handling
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(EntityNotFoundException.class)
    public String handleNotFound(EntityNotFoundException e, Model model) {
        model.addAttribute("error", e.getMessage());
        return "error/404";
    }
    
    @ExceptionHandler(Exception.class)
    public String handleGeneric(Exception e, Model model) {
        log.error("Unexpected error", e);
        model.addAttribute("error", "An error occurred");
        return "error/500";
    }
}
```

### 2. Logging

```java
// ✅ Good: Use SLF4J with appropriate levels
@Slf4j
@Controller
public class UsuarioController {
    
    public String listado(Model model) {
        log.debug("Fetching user list");
        var usuarios = usuarioService.getUsuarios();
        log.info("Found {} users", usuarios.size());
        model.addAttribute("usuarios", usuarios);
        return "/usuario/listado";
    }
    
    public String guardar(Usuario usuario) {
        try {
            usuarioService.save(usuario, true);
            log.info("User saved: {}", usuario.getUsername());
        } catch (Exception e) {
            log.error("Error saving user: {}", usuario.getUsername(), e);
            throw e;
        }
        return "redirect:/usuario/listado";
    }
}
```

### 3. User-Friendly Messages

```java
// ✅ Good: Provide clear error messages
if (!usuarioService.existeUsuarioPorUsernameOCorreo(username, correo)) {
    model.addAttribute("error", "Username or email already exists");
    return "/registro/nuevo";
}

// ❌ Bad: Technical error messages
catch (DataIntegrityViolationException e) {
    model.addAttribute("error", e.getMessage()); // Too technical!
    return "/registro/nuevo";
}
```

---

## Testing

### 1. Unit Tests

```java
// ✅ Good: Test service layer
@ExtendWith(MockitoExtension.class)
class UsuarioServiceImplTest {
    
    @Mock
    private UsuarioDao usuarioDao;
    
    @InjectMocks
    private UsuarioServiceImpl usuarioService;
    
    @Test
    void testGetUsuario() {
        // Given
        Usuario usuario = new Usuario();
        usuario.setIdUsuario(1L);
        when(usuarioDao.findById(1L)).thenReturn(Optional.of(usuario));
        
        // When
        Usuario result = usuarioService.getUsuario(usuario);
        
        // Then
        assertNotNull(result);
        assertEquals(1L, result.getIdUsuario());
    }
}
```

### 2. Integration Tests

```java
// ✅ Good: Test controller endpoints
@SpringBootTest
@AutoConfigureMockMvc
class UsuarioControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void testListado() throws Exception {
        mockMvc.perform(get("/usuario/listado")
                .with(user("admin").roles("ADMIN")))
                .andExpect(status().isOk())
                .andExpect(view().name("/usuario/listado"));
    }
}
```

---

## Performance

### 1. Lazy Loading

```java
// ✅ Good: Use lazy loading for relationships
@OneToMany(fetch = FetchType.LAZY)
List<Rol> roles;

// Load when needed
@Transactional(readOnly = true)
public Usuario getUsuarioWithRoles(Long id) {
    Usuario usuario = usuarioDao.findById(id).orElse(null);
    if (usuario != null) {
        usuario.getRoles().size(); // Trigger lazy load
    }
    return usuario;
}
```

### 2. Caching

```java
// ✅ Good: Cache expensive operations
@Cacheable("usuarios")
@Transactional(readOnly = true)
public List<Usuario> getUsuarios() {
    return usuarioDao.findAll();
}

@CacheEvict(value = "usuarios", allEntries = true)
public void save(Usuario usuario, boolean crearRolUser) {
    usuarioDao.save(usuario);
}
```

### 3. Pagination

```java
// ✅ Good: Use pagination for large datasets
@GetMapping("/listado")
public String listado(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        Model model) {
    Page<Usuario> usuarios = usuarioService.getUsuarios(page, size);
    model.addAttribute("usuarios", usuarios);
    return "/usuario/listado";
}
```

---

## Documentation

### 1. JavaDoc

```java
// ✅ Good: Document public methods
/**
 * Saves or updates a user in the database.
 * 
 * @param usuario The user entity to save
 * @param crearRolUser If true, creates a default ROLE_USER role
 * @throws ServiceException if the user cannot be saved
 */
public void save(Usuario usuario, boolean crearRolUser) {
    // Implementation
}
```

### 2. Code Comments

```java
// ✅ Good: Explain why, not what
// Generate activation key for email verification
// Using 40 characters for security
String clave = demeClave();

// ❌ Bad: Obvious comments
// Set the username
usuario.setUsername(username);
```

### 3. README Updates

- Update README when adding new features
- Document breaking changes
- Include examples for new APIs

---

## Code Review Checklist

Before submitting code, check:

- [ ] Code follows naming conventions
- [ ] No hardcoded passwords or secrets
- [ ] Input validation is present
- [ ] Error handling is appropriate
- [ ] Logging is added for important operations
- [ ] Transactions are used correctly
- [ ] Security annotations are present
- [ ] Code is properly formatted
- [ ] Comments explain complex logic
- [ ] Tests are written (if applicable)

---

## Common Pitfalls to Avoid

### 1. N+1 Query Problem

```java
// ❌ Bad: Causes N+1 queries
List<Usuario> usuarios = usuarioService.getUsuarios();
for (Usuario u : usuarios) {
    u.getRoles().size(); // Query for each user!
}

// ✅ Good: Use JOIN FETCH
@Query("SELECT u FROM Usuario u JOIN FETCH u.roles")
List<Usuario> findAllWithRoles();
```

### 2. Transaction Boundaries

```java
// ❌ Bad: Transaction too large
@Transactional
public void processUsers() {
    for (Usuario u : usuarios) {
        // Long-running operation
        processUser(u); // Could timeout!
    }
}

// ✅ Good: Smaller transactions
public void processUsers() {
    for (Usuario u : usuarios) {
        processUserInTransaction(u);
    }
}

@Transactional
private void processUserInTransaction(Usuario u) {
    processUser(u);
}
```

### 3. Memory Leaks

```java
// ❌ Bad: Loading all data into memory
List<Usuario> usuarios = usuarioDao.findAll(); // Could be millions!

// ✅ Good: Use pagination or streaming
Page<Usuario> usuarios = usuarioDao.findAll(PageRequest.of(0, 100));
```

---

## Summary

- **Security First**: Always validate input, encode passwords, use parameterized queries
- **Clean Code**: Follow naming conventions, organize code logically
- **Error Handling**: Handle exceptions gracefully, log appropriately
- **Performance**: Use lazy loading, pagination, caching when needed
- **Testing**: Write tests for critical functionality
- **Documentation**: Document public APIs and complex logic

For more details, see:
- [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [README.md](README.md)

