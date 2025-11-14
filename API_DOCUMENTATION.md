# API Documentation

Complete API reference for all public endpoints, services, and components.

## Table of Contents

- [Controllers](#controllers)
- [Service Layer](#service-layer)
- [Domain Models](#domain-models)
- [Data Access Layer](#data-access-layer)
- [Usage Examples](#usage-examples)

---

## Controllers

### ContactoController

Handles contact form submissions and email notifications.

#### Endpoints

##### `GET /contacto/listado`
Display the contact form page.

**Response**: Thymeleaf template `/contacto/listado`

---

##### `POST /contacto/enviar-correo`
Submit contact form and send email notification.

**Content-Type**: `multipart/form-data`

**Request Parameters**:
- `nombre` (String, required) - Contact's first name
- `apellidos` (String, required) - Contact's last name
- `email` (String, required) - Contact's email address
- `telefono` (int, required) - Contact's phone number
- `mensaje` (String, required) - Message content
- `archivo` (MultipartFile, optional) - Optional file attachment

**Response**: Redirects to `/contacto/salida`

**Example Request**:
```html
<form action="/contacto/enviar-correo" method="post" enctype="multipart/form-data">
    <input type="text" name="nombre" required>
    <input type="text" name="apellidos" required>
    <input type="email" name="email" required>
    <input type="number" name="telefono" required>
    <textarea name="mensaje" required></textarea>
    <input type="file" name="archivo">
    <button type="submit">Send</button>
</form>
```

---

### ProyectosController

Manages project CRUD operations. Requires authentication.

#### Endpoints

##### `GET /proyectos/listado`
List all projects.

**Authentication**: Required (ADMIN, USER)

**Response**: Thymeleaf template with projects list

**Model Attributes**:
- `proyectos` (List<Proyectos>) - List of all projects
- `totalProyectos` (int) - Total number of projects

---

##### `GET /proyectos/nuevo`
Display form to create a new project.

**Authentication**: Required (ADMIN, USER)

**Response**: Thymeleaf template `/proyectos/modifica`

---

##### `POST /proyectos/guardar`
Save a new or update an existing project.

**Authentication**: Required (ADMIN, USER)

**Request Body**: Proyectos object (form data)

**Response**: Redirects to `/proyectos/listado`

**Example**:
```html
<form action="/proyectos/guardar" method="post">
    <input type="hidden" name="id" th:value="${proyectos.id}">
    <input type="text" name="nombre" required>
    <textarea name="descripcion"></textarea>
    <input type="number" name="progreso" min="0" max="100">
    <textarea name="comentarios"></textarea>
    <input type="checkbox" name="activo">
    <button type="submit">Save</button>
</form>
```

---

##### `GET /proyectos/modificar/{id}`
Display form to edit an existing project.

**Authentication**: Required (ADMIN, USER)

**Path Variables**:
- `id` (Long) - Project ID

**Response**: Thymeleaf template `/proyectos/modifica` with project data

---

##### `GET /proyectos/eliminar/{id}`
Delete a project.

**Authentication**: Required (ADMIN, USER)

**Path Variables**:
- `id` (Long) - Project ID

**Response**: Redirects to `/proyectos/listado`

---

### UsuarioController

Manages user CRUD operations. Admin-only for listing.

#### Endpoints

##### `GET /usuario/listado`
List all users (Admin only).

**Authentication**: Required (ADMIN)

**Response**: Thymeleaf template with users list

**Model Attributes**:
- `usuarios` (List<Usuario>) - List of all users
- `totalUsuarios` (int) - Total number of users

---

##### `GET /usuario/nuevo`
Display form to create a new user.

**Authentication**: Required (ADMIN, USER)

**Response**: Thymeleaf template `/usuario/modifica`

---

##### `POST /usuario/guardar`
Save a new or update an existing user.

**Authentication**: Required (ADMIN, USER)

**Content-Type**: `multipart/form-data`

**Request Parameters**:
- User fields (form data)
- `imagenFile` (MultipartFile, optional) - Profile image

**Response**: Redirects to `/usuario/listado`

---

##### `GET /usuario/modificar/{idUsuario}`
Display form to edit an existing user.

**Authentication**: Required (ADMIN, USER)

**Path Variables**:
- `idUsuario` (Long) - User ID

**Response**: Thymeleaf template `/usuario/modifica` with user data

---

##### `GET /usuario/eliminar/{idUsuario}`
Delete a user.

**Authentication**: Required (ADMIN, USER)

**Path Variables**:
- `idUsuario` (Long) - User ID

**Response**: Redirects to `/usuario/listado`

---

### RegistroController

Handles user registration and account activation.

#### Endpoints

##### `GET /registro/nuevo`
Display registration form.

**Response**: Thymeleaf template `/registro/nuevo`

---

##### `POST /registro/crearUsuario`
Create a new user account.

**Request Body**: Usuario object (form data)

**Response**: Thymeleaf template `/registro/salida`

**Process**:
1. Validates username/email uniqueness
2. Generates activation key
3. Sends activation email
4. Sets user as inactive

**Model Attributes**:
- `titulo` (String) - Message title
- `mensaje` (String) - Status message

---

##### `GET /registro/recordar`
Display password recovery form.

**Response**: Thymeleaf template `/registro/recordar`

---

##### `POST /registro/recordarUsuario`
Request password recovery.

**Request Body**: Usuario object with username or email

**Response**: Thymeleaf template `/registro/salida`

**Process**:
1. Finds user by username or email
2. Generates new password
3. Sends recovery email with activation link

---

##### `GET /registro/activacion/{usuario}/{id}`
Display account activation page.

**Path Variables**:
- `usuario` (String) - Username
- `id` (String) - Activation key

**Response**: 
- `/registro/activa` if valid
- `/registro/salida` if invalid

---

##### `POST /registro/activar`
Complete account activation with password and image.

**Content-Type**: `multipart/form-data`

**Request Parameters**:
- User fields (form data)
- `imagenFile` (MultipartFile, optional) - Profile image

**Response**: Redirects to `/` (home page)

**Process**:
1. Encodes password with BCrypt
2. Saves profile image if provided
3. Activates user account

---

### ReporteController

Generates reports in various formats.

#### Endpoints

##### `GET /reportes/plantillareportes`
Display reports page.

**Authentication**: Required (ADMIN, USER)

**Response**: Thymeleaf template `/reportes/plantillareportes`

---

##### `GET /reportes/usuarios?tipo={format}`
Generate users report.

**Authentication**: Required (ADMIN, USER)

**Query Parameters**:
- `tipo` (String, required) - Report format: `Pdf`, `vPdf`, `Xls`, `Csv`

**Response**: File download or inline display

**Example**:
```
GET /reportes/usuarios?tipo=Pdf
GET /reportes/usuarios?tipo=vPdf  (inline PDF)
GET /reportes/usuarios?tipo=Xls
GET /reportes/usuarios?tipo=Csv
```

---

##### `GET /reportes/proyectos?tipo={format}`
Generate projects report.

**Authentication**: Required (ADMIN, USER)

**Query Parameters**:
- `tipo` (String, required) - Report format: `Pdf`, `vPdf`, `Xls`, `Csv`

**Response**: File download or inline display

---

### Service Page Controllers

Simple controllers that return view templates:

- `GET /servicios/listado` → `/servicios/listado`
- `GET /desarrolloSoftware/listado` → `/desarrolloSoftware/listado`
- `GET /ecommerce/listado` → `/ecommerce/listado`
- `GET /ubicacion/listado` → `/ubicacion/listado`

---

## Service Layer

### CorreoService

Email service interface for sending emails.

#### Methods

##### `void enviarCorreo(Contacto correoRequest)`
Send a simple text email from contact form.

**Parameters**:
- `correoRequest` (Contacto) - Contact information

**Implementation**: `CorreoServiceImpl`

**Example**:
```java
@Autowired
private CorreoService correoService;

Contacto contacto = new Contacto();
contacto.setNombre("John");
contacto.setApellidos("Doe");
contacto.setEmail("john@example.com");
contacto.setTelefono(1234567890);
contacto.setMensaje("Hello!");

correoService.enviarCorreo(contacto);
```

---

##### `void enviarCorreoHtml(String para, String asunto, String contenidoHtml)`
Send an HTML email.

**Parameters**:
- `para` (String) - Recipient email
- `asunto` (String) - Email subject
- `contenidoHtml` (String) - HTML content

**Throws**: `MessagingException`

**Example**:
```java
String htmlContent = "<h1>Welcome</h1><p>Your account has been activated.</p>";
correoService.enviarCorreoHtml("user@example.com", "Welcome", htmlContent);
```

---

### ProyectosService

Business logic for project management.

#### Methods

##### `List<Proyectos> getProyectos(boolean activos)`
Get list of projects.

**Parameters**:
- `activos` (boolean) - If true, returns only active projects

**Returns**: List of Proyectos

**Example**:
```java
@Autowired
private ProyectosService proyectosService;

// Get all projects
List<Proyectos> allProjects = proyectosService.getProyectos(false);

// Get only active projects
List<Proyectos> activeProjects = proyectosService.getProyectos(true);
```

---

##### `Proyectos getProyectos(Proyectos proyecto)`
Get a single project by ID.

**Parameters**:
- `proyecto` (Proyectos) - Project with ID set

**Returns**: Proyectos object or null

**Example**:
```java
Proyectos proyecto = new Proyectos();
proyecto.setId(1L);
Proyectos found = proyectosService.getProyectos(proyecto);
```

---

##### `void save(Proyectos proyecto)`
Save or update a project.

**Parameters**:
- `proyecto` (Proyectos) - Project to save

**Note**: If ID is null, creates new; otherwise updates existing

**Example**:
```java
Proyectos proyecto = new Proyectos();
proyecto.setNombre("New Project");
proyecto.setDescripcion("Description");
proyecto.setProgreso(0);
proyecto.setActivo(true);
proyectosService.save(proyecto);
```

---

##### `void delete(Proyectos proyecto)`
Delete a project.

**Parameters**:
- `proyecto` (Proyectos) - Project with ID set

**Example**:
```java
Proyectos proyecto = new Proyectos();
proyecto.setId(1L);
proyectosService.delete(proyecto);
```

---

### UsuarioService

Business logic for user management.

#### Methods

##### `List<Usuario> getUsuarios()`
Get all users.

**Returns**: List of all Usuario objects

---

##### `Usuario getUsuario(Usuario usuario)`
Get user by ID.

**Parameters**:
- `usuario` (Usuario) - User with idUsuario set

**Returns**: Usuario object or null

---

##### `Usuario getUsuarioPorUsername(String username)`
Get user by username.

**Parameters**:
- `username` (String) - Username

**Returns**: Usuario object or null

---

##### `Usuario getUsuarioPorUsernameYPassword(String username, String password)`
Get user by username and password (plain text).

**Parameters**:
- `username` (String) - Username
- `password` (String) - Plain text password

**Returns**: Usuario object or null

**Note**: Used for activation, not authentication

---

##### `Usuario getUsuarioPorUsernameOCorreo(String username, String correo)`
Get user by username or email.

**Parameters**:
- `username` (String) - Username
- `correo` (String) - Email

**Returns**: Usuario object or null

---

##### `boolean existeUsuarioPorUsernameOCorreo(String username, String correo)`
Check if user exists by username or email.

**Parameters**:
- `username` (String) - Username
- `correo` (String) - Email

**Returns**: true if user exists

---

##### `void save(Usuario usuario, boolean crearRolUser)`
Save or update a user.

**Parameters**:
- `usuario` (Usuario) - User to save
- `crearRolUser` (boolean) - If true, creates default ROLE_USER

**Example**:
```java
Usuario usuario = new Usuario();
usuario.setUsername("johndoe");
usuario.setPassword("encodedPassword");
usuario.setNombre("John");
usuario.setApellidos("Doe");
usuario.setCorreo("john@example.com");
usuario.setActivo(true);

// Create new user with default role
usuarioService.save(usuario, true);

// Update existing user without changing role
usuarioService.save(usuario, false);
```

---

##### `void delete(Usuario usuario)`
Delete a user.

**Parameters**:
- `usuario` (Usuario) - User with idUsuario set

---

### RegistroService

Business logic for user registration and activation.

#### Methods

##### `Model crearUsuario(Model model, Usuario usuario)`
Create a new user account and send activation email.

**Parameters**:
- `model` (Model) - Spring Model object
- `usuario` (Usuario) - User data

**Returns**: Model with status message

**Throws**: `MessagingException`

**Process**:
1. Validates username/email uniqueness
2. Generates random activation key
3. Sets user as inactive
4. Saves user
5. Sends activation email

---

##### `Model activar(Model model, String usuario, String clave)`
Validate activation link and prepare activation form.

**Parameters**:
- `model` (Model) - Spring Model object
- `usuario` (String) - Username
- `clave` (String) - Activation key

**Returns**: Model with user data if valid

---

##### `void activar(Usuario usuario, MultipartFile imagenFile)`
Complete user activation.

**Parameters**:
- `usuario` (Usuario) - User with new password
- `imagenFile` (MultipartFile) - Optional profile image

**Process**:
1. Encodes password with BCrypt
2. Saves profile image if provided
3. Activates user account

---

##### `Model recordarUsuario(Model model, Usuario usuario)`
Request password recovery.

**Parameters**:
- `model` (Model) - Spring Model object
- `usuario` (Usuario) - User with username or email

**Returns**: Model with status message

**Throws**: `MessagingException`

---

### ReporteService

Service for generating reports.

#### Methods

##### `ResponseEntity<Resource> generarReporte(String reporte, Map<String,Object> parametros, String tipo)`
Generate a report in specified format.

**Parameters**:
- `reporte` (String) - Report name (without .jasper extension)
- `parametros` (Map<String,Object>) - Report parameters (can be null)
- `tipo` (String) - Format: `Pdf`, `vPdf`, `Xls`, `Csv`

**Returns**: ResponseEntity with file resource

**Throws**: `IOException`

**Example**:
```java
@Autowired
private ReporteService reporteService;

// Generate PDF report
ResponseEntity<Resource> pdf = reporteService.generarReporte(
    "usuarios", 
    null, 
    "Pdf"
);

// Generate Excel report with parameters
Map<String, Object> params = new HashMap<>();
params.put("fechaInicio", "2024-01-01");
ResponseEntity<Resource> excel = reporteService.generarReporte(
    "proyectos", 
    params, 
    "Xls"
);
```

---

## Domain Models

### Contacto

Contact form data model.

**Fields**:
- `nombre` (String) - First name
- `apellidos` (String) - Last name
- `email` (String) - Email address
- `telefono` (int) - Phone number
- `mensaje` (String) - Message content

**Example**:
```java
Contacto contacto = new Contacto();
contacto.setNombre("John");
contacto.setApellidos("Doe");
contacto.setEmail("john@example.com");
contacto.setTelefono(1234567890);
contacto.setMensaje("Hello, I need help!");
```

---

### Proyectos

Project entity model.

**Table**: `proyectos`

**Fields**:
- `id` (Long) - Primary key (auto-generated)
- `nombre` (String) - Project name
- `descripcion` (String) - Project description
- `progreso` (int) - Progress percentage (0-100)
- `comentarios` (String) - Additional comments
- `activo` (boolean) - Active status

**JPA Annotations**:
- `@Entity`
- `@Table(name = "proyectos")`
- `@Id @GeneratedValue(strategy = GenerationType.IDENTITY)`

**Example**:
```java
Proyectos proyecto = new Proyectos();
proyecto.setNombre("Website Redesign");
proyecto.setDescripcion("Complete redesign of company website");
proyecto.setProgreso(75);
proyecto.setComentarios("On track for completion");
proyecto.setActivo(true);
```

---

### Usuario

User entity model.

**Table**: `usuario`

**Fields**:
- `idUsuario` (Long) - Primary key (auto-generated)
- `username` (String) - Unique username
- `password` (String) - BCrypt encoded password
- `nombre` (String) - First name
- `apellidos` (String) - Last name
- `correo` (String) - Email address
- `telefono` (String) - Phone number
- `rutaImagen` (String) - Profile image path
- `activo` (boolean) - Account active status
- `roles` (List<Rol>) - User roles (OneToMany)

**JPA Annotations**:
- `@Entity`
- `@Table(name = "usuario")`
- `@OneToMany` relationship with Rol

**Example**:
```java
Usuario usuario = new Usuario();
usuario.setUsername("johndoe");
usuario.setPassword("$2a$10$encoded...");
usuario.setNombre("John");
usuario.setApellidos("Doe");
usuario.setCorreo("john@example.com");
usuario.setTelefono("1234567890");
usuario.setActivo(true);
```

---

### Rol

Role entity model for Spring Security.

**Table**: `rol`

**Fields**:
- `idRol` (Long) - Primary key (auto-generated)
- `nombre` (String) - Role name (e.g., "ROLE_USER", "ROLE_ADMIN")
- `idUsuario` (Long) - Foreign key to usuario

**JPA Annotations**:
- `@Entity`
- `@Table(name = "rol")`

**Example**:
```java
Rol rol = new Rol();
rol.setNombre("ROLE_ADMIN");
rol.setIdUsuario(1L);
```

---

## Data Access Layer

### UsuarioDao

Spring Data JPA repository for Usuario.

**Extends**: `JpaRepository<Usuario, Long>`

**Custom Methods**:
- `Usuario findByUsername(String username)`
- `Usuario findByUsernameAndPassword(String username, String password)`
- `Usuario findByUsernameOrCorreo(String username, String correo)`
- `boolean existsByUsernameOrCorreo(String username, String correo)`

**Example**:
```java
@Autowired
private UsuarioDao usuarioDao;

Usuario user = usuarioDao.findByUsername("johndoe");
boolean exists = usuarioDao.existsByUsernameOrCorreo("johndoe", "john@example.com");
```

---

### ProyectosDao

Spring Data JPA repository for Proyectos.

**Extends**: `JpaRepository<Proyectos, Long>`

**Example**:
```java
@Autowired
private ProyectosDao proyectosDao;

List<Proyectos> all = proyectosDao.findAll();
Optional<Proyectos> proyecto = proyectosDao.findById(1L);
```

---

### RolDao

Spring Data JPA repository for Rol.

**Extends**: `JpaRepository<Rol, Long>`

---

## Usage Examples

### Complete User Registration Flow

```java
// 1. User submits registration form
@PostMapping("/registro/crearUsuario")
public String crearUsuario(Model model, Usuario usuario) {
    model = registroService.crearUsuario(model, usuario);
    return "/registro/salida";
}

// 2. User receives email and clicks activation link
// GET /registro/activacion/{usuario}/{id}

// 3. User completes activation
@PostMapping("/registro/activar")
public String activar(Usuario usuario, MultipartFile imagenFile) {
    registroService.activar(usuario, imagenFile);
    return "redirect:/";
}
```

### Creating and Managing Projects

```java
// Create new project
Proyectos proyecto = new Proyectos();
proyecto.setNombre("New Project");
proyecto.setDescripcion("Description");
proyecto.setProgreso(0);
proyecto.setActivo(true);
proyectosService.save(proyecto);

// Update project
proyecto.setProgreso(50);
proyecto.setComentarios("Halfway done");
proyectosService.save(proyecto);

// Get active projects
List<Proyectos> active = proyectosService.getProyectos(true);

// Delete project
proyectosService.delete(proyecto);
```

### Generating Reports

```java
// Generate PDF report (download)
ResponseEntity<Resource> pdf = reporteService.generarReporte(
    "usuarios", null, "Pdf"
);

// Generate inline PDF
ResponseEntity<Resource> inline = reporteService.generarReporte(
    "proyectos", null, "vPdf"
);

// Generate Excel report
ResponseEntity<Resource> excel = reporteService.generarReporte(
    "usuarios", null, "Xls"
);

// Generate CSV report
ResponseEntity<Resource> csv = reporteService.generarReporte(
    "proyectos", null, "Csv"
);
```

---

## Error Handling

### Common Errors

1. **Username/Email Already Exists**
   - Message: "Ya existe un usuario con el username {username} o el correo {correo}"
   - Solution: Use different username or email

2. **Invalid Activation Link**
   - Message: "Error activando usuario"
   - Solution: Request new activation email

3. **Access Denied (403)**
   - Message: "No tiene permisos para ver esta página"
   - Solution: Ensure user has required role

---

## Security Considerations

1. **Password Storage**: Always use BCrypt encoding
2. **SQL Injection**: Handled by JPA parameterized queries
3. **CSRF Protection**: Enabled by default in Spring Security
4. **XSS Protection**: Thymeleaf automatically escapes HTML
5. **File Upload**: Validate file types and sizes

---

For more information, see [README.md](README.md) and [ARCHITECTURE.md](ARCHITECTURE.md).

