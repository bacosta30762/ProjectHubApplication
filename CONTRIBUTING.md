# Contributing Guide

Thank you for your interest in contributing to this project! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Issue Reporting](#issue-reporting)

---

## Code of Conduct

### Our Pledge

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Accept criticism gracefully

---

## Getting Started

1. **Fork the Repository**
   ```bash
   # Click "Fork" on GitHub, then clone your fork
   git clone https://github.com/your-username/projecthub.git
   cd projecthub
   ```

2. **Set Up Development Environment**
   - Follow [SETUP_GUIDE.md](SETUP_GUIDE.md)
   - Ensure all tests pass: `mvn test`
   - Run the application: `mvn spring-boot:run`

3. **Create a Branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   ```

---

## Development Workflow

### 1. Before You Start

- Check existing issues and pull requests
- Discuss major changes in an issue first
- Ensure you understand the project structure

### 2. Making Changes

1. **Update Documentation**
   - Update README if adding features
   - Update API_DOCUMENTATION.md for new endpoints
   - Add code comments for complex logic

2. **Write Tests**
   - Add unit tests for new services
   - Add integration tests for new endpoints
   - Ensure all tests pass

3. **Follow Coding Standards**
   - See [BEST_PRACTICES.md](BEST_PRACTICES.md)
   - Use consistent naming conventions
   - Format code properly

### 3. Testing Your Changes

```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=UsuarioServiceTest

# Check code coverage (if configured)
mvn jacoco:report
```

---

## Coding Standards

### Java Style

- Follow Java naming conventions
- Use meaningful variable and method names
- Keep methods focused and small
- Add JavaDoc for public methods

### Spring Boot Conventions

- Use `@Service` for business logic
- Use `@Controller` for web endpoints
- Use `@Repository` for data access
- Use `@Transactional` appropriately

### Code Formatting

- Use 4 spaces for indentation
- Remove trailing whitespace
- Use consistent brace style
- Limit line length to 120 characters

### Example

```java
/**
 * Saves a user to the database.
 * 
 * @param usuario The user to save
 * @param crearRolUser Whether to create default role
 */
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
```

---

## Commit Guidelines

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples

```
feat(usuario): Add password reset functionality

Implement password reset via email link with token validation.
Includes email template and expiration handling.

Closes #123
```

```
fix(proyectos): Fix project deletion cascade issue

Projects were not being deleted properly when user was deleted.
Added proper cascade configuration.

Fixes #456
```

```
docs(readme): Update installation instructions

Add Windows-specific setup steps and troubleshooting section.
```

### Commit Best Practices

- Make small, focused commits
- Write clear commit messages
- One logical change per commit
- Test before committing

---

## Pull Request Process

### Before Submitting

1. **Update Your Branch**
   ```bash
   git checkout main
   git pull upstream main
   git checkout your-branch
   git rebase main
   ```

2. **Ensure Quality**
   - All tests pass
   - Code follows style guidelines
   - Documentation is updated
   - No merge conflicts

3. **Write a Good PR Description**
   - What changes were made
   - Why the changes were needed
   - How to test the changes
   - Screenshots (if UI changes)

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests pass locally
```

### Review Process

1. **Automated Checks**
   - CI/CD pipeline runs tests
   - Code quality checks
   - Build verification

2. **Code Review**
   - At least one approval required
   - Address review comments
   - Update PR as needed

3. **Merge**
   - Squash and merge (preferred)
   - Delete branch after merge

---

## Issue Reporting

### Before Creating an Issue

1. Search existing issues
2. Check if it's already fixed
3. Verify it's a real issue

### Issue Template

```markdown
**Description**
Clear description of the issue

**Steps to Reproduce**
1. Go to '...'
2. Click on '...'
3. See error

**Expected Behavior**
What should happen

**Actual Behavior**
What actually happens

**Environment**
- Java version: 
- Spring Boot version:
- Database:
- Browser (if applicable):

**Screenshots**
If applicable

**Additional Context**
Any other relevant information
```

### Issue Labels

- `bug`: Something isn't working
- `enhancement`: New feature request
- `documentation`: Documentation improvements
- `question`: Questions or discussions
- `help wanted`: Extra attention needed

---

## Feature Requests

When proposing a new feature:

1. **Open an Issue First**
   - Describe the feature
   - Explain the use case
   - Discuss implementation approach

2. **Get Feedback**
   - Wait for maintainer feedback
   - Discuss design decisions
   - Refine the proposal

3. **Implementation**
   - Follow development workflow
   - Write tests
   - Update documentation

---

## Documentation Contributions

Documentation improvements are always welcome!

### Areas to Improve

- Code examples
- Setup instructions
- API documentation
- Architecture diagrams
- Troubleshooting guides

### Documentation Standards

- Use clear, concise language
- Include code examples
- Add screenshots when helpful
- Keep formatting consistent

---

## Testing Contributions

### Writing Tests

```java
@ExtendWith(MockitoExtension.class)
class UsuarioServiceTest {
    
    @Mock
    private UsuarioDao usuarioDao;
    
    @InjectMocks
    private UsuarioServiceImpl usuarioService;
    
    @Test
    @DisplayName("Should save user with default role")
    void testSaveWithDefaultRole() {
        // Given
        Usuario usuario = new Usuario();
        usuario.setUsername("testuser");
        
        when(usuarioDao.save(any(Usuario.class))).thenReturn(usuario);
        
        // When
        usuarioService.save(usuario, true);
        
        // Then
        verify(usuarioDao).save(usuario);
        verify(rolDao).save(any(Rol.class));
    }
}
```

### Test Coverage

- Aim for >80% coverage
- Test happy paths
- Test error cases
- Test edge cases

---

## Questions?

- Open an issue with the `question` label
- Check existing documentation
- Review closed issues/PRs

---

## Recognition

Contributors will be:
- Listed in CONTRIBUTORS.md (if applicable)
- Credited in release notes
- Appreciated by the maintainers!

---

Thank you for contributing! 🎉

