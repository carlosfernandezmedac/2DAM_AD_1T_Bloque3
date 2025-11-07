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
        this.lastLastName = lname;
        this.customerNumber = custNum;
    }
}
```

---

## 3️⃣ Fichero de Mapeo I – Composición

Hibernate puede usar **anotaciones JPA** o **ficheros XML** (`.hbm.xml`) para definir cómo se mapean las clases y sus atributos a la base de datos.

📄 **Ejemplo de mapeo XML:**
```xml
<hibernate-mapping>
   <class name="com.empresa.model.Customer" table="CUSTOMER">
      <id name="id" type="int" column="id">
         <generator class="native"/>
      </id>
      <property name="firstName" column="first_name" type="string"/>
      <property name="lastName" column="last_name" type="string"/>
      <property name="customerNumber" column="customer_number" type="int"/>
   </class>
</hibernate-mapping>
```

---

## 4️⃣ Fichero de Mapeo II – Elementos Clave

| Elemento | Descripción |
|-----------|-------------|
| `<hibernate-mapping>` | Raíz del documento. Contiene las clases mapeadas. |
| `<class>` | Define la relación entre una clase Java y una tabla SQL. |
| `<meta>` | Información adicional opcional. |
| `<id>` | Clave primaria. |
| `<generator>` | Estrategia de generación (`native`, `identity`, `sequence`). |
| `<property>` | Mapea atributos de clase a columnas SQL. |

---

## 6️⃣ Sesiones y Objetos Hibernate I – Estados

Hibernate utiliza el objeto **`Session`** para interactuar con la base de datos.

| Estado | Descripción |
|---------|-------------|
| **Transient** | El objeto aún no se ha guardado. |
| **Persistent** | El objeto está asociado a una sesión y a una fila de BD. |
| **Detached** | La sesión se ha cerrado y el objeto ya no está sincronizado. |

📄 **Ejemplo de sesión y transacción:**
```java

// Crear el SessionFactory usando la configuración de hibernate.cfg.xml
SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();
Session session = sessionFactory.openSession();
Transaction transaction = null;

try {
    transaction = session.beginTransaction();
    // operaciones de persistencia
    transaction.commit();
} catch (Exception e) {
    if (transaction != null) 
        transaction.rollback();
        e.printStackTrace();
} finally {
    session.close();
}
```
---

📄 **Ejemplo de sesión y transacción:**
```java

SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();
Session session = sessionFactory.openSession();
Session s = sessionFactory.openSession();
s.beginTransaction();
// operaciones de persistencia
s.getTransaction().commit();
s.close();
```

---

## 7️⃣ Métodos Importantes de `Session`

| Método | Descripción |
|---------|-------------|
| `beginTransaction()` | Inicia una transacción. |
| `createQuery()` | Crea una consulta HQL. |
| `get()` | Recupera un objeto por ID (puede devolver `null`). |
| `save()` | Inserta un nuevo registro. |
| `update()` | Actualiza un registro existente. |
| `delete()` | Elimina un registro. |

---

## 8️⃣ Carga, Almacenamiento y Modificación de Objetos

**Carga:**
```java
Cliente c = session.get(Cliente.class, 1);
```

**Guardar:**
```java
session.save(new Cliente("Ana", "López", 200));
```

**Actualizar:**
```java
cliente.setNombre("Ana María");
session.update(cliente);
```

**Eliminar:**
```java
session.delete(cliente);
```

---

## ✅ IMPORTANTE: ¿Por qué Hibernate necesita cargar objetos completos?
Cuando pedimos un ID por teclado, Hibernate **NO trabaja con IDs**, trabaja con **POJOs (objetos Java)**.

❌ INCORRECTO (esto no funciona):
```java
Order o = new Order();
o.setCustomerId(5);
session.save(o);
```

✅ CORRECTO:
```java
Customer c = session.get(Customer.class, idCliente);
Product p = session.get(Product.class, idProducto);

Order o = new Order(new Date(), cantidad, p.getPrecio() * cantidad, c, p);
session.save(o);
```

💡 Hibernate necesita el objeto completo para mantener la relación en la BD.

---

## 9️⃣ Consultas HQL (Hibernate Query Language)

📄 **Consulta clásica (NO tipada, necesita cast):**
```java
String hql = "FROM Customer";
Query consulta = session.createQuery(hql);
List results = consulta.list();
```

📄 **Consulta moderna (TIPADA, sin cast):**
```java
List<Customer> customers = session.createQuery("FROM Customer", Customer.class)
                                  .getResultList();
```

📄 **Listar pedidos de un cliente**
```java
List<Order> pedidos = session.createQuery(
"FROM Order o WHERE o.customer.id = :id ORDER BY o.fecha DESC",
Order.class)
.setParameter("id", idCliente)
.getResultList();

 for (Object obj : results) {
            Order o = (Order) obj; // cast porque List no es tipada
            System.out.println(
                "Pedido " + o.getId() +
                " - Producto: " + o.getProduct().getNombre() +
                " - Importe: " + o.getImporte()
            );
        }
```

📄 **Update con parámetros**
```java
Query q = session.createQuery(
   "update Customer set customerNumber=:num where id=:id");
q.setParameter("num", 25);
q.setParameter("id", 105);
q.executeUpdate();
session.getTransaction().commit();
```

---

## 🔟 Resumen del Tema

| Concepto | Descripción |
|-----------|-------------|
| **Clase persistente** | Objeto Java almacenable en BD. |
| **Fichero .hbm.xml** | Define relaciones entre clases y tablas. |
| **Session** | Administra conexión y operaciones. |
| **Transaction** | Controla unidades de trabajo atómicas. |
| **HQL** | Lenguaje orientado a objetos para consultas. |
| **Persistencia** | Guardar, actualizar o borrar objetos de BD. |

---
