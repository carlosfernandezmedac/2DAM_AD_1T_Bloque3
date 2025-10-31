# Tema 8: Exploración del Mapeo Objeto–Relacional (Hibernate + MySQL)

---

## 1️⃣ Introducción
En esta unidad exploraremos el funcionamiento interno de **Hibernate**, centrándonos en las **clases persistentes**, los **ficheros de mapeo XML**, las **sesiones**, los **métodos de persistencia** y las **consultas HQL**.

📘 **Objetivos del tema:**
- Entender qué es una **clase persistente** y sus reglas.
- Aprender la estructura del **fichero de mapeo (.hbm.xml)**.
- Conocer los **estados y métodos** del objeto `Session`.
- Practicar **inserciones, actualizaciones y consultas HQL**.
- Comprender la **gestión de transacciones en Hibernate**.

---

## 2️⃣ Clases Persistentes

Una **clase persistente** es una clase Java cuyos objetos pueden almacenarse en una base de datos relacional. Hibernate se encarga de convertir las instancias de estas clases en registros de tablas.

🧩 **Reglas básicas para una clase persistente:**
- Debe tener un **constructor por defecto**.
- Debe incluir un **atributo `id`** como clave primaria.
- Todos los atributos deben ser **privados** y tener **getters/setters**.
- No debe ser una clase `final`.

📄 **Ejemplo:**
```java
package com.empresa.model;

public class Customer {
    private int id;
    private String firstName;
    private String lastName;
    private int customerNumber;

    public Customer() {}

    public Customer(String fname, String lname, int custNum) {
        this.firstName = fname;
        this.lastName = lname;
        this.customerNumber = custNum;
    }

    // Getters y Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public int getCustomerNumber() { return customerNumber; }
    public void setCustomerNumber(int customerNumber) { this.customerNumber = customerNumber; }
}
```

---

## 3️⃣ Fichero de Mapeo I – Composición

Hibernate puede usar **anotaciones JPA** o **ficheros XML** (`.hbm.xml`) para definir cómo se mapean las clases y sus atributos a la base de datos.

📄 **Ejemplo de mapeo XML:**
```xml
<hibernate-mapping>
   <class name="com.empresa.model.Customer" table="CUSTOMER">

      <meta attribute="class-description">
         Clase que almacena información de clientes.
      </meta>

      <id name="id" type="int" column="id">
         <generator class="native"/>
      </id>

      <property name="firstName" column="first_name" type="string"/>
      <property name="lastName" column="last_name" type="string"/>
      <property name="customerNumber" column="customer_number" type="int"/>
   </class>
</hibernate-mapping>
```
📁 Archivo: `Customer.hbm.xml`

---

## 4️⃣ Fichero de Mapeo II – Elementos Clave

| Elemento | Descripción |
|-----------|-------------|
| `<hibernate-mapping>` | Raíz del documento. Contiene las clases mapeadas. |
| `<class>` | Define la relación entre una clase Java y una tabla SQL. |
| `<meta>` | Información adicional opcional. |
| `<id>` | Clave primaria. Contiene el generador automático. |
| `<generator>` | Define la estrategia de generación de la PK (`native`, `identity`, `sequence`). |
| `<property>` | Mapea atributos de la clase a columnas SQL. |

---

## 5️⃣ Caso Práctico 1 – Clase Persistente “Vehículo”

🧩 **Requisitos:** Crear una clase persistente `Vehiculo` con los campos:
- `marca`, `motor`, `numeroRuedas`, `numeroKilometros` y un `id` autoincrementable.

📄 **Código:**
```java
public class Vehiculo {
    private int id;
    private String marca;
    private String motor;
    private int numeroRuedas;
    private int numeroKilometros;

    public Vehiculo() {}

    public Vehiculo(String marca, String motor, int numeroRuedas, int numeroKilometros) {
        this.marca = marca;
        this.motor = motor;
        this.numeroRuedas = numeroRuedas;
        this.numeroKilometros = numeroKilometros;
    }

    // Getters y setters omitidos por brevedad
}
```

📄 **Mapeo XML (Vehiculo.hbm.xml):**
```xml
<hibernate-mapping>
   <class name="Vehiculo" table="VEHICULO">
      <id name="id" type="int" column="id">
         <generator class="native"/>
      </id>
      <property name="marca" column="marca" type="string"/>
      <property name="motor" column="motor" type="string"/>
      <property name="numeroRuedas" column="num_ruedas" type="int"/>
      <property name="numeroKilometros" column="km" type="int"/>
   </class>
</hibernate-mapping>
```

---

## 6️⃣ Sesiones y Objetos Hibernate I – Estados

Hibernate utiliza el objeto **`Session`** para interactuar con la base de datos.

| Estado | Descripción |
|---------|-------------|
| **Transient** | El objeto aún no se ha guardado. |
| **Persistent** | El objeto está asociado a una sesión y a una fila de base de datos. |
| **Detached** | La sesión se ha cerrado y el objeto ya no está sincronizado. |

📄 **Ejemplo de sesión y transacción:**
```java
SessionFactory sessionFactory;
Session session = sessionFactory.openSession();
Transaction transaction = null;

try {
    transaction = session.beginTransaction();
    // operaciones de persistencia
    transaction.commit();
} catch (Exception e) {
    if (transaction != null) transaction.rollback();
    e.printStackTrace();
} finally {
    session.close();
}
```

---

## 7️⃣ Sesiones y Objetos Hibernate II – Métodos Importantes

| Método | Descripción |
|---------|-------------|
| `beginTransaction()` | Inicia una transacción. |
| `close()` | Cierra la sesión. |
| `clear()` | Limpia la caché de sesión. |
| `createQuery()` | Crea una consulta HQL. |
| `get()` | Recupera un objeto por ID (puede devolver `null`). |
| `load()` | Recupera un objeto o lanza excepción si no existe. |
| `save()` | Inserta un nuevo registro y devuelve el ID. |
| `update()` | Actualiza un registro existente. |
| `merge()` | Actualiza, sin importar si la sesión existe. |
| `delete()` | Elimina un registro. |

---

## 8️⃣ Carga, Almacenamiento y Modificación de Objetos

🗄️ **Carga:**
```java
Cliente c = session.get(Cliente.class, 1);
```

💾 **Guardar:**
```java
session.save(new Cliente("Ana", "López", 200));
```

✏️ **Actualizar:**
```java
cliente.setNombre("Ana María");
session.update(cliente);
```

🧹 **Eliminar:**
```java
session.delete(cliente);
```

✅ **Guardar o Actualizar automáticamente:**
```java
session.saveOrUpdate(cliente);
```

---

## 9️⃣ Consultas HQL (Hibernate Query Language)

HQL es un lenguaje similar a SQL pero orientado a objetos.

📄 **Ejemplo básico:**
```java
String hql = "FROM Customer";
Query consulta = session.createQuery(hql);
consulta.setFirstResult(0);
consulta.setMaxResults(40);
List results = consulta.list();
```

📄 **Ejemplo de Update con parámetros:**
```java
Query q = session.createQuery(
   "update Customer set customerNumber=:num where id=:id and firstName=:name");
q.setParameter("num", 25);
q.setParameter("id", 105);
q.setParameter("name", "Pepe");
int status = q.executeUpdate();
transaction.commit();
```

🔹 También se pueden usar consultas SQL nativas con `createSQLQuery()`.

---

## 🔟 Gestión de Transacciones con Hibernate

Una **transacción** representa una unidad de trabajo atómica: si algo falla, se revierte todo.

🧠 **Principio ACID:** Atomicidad, Consistencia, Aislamiento y Durabilidad.

📄 **Métodos importantes de `Transaction`:**
| Método | Descripción |
|---------|-------------|
| `begin()` | Inicia una transacción. |
| `commit()` | Confirma la transacción. |
| `rollback()` | Cancela la transacción. |
| `isActive()` | Comprueba si sigue activa. |
| `setTimeout(int)` | Define un tiempo máximo. |

---

## 1️⃣1️⃣ Caso Práctico 2 – Sentencias HQL

📄 **Planteamiento:**
Actualizar el campo `customerNumber` de la entidad `Customer` a `25` cuando `id=105` y `firstName='Pepe'`.

📄 **Solución:**
```java
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();
Query q = session.createQuery(
   "update Customer set customerNumber=:num where id=:id and firstName=:name");
q.setParameter("num", 25);
q.setParameter("id", 105);
q.setParameter("name", "Pepe");
int status = q.executeUpdate();
transaction.commit();

if (status > 0)
    System.out.println("✅ Update realizado");
else
    System.out.println("⚠️ Update no realizado");
```

---

## 1️⃣2️⃣ Resumen del Tema

| Concepto | Descripción |
|-----------|-------------|
| **Clase persistente** | Objeto Java almacenable en BD. |
| **Fichero .hbm.xml** | Define relaciones entre clases y tablas. |
| **Session** | Administra conexión y operaciones. |
| **Transaction** | Controla unidades de trabajo atómicas. |
| **HQL** | Lenguaje orientado a objetos para consultas. |
| **Persistencia** | Guardar, actualizar o borrar objetos de BD. |

---

## 🔗 Webgrafía
- [Hibernate.org](https://hibernate.org/)
- [Oracle Java Docs](https://www.oracle.com/es/java/)
- [Spring Boot JPA Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/data.html)
- [MVN Repository – Hibernate Core](https://mvnrepository.com/artifact/org.hibernate/hibernate-core)

---

> ✨ **Conclusión:** En esta unidad hemos aprendido cómo Hibernate gestiona las clases persistentes, cómo se definen los ficheros de mapeo, cómo se usan las sesiones, y cómo aplicar consultas HQL para manipular datos de manera eficiente en MySQL.
