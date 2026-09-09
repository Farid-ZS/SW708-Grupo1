# Mapa de componentes

Fuente revisada: `src/main/java/org/springframework/samples/petclinic`.

Se han considerado las clases de produccion de la raiz y de los paquetes `model`, `owner`, `vet` y `system`. Para cada clase se anotan solo el nombre, los imports y las firmas de sus metodos publicos; no se detallan los cuerpos.

## Vista por paquetes

| Paquete  | Clases                                                                                                                                                                | Responsabilidad                                                                                                                                  | Depende de                          |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------- |
| `model`  | `BaseEntity`, `NamedEntity`, `Person`                                                                                                                                 | Proporcionar las clases base reutilizables del modelo: identidad, nombre y datos de persona.                                                     | Ningun paquete propio del proyecto. |
| `owner`  | `Owner`, `OwnerController`, `OwnerRepository`, `Pet`, `PetController`, `PetType`, `PetTypeFormatter`, `PetTypeRepository`, `PetValidator`, `Visit`, `VisitController` | Gestionar propietarios, mascotas, tipos y visitas: entidades de negocio, acceso a datos, conversion de formularios, validacion y peticiones web. | `model`.                            |
| `vet`    | `Specialty`, `Vet`, `VetController`, `VetRepository`, `Vets`                                                                                                          | Modelar veterinarios y especialidades, consulta sus datos y los expone en vistas HTML o recursos XML.                                            | `model`.                            |
| `system` | `CacheConfiguration`, `CrashController`, `WebConfiguration`, `WelcomeController`                                                                                      | Configurar aspectos transversales de la aplicacion y atiende las rutas de bienvenida y demostracion de errores.                                  | Ningun paquete propio del proyecto. |
| `(raiz)` | `PetClinicApplication`, `PetClinicRuntimeHints`                                                                                                                       | Arrancar Spring Boot y registra recursos y tipos necesarios para ejecucion nativa/reflexion.                                                     | `model`, `vet`.                     |

> Las referencias entre clases del mismo paquete no aparecen como imports Java porque Java permite resolverlas directamente dentro del paquete. Por eso, por ejemplo, `Owner` usa `Pet`, `Visit` y `Person`, aunque solo `Person` aparece en sus imports explicitos.

## Paquete `model`

### `BaseEntity`

**Imports:** `java.io.Serializable`; `jakarta.persistence.GeneratedValue`, `GenerationType`, `Id`, `MappedSuperclass`.

**Metodos publicos:**

```java
public Integer getId()
public void setId(Integer id)
public boolean isNew()
```

### `NamedEntity`

**Imports:** `jakarta.persistence.Column`, `MappedSuperclass`; `jakarta.validation.constraints.NotBlank`.

**Metodos publicos:**

```java
public String getName()
public void setName(String name)
public String toString()
```

### `Person`

**Imports:** `jakarta.persistence.Column`, `MappedSuperclass`; `jakarta.validation.constraints.NotBlank`, `Size`.

**Metodos publicos:**

```java
public String getFirstName()
public void setFirstName(String firstName)
public String getLastName()
public void setLastName(String lastName)
```

## Paquete `owner`

### `Owner`

**Imports:** `java.util.ArrayList`, `List`, `Objects`; `org.springframework.core.style.ToStringCreator`; `org.springframework.samples.petclinic.model.Person`; `org.springframework.util.Assert`; `jakarta.persistence.CascadeType`, `Column`, `Entity`, `FetchType`, `JoinColumn`, `OneToMany`, `OrderBy`, `Table`; `jakarta.validation.constraints.NotBlank`, `Pattern`.

**Metodos publicos:**

```java
public String getAddress()
public void setAddress(String address)
public String getCity()
public void setCity(String city)
public String getTelephone()
public void setTelephone(String telephone)
public List<Pet> getPets()
public void addPet(Pet pet)
public Pet getPet(String name)
public Pet getPet(Integer id)
public Pet getPet(String name, boolean ignoreNew)
public String toString()
public void addVisit(Integer petId, Visit visit)
```

### `OwnerController`

**Imports:** `java.util.List`, `Objects`, `Optional`; `org.springframework.data.domain.Page`, `PageRequest`, `Pageable`; `org.springframework.stereotype.Controller`; `org.springframework.ui.Model`; `org.springframework.validation.BindingResult`; `org.springframework.web.bind.WebDataBinder`; `org.springframework.web.bind.annotation.GetMapping`, `InitBinder`, `ModelAttribute`, `PathVariable`, `PostMapping`, `RequestParam`; `org.springframework.web.servlet.ModelAndView`; `jakarta.validation.Valid`; `org.springframework.web.servlet.mvc.support.RedirectAttributes`.

**Metodos publicos:**

```java
public void setAllowedFields(WebDataBinder dataBinder)
public Owner findOwner(Integer ownerId)
public String initCreationForm()
public String processCreationForm(Owner owner, BindingResult result, RedirectAttributes redirectAttributes)
public String initFindForm()
public String processFindForm(int page, Owner owner, BindingResult result, Model model)
public String initUpdateOwnerForm()
public String processUpdateOwnerForm(Owner owner, BindingResult result, int ownerId,
        RedirectAttributes redirectAttributes)
public ModelAndView showOwner(int ownerId)
```

### `OwnerRepository`

**Imports:** `java.util.Optional`; `org.springframework.data.domain.Page`, `Pageable`; `org.springframework.data.jpa.repository.JpaRepository`.

**Metodos publicos:**

```java
public Page<Owner> findByLastNameStartingWith(String lastName, Pageable pageable)
public Optional<Owner> findById(Integer id)
```

Tambien hereda los metodos publicos de `JpaRepository`, entre ellos las operaciones estandar de consulta y persistencia.

### `Pet`

**Imports:** `java.time.LocalDate`; `java.util.Collection`, `LinkedHashSet`, `Set`; `org.springframework.format.annotation.DateTimeFormat`; `org.springframework.samples.petclinic.model.NamedEntity`; `jakarta.persistence.CascadeType`, `Column`, `Entity`, `FetchType`, `JoinColumn`, `ManyToOne`, `OneToMany`, `OrderBy`, `Table`.

**Metodos publicos:**

```java
public void setBirthDate(LocalDate birthDate)
public LocalDate getBirthDate()
public PetType getType()
public void setType(PetType type)
public Collection<Visit> getVisits()
public void addVisit(Visit visit)
```

### `PetController`

**Imports:** `java.time.LocalDate`; `java.util.Collection`, `Objects`, `Optional`; `org.springframework.dao.DataIntegrityViolationException`; `org.springframework.stereotype.Controller`; `org.springframework.ui.ModelMap`; `org.springframework.util.Assert`, `StringUtils`; `org.springframework.validation.BindingResult`; `org.springframework.web.bind.WebDataBinder`; `org.springframework.web.bind.annotation.GetMapping`, `InitBinder`, `ModelAttribute`, `PathVariable`, `PostMapping`, `RequestMapping`; `jakarta.validation.Valid`; `org.springframework.web.servlet.mvc.support.RedirectAttributes`.

**Metodos publicos:**

```java
public Collection<PetType> populatePetTypes()
public Owner findOwner(int ownerId)
public Pet findPet(int ownerId, Integer petId)
public void initOwnerBinder(WebDataBinder dataBinder)
public void initPetBinder(WebDataBinder dataBinder)
public String initCreationForm(Owner owner, ModelMap model)
public String processCreationForm(Owner owner, Pet pet, BindingResult result,
        RedirectAttributes redirectAttributes)
public String initUpdateForm()
public String processUpdateForm(Owner owner, Pet pet, BindingResult result,
        RedirectAttributes redirectAttributes)
```

### `PetType`

**Imports:** `org.springframework.samples.petclinic.model.NamedEntity`; `jakarta.persistence.Entity`, `Table`.

**Metodos publicos:** no declara metodos; hereda los metodos publicos de `NamedEntity` y `BaseEntity`.

### `PetTypeFormatter`

**Imports:** `java.text.ParseException`; `java.util.Collection`, `Locale`, `Objects`; `org.springframework.format.Formatter`, `org.springframework.stereotype.Component`.

**Metodos publicos:**

```java
public String print(PetType petType, Locale locale)
public PetType parse(String text, Locale locale) throws ParseException
```

### `PetTypeRepository`

**Imports:** `java.util.List`; `org.springframework.data.jpa.repository.JpaRepository`; `org.springframework.data.jpa.repository.Query`.

**Metodos publicos:**

```java
public List<PetType> findPetTypes()
```

Tambien hereda los metodos publicos de `JpaRepository`.

### `PetValidator`

**Imports:** `org.springframework.util.StringUtils`; `org.springframework.validation.Errors`, `Validator`.

**Metodos publicos:**

```java
public void validate(Object obj, Errors errors)
public boolean supports(Class<?> clazz)
```

### `Visit`

**Imports:** `java.time.LocalDate`; `org.springframework.format.annotation.DateTimeFormat`; `org.springframework.samples.petclinic.model.BaseEntity`; `jakarta.persistence.Column`, `Entity`, `Table`; `jakarta.validation.constraints.NotBlank`.

**Metodos publicos:**

```java
public Visit()
public LocalDate getDate()
public void setDate(LocalDate date)
public String getDescription()
public void setDescription(String description)
```

### `VisitController`

**Imports:** `java.time.LocalDate`; `java.util.Map`, `Optional`; `org.springframework.stereotype.Controller`; `org.springframework.validation.BindingResult`; `org.springframework.web.bind.WebDataBinder`; `org.springframework.web.bind.annotation.GetMapping`, `InitBinder`, `ModelAttribute`, `PathVariable`, `PostMapping`; `jakarta.validation.Valid`; `org.springframework.web.servlet.mvc.support.RedirectAttributes`.

**Metodos publicos:**

```java
public void setAllowedFields(WebDataBinder dataBinder)
public Visit loadPetWithVisit(int ownerId, int petId, Map<String, Object> model)
public LocalDate minVisitDate()
public String initNewVisitForm()
public String processNewVisitForm(Owner owner, int petId, Visit visit,
        BindingResult result, RedirectAttributes redirectAttributes)
```

## Paquete `vet`

### `Specialty`

**Imports:** `org.springframework.samples.petclinic.model.NamedEntity`; `jakarta.persistence.Entity`, `Table`.

**Metodos publicos:** no declara metodos; hereda los metodos publicos de `NamedEntity` y `BaseEntity`.

### `Vet`

**Imports:** `java.util.Comparator`, `HashSet`, `List`, `Set`; `java.util.stream.Collectors`; `org.springframework.samples.petclinic.model.NamedEntity`, `Person`; `jakarta.persistence.Entity`, `FetchType`, `JoinColumn`, `JoinTable`, `ManyToMany`, `Table`; `jakarta.xml.bind.annotation.XmlElement`.

**Metodos publicos:**

```java
public List<Specialty> getSpecialties()
public int getNrOfSpecialties()
public void addSpecialty(Specialty specialty)
```

### `VetController`

**Imports:** `java.util.List`; `org.springframework.data.domain.Page`, `PageRequest`, `Pageable`; `org.springframework.stereotype.Controller`; `org.springframework.ui.Model`; `org.springframework.web.bind.annotation.GetMapping`, `RequestParam`, `ResponseBody`.

**Metodos publicos:**

```java
public String showVetList(int page, Model model)
public Vets showResourcesVetList()
```

### `VetRepository`

**Imports:** `java.util.Collection`; `org.springframework.cache.annotation.Cacheable`; `org.springframework.dao.DataAccessException`; `org.springframework.data.domain.Page`, `Pageable`; `org.springframework.data.repository.Repository`; `org.springframework.transaction.annotation.Transactional`.

**Metodos publicos:**

```java
public Collection<Vet> findAll() throws DataAccessException
public Page<Vet> findAll(Pageable pageable) throws DataAccessException
```

### `Vets`

**Imports:** `java.util.ArrayList`, `List`; `jakarta.xml.bind.annotation.XmlElement`, `XmlRootElement`.

**Metodos publicos:**

```java
public List<Vet> getVetList()
```

## Paquete `system`

### `CacheConfiguration`

**Imports:** `javax.cache.configuration.MutableConfiguration`; `org.springframework.boot.cache.autoconfigure.JCacheManagerCustomizer`; `org.springframework.cache.annotation.EnableCaching`; `org.springframework.context.annotation.Bean`, `Configuration`.

**Metodos publicos:**

```java
public JCacheManagerCustomizer petclinicCacheConfigurationCustomizer()
```

### `CrashController`

**Imports:** `org.springframework.stereotype.Controller`; `org.springframework.web.bind.annotation.GetMapping`.

**Metodos publicos:**

```java
public String triggerException()
```

### `WebConfiguration`

**Imports:** `java.util.Locale`; `org.springframework.context.annotation.Bean`, `Configuration`; `org.springframework.web.servlet.LocaleResolver`; `org.springframework.web.servlet.config.annotation.InterceptorRegistry`, `WebMvcConfigurer`; `org.springframework.web.servlet.i18n.LocaleChangeInterceptor`, `SessionLocaleResolver`.

**Metodos publicos:**

```java
public LocaleResolver localeResolver()
public LocaleChangeInterceptor localeChangeInterceptor()
public void addInterceptors(InterceptorRegistry registry)
```

### `WelcomeController`

**Imports:** `org.springframework.stereotype.Controller`; `org.springframework.web.bind.annotation.GetMapping`.

**Metodos publicos:**

```java
public String welcome()
```

## Paquete raiz

### `PetClinicApplication`

**Imports:** `org.springframework.boot.SpringApplication`; `org.springframework.boot.autoconfigure.SpringBootApplication`; `org.springframework.context.annotation.ImportRuntimeHints`.

**Metodos publicos:**

```java
public static void main(String[] args)
```

### `PetClinicRuntimeHints`

**Imports:** `org.springframework.aot.hint.RuntimeHints`, `RuntimeHintsRegistrar`; `org.springframework.samples.petclinic.model.BaseEntity`, `Person`; `org.springframework.samples.petclinic.vet.Vet`.

**Metodos publicos:**

```java
public void registerHints(RuntimeHints hints, ClassLoader classLoader)
```

## Roles dentro de `owner`

Las 11 clases se pueden agrupar en estos roles:

| Rol                           | Clases                                                | Que hacen                                                                                          |
| ----------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Reciben peticiones web        | `OwnerController`, `PetController`, `VisitController` | Definen rutas MVC, reciben parametros/formularios y devuelven vistas o redirecciones.              |
| Hablan con la base de datos   | `OwnerRepository`, `PetTypeRepository`                | Definen consultas y operaciones de persistencia para `Owner` y `PetType` mediante Spring Data JPA. |
| Representan cosas del negocio | `Owner`, `Pet`, `PetType`, `Visit`                    | Son entidades del dominio y expresan sus datos y relaciones.                                       |
| Adaptan y validan formularios | `PetTypeFormatter`, `PetValidator`                    | Convierten `PetType` desde/hacia texto y validan los datos de una mascota.                         |

## Pregunta para el equipo

`model` no depende de ningun paquete propio porque es la capa mas basica y reutilizable del dominio. Sus clases solo necesitan tipos de Java y anotaciones de librerias externas; no conocen controladores, repositorios ni detalles de otros modulos de la aplicacion.

`owner` depende de `model` porque sus entidades reutilizan las clases base: `Owner` extiende `Person`, `Pet` y `PetType` extienden `NamedEntity`, y `Visit` extiende `BaseEntity`. Esa direccion tiene sentido: el paquete funcional `owner` especializa las abstracciones comunes de `model`.

## Flujo: ficha de un dueño

1. **Navegador web**: Realiza una petición HTTP `GET` a la ruta `/owners/1`.
2. **`OwnerController.java`** + método `showOwner(int ownerId)`: Intercepta la petición mapeada con `@GetMapping("/owners/{ownerId}")` y delega la consulta invocando a `this.owners.findById(ownerId)`.
3. **`OwnerRepository.java`** + método `findById(Integer id)`: Ejecuta la consulta de persistencia con Spring Data JPA para buscar el registro por su identificador.
4. **`schema.sql`** + tabla `owners`: La base de datos resuelve la consulta SQL física (`SELECT`) sobre la tabla relacional `owners`.
5. **`Owner.java`** + entidad de dominio (`@Entity`, `@Table(name = "owners")`): JPA mapea las columnas de la tabla en los atributos del objeto `Owner` (junto con sus mascotas y visitas asociadas).
6. **`ownerDetails.html`** (`src/main/resources/templates/owners/ownerDetails.html`): El motor Thymeleaf procesa el modelo devuelto por el controlador, renderiza la ficha del dueño y envía el HTML final de respuesta al navegador.
