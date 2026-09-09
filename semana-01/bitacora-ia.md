# Bitácora de uso de IA – Spring PetClinic

## 1. Objetivo

Utilizar una herramienta de Inteligencia Artificial como apoyo para comprender la estructura de un proyecto de software desconocido, en este caso **Spring PetClinic**, y posteriormente verificar las afirmaciones proporcionadas por la IA directamente contra el código fuente.

El propósito de la actividad no es asumir que toda la información proporcionada por la IA es correcta, sino comprobar cada afirmación mediante la revisión del código.

---

## 2. Prompt utilizado

Se realizó la siguiente consulta a ChatGPT:

> Explícame cómo está organizado el código de Spring PetClinic: paquetes, responsabilidades y cómo se atiende la petición GET `/owners/1`.

---

## 3. Resumen de la respuesta de la IA

La IA indicó que Spring PetClinic es una aplicación desarrollada con Spring Boot y que su código principal se encuentra dentro del paquete:

```text
org.springframework.samples.petclinic
```

Dentro de este paquete, el proyecto se divide principalmente según las funcionalidades o elementos del dominio.

Entre los paquetes encontrados se encuentran:

```text
petclinic
├── model
├── owner
├── system
└── vet
```

Además, en el paquete principal existen clases como:

```text
PetClinicApplication.java
PetClinicRuntimeHints.java
```

### Paquete `model`

Contiene clases base utilizadas por diferentes entidades del sistema.

Su responsabilidad es proporcionar estructuras comunes que pueden ser reutilizadas por otras clases del dominio.

### Paquete `owner`

Contiene la funcionalidad relacionada con los propietarios de mascotas, las mascotas y sus visitas.

Entre las clases encontradas se encuentran:

```text
Owner.java
OwnerController.java
OwnerRepository.java

Pet.java
PetController.java
PetType.java
PetTypeFormatter.java
PetTypeRepository.java
PetValidator.java

Visit.java
VisitController.java
```

Dentro de este paquete se pueden observar diferentes responsabilidades.

- `Owner.java`: representa a un propietario dentro del dominio.
- `Pet.java`: representa una mascota.
- `Visit.java`: representa una visita asociada a una mascota.
- `OwnerController.java`: recibe y procesa las peticiones HTTP relacionadas con propietarios.
- `OwnerRepository.java`: proporciona acceso a los datos de los propietarios.
- `PetController.java`: procesa las operaciones relacionadas con mascotas.
- `VisitController.java`: procesa las operaciones relacionadas con visitas.

### Paquete `vet`

Agrupa la funcionalidad relacionada con los veterinarios del sistema.

### Paquete `system`

Contiene componentes relacionados con funcionalidades generales o de infraestructura de la aplicación.

---

## 4. Petición analizada: `GET /owners/1`

La IA explicó que una petición como:

```http
GET /owners/1
```

es atendida por Spring MVC mediante `OwnerController`.

Dentro de `OwnerController` existe un método asociado a la ruta:

```java
@GetMapping("/owners/{ownerId}")
```

El método correspondiente es:

```java
public ModelAndView showOwner(@PathVariable("ownerId") int ownerId)
```

Por lo tanto, si el usuario realiza:

```http
GET /owners/1
```

el valor:

```text
1
```

es capturado como el parámetro:

```java
ownerId
```

El flujo simplificado de la petición es:

```text
Navegador
   |
   | GET /owners/1
   v
Spring MVC
   |
   v
OwnerController
   |
   | showOwner(1)
   v
OwnerRepository
   |
   | findById(1)
   v
Base de datos
   |
   v
Owner
   |
   v
ModelAndView
   |
   v
owners/ownerDetails
   |
   v
Respuesta HTML
```

---

## 5. Funcionamiento de `OwnerController`

El controlador utiliza `OwnerRepository` para acceder a la información de los propietarios.

La dependencia se declara de la siguiente manera:

```java
private final OwnerRepository owners;
```

y es recibida mediante el constructor de `OwnerController`.

Para mostrar un propietario específico existe el método asociado a:

```java
@GetMapping("/owners/{ownerId}")
```

Dentro del método se crea un `ModelAndView` utilizando la vista:

```text
owners/ownerDetails
```

Posteriormente se realiza una búsqueda utilizando:

```java
this.owners.findById(ownerId)
```

Si el propietario existe, el objeto recuperado se añade al modelo y posteriormente se utiliza la vista `owners/ownerDetails` para presentar su información.

---

## 6. Funcionamiento de `OwnerRepository`

La IA indicó que `OwnerRepository` es la interfaz utilizada para acceder a los datos de los propietarios.

Al revisar el código se encontró que está declarada como:

```java
public interface OwnerRepository extends JpaRepository<Owner, Integer>
```

Esto permite utilizar las operaciones proporcionadas por Spring Data JPA.

Además, en esta versión del proyecto aparece explícitamente el método:

```java
Optional<Owner> findById(Integer id);
```

También existe una operación para buscar propietarios utilizando el inicio de su apellido:

```java
Page<Owner> findByLastNameStartingWith(
    String lastName,
    Pageable pageable
);
```

---

## 7. Verificación de las afirmaciones de la IA

Después de recibir la explicación de la IA, se revisó directamente el código fuente del proyecto para comprobar cada afirmación.

| N.º | Afirmación realizada por la IA | Resultado | Evidencia encontrada |
|---|---|---|---|
| 1 | Existe un paquete llamado `owner`. | Cierta | Se encuentra en `src/main/java/org/springframework/samples/petclinic/owner`. |
| 2 | Existe una clase llamada `OwnerController`. | Cierta | Se encontró `OwnerController.java` dentro del paquete `owner`. |
| 3 | `OwnerController` gestiona peticiones relacionadas con propietarios. | Cierta | Contiene rutas para crear, buscar, editar y mostrar propietarios. |
| 4 | La petición `GET /owners/1` es atendida por `OwnerController`. | Cierta | Existe `@GetMapping("/owners/{ownerId}")`. |
| 5 | El número `1` de `/owners/1` se recibe mediante `@PathVariable`. | Cierta | El método `showOwner` recibe `@PathVariable("ownerId") int ownerId`. |
| 6 | El controlador utiliza `OwnerRepository` para obtener el propietario. | Cierta | Se ejecuta `this.owners.findById(ownerId)`. |
| 7 | `OwnerRepository` extiende `JpaRepository<Owner, Integer>`. | Cierta | La interfaz está declarada utilizando dicha herencia. |
| 8 | La vista utilizada para mostrar al propietario es `owners/ownerDetails`. | Cierta | Se crea `new ModelAndView("owners/ownerDetails")`. |
| 9 | Si el propietario existe, se agrega al `ModelAndView`. | Cierta | El objeto `Owner` recuperado es añadido mediante `mav.addObject(owner)`. |
| 10 | Si el propietario no existe, el controlador devuelve automáticamente HTTP 404. | **Falsa** | El código de `showOwner()` lanza una `IllegalArgumentException`; este método por sí solo no permite afirmar que la respuesta sea automáticamente HTTP 404. |

---

## 8. Error encontrado en la respuesta de la IA

Durante la explicación inicial se afirmó que, cuando se realiza una petición como:

```http
GET /owners/1
```

y el propietario solicitado no existe, Spring PetClinic devuelve automáticamente una respuesta HTTP `404 Not Found`.

Sin embargo, al revisar directamente el código de `OwnerController`, se encontró que el propietario se obtiene mediante:

```java
Optional<Owner> optionalOwner = this.owners.findById(ownerId);
```

y posteriormente se utiliza:

```java
Owner owner = optionalOwner.orElseThrow(
    () -> new IllegalArgumentException(...)
);
```

Por lo tanto, si el propietario no existe, este método lanza una:

```text
IllegalArgumentException
```

La afirmación de que necesariamente se produce automáticamente un HTTP 404 no puede deducirse únicamente de este código.

### Clasificación

**Afirmación: FALSA**

### Qué dijo la IA

La IA afirmó que un propietario inexistente provoca automáticamente una respuesta:

```text
HTTP 404 Not Found
```

### Qué dice el código

El código observado en `OwnerController` utiliza `orElseThrow()` y genera una:

```text
IllegalArgumentException
```

Por ello, la respuesta inicial de la IA fue demasiado específica y no estaba respaldada directamente por el código revisado.

Este caso demuestra la importancia de comprobar las respuestas generadas por una IA antes de asumirlas como correctas.

---

## 9. Flujo verificado de `GET /owners/1`

Después de revisar el código, el flujo que sí se puede afirmar a partir de la implementación observada es:

```text
GET /owners/1
      |
      v
OwnerController
      |
      | @GetMapping("/owners/{ownerId}")
      |
      v
showOwner(ownerId = 1)
      |
      v
OwnerRepository
      |
      | findById(1)
      |
      v
Optional<Owner>
      |
      +----------------------------+
      |                            |
   Existe                      No existe
      |                            |
      v                            v
  Owner                     IllegalArgumentException
      |
      v
mav.addObject(owner)
      |
      v
owners/ownerDetails
      |
      v
Respuesta al usuario
```

---

## 10. Conclusión

El uso de ChatGPT permitió obtener rápidamente una explicación inicial sobre la estructura de Spring PetClinic y comprender el recorrido general de una petición como:

```http
GET /owners/1
```

La mayoría de las afirmaciones realizadas por la IA pudieron comprobarse directamente en el código fuente, especialmente la existencia de `OwnerController`, `OwnerRepository`, el uso de `@GetMapping`, `@PathVariable`, `findById()` y la vista `owners/ownerDetails`.

Sin embargo, también se encontró una afirmación incorrecta: la IA indicó inicialmente que un propietario inexistente genera automáticamente una respuesta HTTP 404. La revisión del código mostró que `showOwner()` lanza una `IllegalArgumentException`, por lo que la afirmación original no estaba respaldada directamente por la implementación observada.

La actividad demuestra que la Inteligencia Artificial puede ser útil como herramienta de orientación para explorar código desconocido, pero sus respuestas deben ser contrastadas con el código fuente antes de considerarlas correctas.