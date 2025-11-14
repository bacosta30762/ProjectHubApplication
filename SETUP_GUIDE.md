# Setup Guide

Complete step-by-step guide to set up and run the project.

## Prerequisites

### Required Software

1. **Java Development Kit (JDK) 19+**
   - Download: https://adoptium.net/
   - Verify: `java -version`

2. **Maven 3.6+**
   - Download: https://maven.apache.org/download.cgi
   - Verify: `mvn -version`

3. **MySQL 8.0+**
   - Download: https://dev.mysql.com/downloads/mysql/
   - Verify: `mysql --version`

4. **IDE (Optional but Recommended)**
   - IntelliJ IDEA: https://www.jetbrains.com/idea/
   - Eclipse: https://www.eclipse.org/
   - VS Code: https://code.visualstudio.com/

5. **Git**
   - Download: https://git-scm.com/downloads
   - Verify: `git --version`

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/yourusername/Trabajo-Final-Aplicaciones-Web.git
cd Trabajo-Final-Aplicaciones-Web
```

---

## Step 2: Database Setup

### 2.1 Create Database Schema

Open MySQL command line or MySQL Workbench:

```bash
mysql -u root -p
```

Execute the SQL script:

```sql
-- Create database
CREATE SCHEMA IF NOT EXISTS TrabajoFinal;
USE TrabajoFinal;

-- Create user
CREATE USER IF NOT EXISTS 'admin01'@'%' IDENTIFIED BY 'admin01';
GRANT ALL PRIVILEGES ON TrabajoFinal.* TO 'admin01'@'%';
FLUSH PRIVILEGES;

-- Create proyectos table
CREATE TABLE IF NOT EXISTS proyectos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    descripcion VARCHAR(100),
    progreso INT,
    comentarios VARCHAR(100),
    activo BOOLEAN DEFAULT TRUE
) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8mb4;

-- Create usuario table
CREATE TABLE IF NOT EXISTS usuario (
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

-- Create rol table
CREATE TABLE IF NOT EXISTS rol (
    id_rol BIGINT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    id_usuario BIGINT NOT NULL,
    FOREIGN KEY (id_usuario) REFERENCES usuario(id_usuario) ON DELETE CASCADE
) ENGINE = InnoDB DEFAULT CHARACTER SET = utf8mb4;

-- Insert sample data (optional)
INSERT INTO proyectos(nombre, descripcion, progreso, comentarios, activo) VALUES 
('Proyecto 1', 'Descripción del proyecto 1', 40, 'Comentarios del proyecto 1', TRUE),
('Proyecto 2', 'Descripción del proyecto 2', 60, 'Comentarios del proyecto 2', TRUE),
('Proyecto 3', 'Descripción del proyecto 3', 80, 'Comentarios del proyecto 3', TRUE);
```

Or use the provided SQL file:

```bash
mysql -u root -p < BaseDatos_ProyectoFinal.sql
```

**Note**: You may need to manually create the `usuario` and `rol` tables if they're not in the SQL file.

---

## Step 3: Configure Application Properties

Edit `ProyectoFinal/src/main/resources/application.properties`:

### 3.1 Database Configuration

```properties
# Database connection
spring.datasource.url=jdbc:mysql://localhost:3306/trabajofinal?useTimezone=true&serverTimezone=UTC
spring.datasource.username=admin01
spring.datasource.password=admin01
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect

# JPA/Hibernate settings
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

**Important**: 
- Change `username` and `password` if you used different credentials
- Use `ddl-auto=validate` in production (not `create` or `update`)

### 3.2 Server Configuration

```properties
# Server port
server.port=80

# Logging
logging.pattern.dateformat=hh:mm
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.type.descriptor.sql.BasicBinder=TRACE
spring.main.banner-mode=off
```

**Note**: Port 80 requires administrator privileges. Use port 8080 for development:

```properties
server.port=8080
```

### 3.3 Email Configuration (Gmail Example)

```properties
# Email settings
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

# Server URL for email links
servidor.http=localhost
```

#### Setting Up Gmail App Password

1. Go to your Google Account: https://myaccount.google.com/
2. Enable 2-Step Verification
3. Go to App Passwords: https://myaccount.google.com/apppasswords
4. Generate a new app password for "Mail"
5. Use the generated password in `spring.mail.password`

**Security Note**: Never commit real passwords to version control!

### 3.4 Thymeleaf Configuration

```properties
# Disable cache in development
spring.thymeleaf.cache=false
```

Set to `true` in production for better performance.

---

## Step 4: Build the Project

### 4.1 Using Maven Command Line

```bash
cd ProyectoFinal
mvn clean install
```

This will:
- Download dependencies
- Compile source code
- Run tests
- Package the application

### 4.2 Using IDE

**IntelliJ IDEA**:
1. File → Open → Select `pom.xml`
2. Wait for Maven to import dependencies
3. Build → Build Project

**Eclipse**:
1. File → Import → Maven → Existing Maven Projects
2. Select the project directory
3. Right-click project → Run As → Maven install

---

## Step 5: Run the Application

### 5.1 Using Maven

```bash
cd ProyectoFinal
mvn spring-boot:run
```

### 5.2 Using IDE

**IntelliJ IDEA**:
1. Open `TrabajoFinalApplication.java`
2. Right-click → Run 'TrabajoFinalApplication'

**Eclipse**:
1. Right-click project → Run As → Spring Boot App

### 5.3 Using JAR File

```bash
cd ProyectoFinal/target
java -jar TrabajoFinalAplicaciones-1.jar
```

---

## Step 6: Verify Installation

1. **Check Application Logs**:
   - Look for "Started TrabajoFinalApplication"
   - No database connection errors
   - No port binding errors

2. **Access the Application**:
   - Home: http://localhost/ (or http://localhost:8080/)
   - Login: http://localhost/login
   - Registration: http://localhost/registro/nuevo

3. **Test Database Connection**:
   - Try to register a new user
   - Check MySQL for new records

---

## Step 7: Create First Admin User

### Option 1: Via Database

```sql
USE TrabajoFinal;

-- Insert admin user (password: admin123 - BCrypt encoded)
INSERT INTO usuario (username, password, nombre, apellidos, correo, telefono, activo) 
VALUES ('admin', '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy', 
        'Admin', 'User', 'admin@example.com', '1234567890', TRUE);

-- Insert admin role
INSERT INTO rol (nombre, id_usuario) 
VALUES ('ROLE_ADMIN', LAST_INSERT_ID());

-- Also add USER role
INSERT INTO rol (nombre, id_usuario) 
VALUES ('ROLE_USER', (SELECT id_usuario FROM usuario WHERE username = 'admin'));
```

### Option 2: Via Registration Flow

1. Register a new user at `/registro/nuevo`
2. Check email for activation link
3. Complete activation
4. Manually add `ROLE_ADMIN` role in database:

```sql
INSERT INTO rol (nombre, id_usuario) 
VALUES ('ROLE_ADMIN', (SELECT id_usuario FROM usuario WHERE username = 'your-username'));
```

---

## Troubleshooting

### Common Issues

#### 1. Port Already in Use

**Error**: `Port 80 is already in use`

**Solution**:
- Change port in `application.properties` to 8080
- Or stop the service using port 80

#### 2. Database Connection Failed

**Error**: `Communications link failure`

**Solutions**:
- Verify MySQL is running: `mysql -u root -p`
- Check database name, username, password
- Ensure MySQL allows connections from localhost
- Check firewall settings

#### 3. Email Not Sending

**Error**: `Authentication failed`

**Solutions**:
- Verify Gmail app password (not regular password)
- Check 2-Step Verification is enabled
- Verify SMTP settings
- Check firewall/antivirus blocking port 587

#### 4. Maven Dependencies Not Downloading

**Error**: `Could not resolve dependencies`

**Solutions**:
- Check internet connection
- Clear Maven cache: `mvn clean`
- Update Maven: `mvn -U clean install`
- Check Maven settings.xml for proxy configuration

#### 5. Thymeleaf Template Not Found

**Error**: `Template not found`

**Solutions**:
- Verify templates are in `src/main/resources/templates/`
- Check template path in controller
- Ensure Thymeleaf dependency is in `pom.xml`

#### 6. ClassNotFoundException

**Error**: `ClassNotFoundException: com.mysql.cj.jdbc.Driver`

**Solutions**:
- Verify MySQL connector dependency in `pom.xml`
- Run `mvn clean install` to download dependencies
- Check IDE project settings

---

## Development Setup

### Recommended IDE Plugins

**IntelliJ IDEA**:
- Lombok Plugin
- Spring Boot Plugin
- Database Navigator

**Eclipse**:
- Spring Tools 4
- Lombok (install manually)
- Maven Integration

**VS Code**:
- Java Extension Pack
- Spring Boot Extension Pack
- MySQL Extension

### Development Best Practices

1. **Use Git**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

2. **Create .gitignore**:
   ```
   target/
   .idea/
   *.iml
   .classpath
   .project
   .settings/
   ```

3. **Environment Variables**:
   - Use `.env` file for sensitive data (not committed)
   - Or use Spring profiles (`application-dev.properties`)

4. **Hot Reload**:
   - Spring Boot DevTools enables automatic restart
   - Changes to Java files require restart
   - Changes to templates/resources reload automatically

---

## Production Deployment

### Checklist

- [ ] Change all default passwords
- [ ] Use environment variables for secrets
- [ ] Enable HTTPS
- [ ] Set `spring.thymeleaf.cache=true`
- [ ] Set `spring.jpa.show-sql=false`
- [ ] Configure production database
- [ ] Set up logging
- [ ] Configure backup strategy
- [ ] Set up monitoring
- [ ] Review security settings

### Environment Variables

```bash
export SPRING_DATASOURCE_URL=jdbc:mysql://prod-db:3306/trabajofinal
export SPRING_DATASOURCE_USERNAME=prod_user
export SPRING_DATASOURCE_PASSWORD=secure_password
export SPRING_MAIL_USERNAME=prod-email@example.com
export SPRING_MAIL_PASSWORD=app_password
```

---

## Next Steps

1. Read [README.md](README.md) for project overview
2. Review [API_DOCUMENTATION.md](API_DOCUMENTATION.md) for API details
3. Check [ARCHITECTURE.md](ARCHITECTURE.md) for design patterns
4. Start developing!

---

## Getting Help

- Check application logs for errors
- Review Spring Boot documentation: https://spring.io/projects/spring-boot
- Check MySQL documentation: https://dev.mysql.com/doc/
- Review project issues on GitHub

---

Happy coding! 🚀

