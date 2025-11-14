# ProjectHub

A comprehensive Spring Boot web application for project management with user authentication, role-based access control, progress tracking, and reporting capabilities.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Project Structure](#project-structure)
- [API Documentation](#api-documentation)
- [Configuration](#configuration)
- [Security](#security)
- [Internationalization](#internationalization)
- [Best Practices](#best-practices)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project is a full-stack web application built with Spring Boot 3.1.3, featuring:

- **User Management**: Registration, authentication, and role-based access control
- **Project Management**: CRUD operations for project tracking
- **Contact System**: Email integration for contact forms
- **Reporting**: JasperReports integration for PDF, Excel, and CSV exports
- **Multi-language Support**: Internationalization (i18n) with English and Spanish
- **Security**: Spring Security with BCrypt password encoding

## ✨ Features

- ✅ User registration with email activation
- ✅ Password recovery system
- ✅ Role-based access control (ADMIN, USER)
- ✅ Project management with progress tracking
- ✅ Contact form with email notifications
- ✅ Report generation (PDF, Excel, CSV)
- ✅ Multi-language support (English/Spanish)
- ✅ Image upload for user profiles
- ✅ Responsive UI with Bootstrap 5

## 🛠 Technology Stack

### Backend
- **Java 19**
- **Spring Boot 3.1.3**
- **Spring Security 6** - Authentication and authorization
- **Spring Data JPA** - Database abstraction
- **Hibernate** - ORM framework
- **MySQL** - Database
- **JasperReports 6.20.6** - Report generation
- **Lombok** - Boilerplate code reduction

### Frontend
- **Thymeleaf** - Server-side templating
- **Bootstrap 5.3.2** - UI framework
- **jQuery 3.7.1** - JavaScript library
- **Font Awesome 6.4.2** - Icons

### Other
- **JavaMail** - Email functionality
- **Maven** - Build tool

## 📦 Prerequisites

Before you begin, ensure you have the following installed:

- **Java Development Kit (JDK) 19** or higher
- **Maven 3.6+**
- **MySQL 8.0+**
- **IDE** (IntelliJ IDEA, Eclipse, or VS Code recommended)
- **Git** (for version control)

## 🚀 Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/projecthub.git
cd projecthub
```

### 2. Database Setup

1. Open MySQL and run the provided SQL script:

```bash
mysql -u root -p < BaseDatos_ProyectoFinal.sql
```

Or manually execute the SQL commands in `BaseDatos_ProyectoFinal.sql`:

```sql
CREATE SCHEMA projecthub;
USE projecthub;

CREATE USER 'admin01'@'%' IDENTIFIED BY 'admin01';
GRANT ALL PRIVILEGES ON projecthub.* TO 'admin01'@'%';

CREATE TABLE proyectos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre varchar(50),
    descripcion VARCHAR(100),
    progreso int,
    comentarios VARCHAR(100),
    activo bool
) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8mb4;
```

2. Create additional tables for `usuario` and `rol`:

```sql
CREATE TABLE usuario (
    id_usuario BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    nombre VARCHAR(100),
    apellidos VARCHAR(100),
    correo VARCHAR(100) UNIQUE,
    telefono VARCHAR(20),
    rutaImagen VARCHAR(255),
    activo BOOLEAN DEFAULT FALSE
) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8mb4;

CREATE TABLE rol (
    id_rol BIGINT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    id_usuario BIGINT NOT NULL,
    FOREIGN KEY (id_usuario) REFERENCES usuario(id_usuario)
) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8mb4;
```

### 3. Configure Application Properties

Edit `ProyectoFinal/src/main/resources/application.properties`:

```properties
# Server Configuration
server.port=80

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/projecthub?useTimezone=true&serverTimezone=UTC
spring.datasource.username=admin01
spring.datasource.password=admin01
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect

# Email Configuration (Gmail example)
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

# Server URL for email links
servidor.http=localhost
```

**Important**: For Gmail, you'll need to:
1. Enable 2-Factor Authentication
2. Generate an App Password (not your regular password)
3. Use the App Password in `spring.mail.password`

### 4. Build and Run

#### Using Maven:

```bash
cd ProyectoFinal
mvn clean install
mvn spring-boot:run
```

#### Using IDE:

1. Import the project as a Maven project
2. Run `TrabajoFinalApplication.java` (or `ProjectHubApplication.java` if renamed)
3. The application will start on `http://localhost`

### 5. Access the Application

- **Home**: http://localhost/
- **Login**: http://localhost/login
- **Registration**: http://localhost/registro/nuevo

## 📁 Project Structure

```
ProyectoFinal/
├── src/
│   ├── main/
│   │   ├── java/com/TrabajoFinal/
│   │   │   ├── controller/          # REST/Web controllers
│   │   │   │   ├── ContactoController.java
│   │   │   │   ├── DesarrolloSoftware.java
│   │   │   │   ├── EcommerceController.java
│   │   │   │   ├── ProyectosController.java
│   │   │   │   ├── RegistroController.java
│   │   │   │   ├── ReporteController.java
│   │   │   │   ├── ServiciosController.java
│   │   │   │   ├── UbicacionController.java
│   │   │   │   └── UsuarioController.java
│   │   │   ├── dao/                  # Data Access Objects
│   │   │   │   ├── ProyectosDao.java
│   │   │   │   ├── RolDao.java
│   │   │   │   └── UsuarioDao.java
│   │   │   ├── domain/               # Entity models
│   │   │   │   ├── Contacto.java
│   │   │   │   ├── Proyectos.java
│   │   │   │   ├── Rol.java
│   │   │   │   └── Usuario.java
│   │   │   ├── service/              # Business logic interfaces
│   │   │   │   ├── CorreoService.java
│   │   │   │   ├── ProyectosService.java
│   │   │   │   ├── RegistroService.java
│   │   │   │   ├── ReporteService.java
│   │   │   │   ├── UsuarioService.java
│   │   │   │   └── UsuarioDetailsService.java
│   │   │   ├── service/impl/         # Service implementations
│   │   │   │   ├── CorreoServiceImpl.java
│   │   │   │   ├── ProyectosServiceImpl.java
│   │   │   │   ├── RegistroServiceImpl.java
│   │   │   │   ├── ReporteServiceImpl.java
│   │   │   │   └── UsuarioServiceImpl.java
│   │   │   ├── ProjectConfig.java    # Configuration class
│   │   │   └── TrabajoFinalApplication.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── messages.properties    # i18n messages
│   │       ├── messages_es.properties
│   │       ├── messages_en.properties
│   │       ├── reportes/              # JasperReports templates
│   │       ├── static/                # CSS, JS, images
│   │       └── templates/             # Thymeleaf templates
│   └── test/
└── pom.xml
```

## 📚 API Documentation

### Public Endpoints (No Authentication Required)

#### Contact Controller
- `GET /contacto/listado` - Display contact form page
- `POST /contacto/enviar-correo` - Submit contact form

#### Service Pages
- `GET /servicios/listado` - Services page
- `GET /desarrolloSoftware/listado` - Software development page
- `GET /ecommerce/listado` - E-commerce page
- `GET /ubicacion/listado` - Location page

#### Registration
- `GET /registro/nuevo` - Registration form
- `POST /registro/crearUsuario` - Create new user
- `GET /registro/recordar` - Password recovery form
- `POST /registro/recordarUsuario` - Request password recovery
- `GET /registro/activacion/{usuario}/{id}` - Activate user account
- `POST /registro/activar` - Complete activation with image upload

### Protected Endpoints (Authentication Required)

#### Projects Controller
- `GET /proyectos/listado` - List all projects (ADMIN, USER)
- `GET /proyectos/nuevo` - Create new project form (ADMIN, USER)
- `POST /proyectos/guardar` - Save project (ADMIN, USER)
- `GET /proyectos/modificar/{id}` - Edit project form (ADMIN, USER)
- `GET /proyectos/eliminar/{id}` - Delete project (ADMIN, USER)

#### User Controller
- `GET /usuario/listado` - List all users (ADMIN only)
- `GET /usuario/nuevo` - Create new user form (ADMIN, USER)
- `POST /usuario/guardar` - Save user (ADMIN, USER)
- `GET /usuario/modificar/{idUsuario}` - Edit user form (ADMIN, USER)
- `GET /usuario/eliminar/{idUsuario}` - Delete user (ADMIN, USER)

#### Reports Controller
- `GET /reportes/plantillareportes` - Reports page (ADMIN, USER)
- `GET /reportes/usuarios?tipo={Pdf|Xls|Csv|vPdf}` - Generate users report (ADMIN, USER)
- `GET /reportes/proyectos?tipo={Pdf|Xls|Csv|vPdf}` - Generate projects report (ADMIN, USER)

For detailed API documentation with request/response examples, see [API_DOCUMENTATION.md](API_DOCUMENTATION.md).

## ⚙️ Configuration

### Security Configuration

The security is configured in `ProjectConfig.java`:

- **Public Routes**: `/`, `/index`, `/servicios/**`, `/contacto/**`, `/ubicacion/**`, `/desarrolloSoftware/**`, `/ecommerce/**`, `/registro/**`
- **User Routes**: `/proyectos/**`, `/usuario/nuevo`, `/usuario/guardar`, `/usuario/modificar/**`, `/reportes/**`
- **Admin Routes**: `/usuario/listado`, `/usuario/eliminar/**`

### Internationalization

The application supports multiple languages:
- Change language via URL parameter: `?lang=en` or `?lang=es`
- Messages are stored in `messages.properties`, `messages_en.properties`, `messages_es.properties`

## 🔒 Security

### Password Encoding
- Passwords are encoded using BCrypt
- Default role `ROLE_USER` is assigned to new users
- Admin users must be manually assigned `ROLE_ADMIN`

### User Activation Flow
1. User registers → receives email with activation link
2. User clicks link → account activation page
3. User uploads profile image and sets password → account activated

## 🌍 Internationalization

The application supports English and Spanish. To add a new language:

1. Create `messages_{locale}.properties` file
2. Add translations for all keys
3. The locale resolver will automatically detect the language

## 📖 Best Practices

### Code Organization
- **Controllers**: Handle HTTP requests/responses only
- **Services**: Contain business logic
- **DAOs**: Data access layer using Spring Data JPA
- **Domain**: Entity models with JPA annotations

### Security Best Practices
1. Never commit passwords or API keys to version control
2. Use environment variables for sensitive configuration
3. Always validate user input
4. Use parameterized queries (handled by JPA)
5. Implement CSRF protection (enabled by default in Spring Security)

### Development Best Practices
1. Use `@Transactional` for database operations
2. Use `@Slf4j` for logging
3. Follow RESTful naming conventions
4. Validate input at controller level
5. Handle exceptions appropriately

## 🧪 Testing

Run tests with:

```bash
mvn test
```

## 📝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is open source and available under the MIT License. See the repository for license details.

## 👤 Author

**Brandon Acosta Cascante**

## 🙏 Acknowledgments

- Spring Boot team for the excellent framework
- Bootstrap team for the UI framework
- All open-source contributors

---

For detailed API documentation, see [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
For architecture details, see [ARCHITECTURE.md](ARCHITECTURE.md)

