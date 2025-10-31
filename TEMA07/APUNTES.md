# Tema 7: Mapeo Objeto–Relacional (Hibernate + MySQL)

---

## 1️⃣ Introducción
En esta unidad aprenderás a conectar tu aplicación Java con una base de datos relacional usando el framework **Hibernate**, el ORM (Object Relational Mapping) más popular en Java.

📘 **Objetivos del tema:**
- Comprender el concepto de **Mapeo Objeto–Relacional (ORM)**.
- Conocer las **fases del mapeo** y los principales frameworks ORM.
- Aprender a **instalar y configurar Hibernate** en un proyecto Spring Boot.
- Crear una **app web con MySQL** que persista entidades Java.
- Configurar **logs** para monitorizar las operaciones SQL de Hibernate.

---

## 2️⃣ Qué es el Mapeo Objeto–Relacional (ORM)

Java trabaja con **objetos**, mientras que las bases de datos relacionales trabajan con **tablas**. El ORM sirve para **traducir entre ambos mundos**, evitando tener que escribir SQL manual.

🧩 **Ejemplo básico:**
Un objeto Java de tipo `Cliente`:
```java
class Cliente {
    private int id;
    private String nombre;
    private String email;
}
```
Se mapea con una tabla SQL:
```sql
CREATE TABLE clientes (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100),
  email VARCHAR(100)
);
```
Hibernate realiza automáticamente este mapeo.

---

## 3️⃣ Ventajas e Inconvenientes del ORM

| ✅ Ventajas | ⚠️ Inconvenientes |
|-------------|-------------------|
| Abstracción del SQL nativo | Consumo de recursos |
| Mayor productividad | Complejidad en consultas muy avanzadas |
| Orientación a objetos total | Curva de aprendizaje inicial |
| Gestión automática de transacciones | Logs más extensos |

---

## 4️⃣ Fases del Mapeo Objeto–Relacional

1. **Fase 1: Objetos:** Clases Java (POJOs) que representan entidades.  
2. **Fase 2: Persistencia:** Hibernate traduce objetos a registros SQL.  
3. **Fase 3: Relacional:** Los datos se almacenan realmente en MySQL.

📊 Hibernate usa internamente caché, sesiones, transacciones y consultas HQL (Hibernate Query Language) para optimizar el proceso.

---

## 5️⃣ Herramientas ORM más Usadas en Java

| Framework | Características |
|------------|----------------|
| **Ebean** | Consultas SQL y DTO; caché L2 y soporte para múltiples BBDD. |
| **MyBatis (iBatis)** | Control manual de consultas SQL, ideal para proyectos con SQL complejo. |
| **Hibernate** | Framework estándar JPA; potente, flexible y el más usado. |

En este tema trabajaremos con **Hibernate**, el ORM por excelencia.

---

## 6️⃣ Arquitectura y Componentes de Hibernate

![alt text](img/arquitectura.png)


🧠 **Componentes principales:**
- **SessionFactory:** Crea sesiones para comunicarse con la base de datos.
- **Session:** Representa la conexión activa a la base de datos.
- **Transaction:** Gestiona las operaciones atómicas (commit/rollback).
- **Query / HQL:** Lenguaje orientado a objetos para consultas.
- **Criteria:** API para consultas dinámicas sin SQL.

🗺️ **Esquema general:**
```
Aplicación Java → Hibernate ORM → Driver JDBC → MySQL
```

---

## 7️⃣ Instalación del Proyecto Spring Boot + Hibernate + MySQL

### 🧩 Paso 1: Crear Proyecto en Spring Initializr

1. Entra en 👉 [https://start.spring.io](https://start.spring.io)
2. Configura:
   - **Project:** Maven
   - **Language:** Java
   - **Spring Boot:** versión estable
   - **Packaging:** jar
   - **Java:** 17 o superior
3. Añade dependencias:
   - `Spring Web`
   - `Spring Data JPA`
   - `MySQL Driver`
4. Pulsa **Generate** y descomprime el proyecto.

---

### 🧩 Paso 2: Dependencias en `pom.xml`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

---

### 🧩 Paso 3: Configuración de Hibernate en `application.properties`

📁 `src/main/resources/application.properties`

```properties
# Conexión a MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/empresa_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=tu_contraseña
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Dialecto e inicialización
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Logs de Hibernate
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.type.descriptor.sql=trace
```

✅ Esto permite a Hibernate crear y actualizar automáticamente las tablas según las entidades.

---

## 8️⃣ Caso Práctico: Mini App con Hibernate

Vamos a construir una pequeña app de gestión de clientes y pedidos 👇

### 🧩 Paso 1: Crear la Entidad `Cliente`

📁 `src/main/java/com/empresa/model/Cliente.java`

```java
package com.empresa.model;

import jakarta.persistence.*;

@Entity
@Table(name = "clientes")
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String nombre;

    private String email;

    // Getters y setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

### 🧩 Paso 2: Crear el Repositorio JPA

📁 `src/main/java/com/empresa/repository/ClienteRepository.java`

```java
package com.empresa.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.empresa.model.Cliente;

public interface ClienteRepository extends JpaRepository<Cliente, Long> {
}
```

### 🧩 Paso 3: Controlador REST

📁 `src/main/java/com/empresa/controller/ClienteController.java`

```java
package com.empresa.controller;

import com.empresa.model.Cliente;
import com.empresa.repository.ClienteRepository;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/clientes")
public class ClienteController {

    private final ClienteRepository repo;

    public ClienteController(ClienteRepository repo) {
        this.repo = repo;
    }

    @GetMapping
    public List<Cliente> listar() {
        return repo.findAll();
    }

    @PostMapping
    public Cliente crear(@RequestBody Cliente cliente) {
        return repo.save(cliente);
    }
}
```

🚀 **Ejecuta la aplicación** y prueba en Postman o navegador:

- GET 👉 `http://localhost:8080/clientes`
- POST 👉 `http://localhost:8080/clientes` con cuerpo JSON:
```json
{
  "nombre": "Ana Pérez",
  "email": "ana@empresa.com"
}
```

---

## 9️⃣ Caso Práctico: Configurar Logs de Hibernate

Si DevOps solicita ver las consultas SQL que ejecuta Hibernate, puedes activar los logs:

```properties
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.type.descriptor.sql=trace
```

Así verás en consola las consultas SQL exactas:
```sql
Hibernate: insert into clientes (email, nombre) values (?, ?)
Hibernate: select * from clientes
```

---

## 🔟 Resumen del Tema

| Concepto | Descripción |
|-----------|-------------|
| **ORM** | Traduce objetos Java a tablas SQL. |
| **Hibernate** | Framework JPA que facilita la persistencia. |
| **SessionFactory / Session** | Gestionan la conexión con la base de datos. |
| **`application.properties`** | Archivo de configuración de conexión y logs. |
| **HQL / Criteria** | Alternativas orientadas a objetos al SQL clásico. |
| **Spring Boot + Hibernate** | Combinación ideal para apps Java modernas. |

---

## 🔗 Webgrafía
- [Spring Initializr](https://start.spring.io/)
- [Hibernate.org](https://hibernate.org/)
- [MVN Repository – Hibernate](https://mvnrepository.com/artifact/org.hibernate/hibernate-core)
- [MySQL Connector/J Docs](https://dev.mysql.com/doc/connector-j/en/)
- [Spring Boot Data JPA Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/data.html)

---

> ✨ **Conclusión:** Hibernate simplifica la comunicación entre Java y MySQL, automatizando las consultas, relaciones y transacciones, permitiendo desarrollar aplicaciones limpias, robustas y orientadas a objetos.

