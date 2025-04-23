# 🧾 Apuntes Spring Boot – Arquitectura en Capas

Este repositorio documenta la arquitectura en capas utilizada en un proyecto con Spring Boot, enfocada en el manejo de entidades como **Expedientes** con el uso de VO (Value Objects), Repositorios, Servicios e Implementaciones.

## Índice

- [Entity](#entity)
- [Value Object (VO)](#value-object-vo)
- [Converter](#converter)
- [Repository](#repository)
- [Service (Interfaz)](#service-interfaz)
- [ServiceImpl (Implementación del Service)](#serviceimpl-implementación-del-service)
- [Controller](#controller)
- [Diferencias entre Repository y Service](#diferencias-entre-repository-y-service)
- [Ventajas de esta arquitectura](#ventajas-de-esta-arquitectura)
- [Flujo de datos entre frontend (Angular) y backend (Spring Boot)](#🧩-flujo-de-datos-entre-frontend-angular-y-backend-spring-boot)


---

## Entity

- Representa una tabla de la base de datos.
- Se anota con `@Entity` y se mapea con `@Table`.
- Utiliza anotaciones como `@Id`, `@GeneratedValue`, `@Column`, `@ManyToOne`, etc.
- No debe usarse directamente en el frontend.


---


## Value Object (VO)

- Objeto de datos **no ligado a la base de datos**.
- Se usa para transferir datos entre capas o con el frontend (como un DTO).
- Evita exponer directamente las entidades (`Entity`).
- Permite transformación de datos, omisión de relaciones, formatos personalizados, etc.

---

## Converter

- Se encarga de transformar datos entre `Entity` y `VO`.
- Se recomienda crear un paquete `converter` para mantener limpio el código.
- Evita duplicar lógica de conversión en múltiples clases.
- Facilita pruebas unitarias y mantenimiento.

**Ejemplo:**

```java
@Component
public class ExpedienteConverter {

    public ExpedienteVO toVO(ExpedienteEntity entity) {
        if (entity == null) return null;
        ExpedienteVO vo = new ExpedienteVO();
        vo.setId(entity.getId());
        vo.setIdTipoExpediente(entity.getIdTipoExpediente());
        vo.setAnio(entity.getAnio());
        vo.setCodigo(entity.getCodigo());
        vo.setFechaAlta(entity.getFechaAlta());
        vo.setNEstado(entity.getNEstado());
        vo.setIdUsuarioResponsable(entity.getIdUsuarioResponsable());
        vo.setAsunto(entity.getAsunto());
        return vo;
    }

    public ExpedienteEntity toEntity(ExpedienteVO vo) {
        if (vo == null) return null;
        ExpedienteEntity entity = new ExpedienteEntity();
        entity.setId(vo.getId());
        entity.setIdTipoExpediente(vo.getIdTipoExpediente());
        entity.setAnio(vo.getAnio());
        entity.setCodigo(vo.getCodigo());
        entity.setFechaAlta(vo.getFechaAlta());
        entity.setNEstado(vo.getNEstado());
        entity.setIdUsuarioResponsable(vo.getIdUsuarioResponsable());
        entity.setAsunto(vo.getAsunto());
        return entity;
    }
}
```

---

## Repository

- Capa de **acceso a datos**.
- Extiende de `JpaRepository<Entity, ID>`.
- Permite crear métodos de consulta personalizados según los atributos de la entidad.

**Ejemplo:**

```java
@Repository
public interface ExpedienteRepository extends JpaRepository<ExpedienteEntity, Long> {
    List<ExpedienteEntity> findByAnio(int anio);
}
```

---

## Service (Interfaz)

- Define el **contrato** de lo que se puede hacer con el recurso.
- Separa la lógica del negocio de la presentación.
- Mejora el mantenimiento y testeo del código.

**Ejemplo:**

```java
public interface ExpedienteService {
    List<ExpedienteVO> findAll();
    ExpedienteVO findById(Long id);
    ExpedienteVO save(ExpedienteVO expediente);
    void deleteById(Long id);
}
```

---


## ServiceImpl (Implementación del Service)

- Implementa la lógica real definida en la interfaz `Service`.
- Usa el repositorio (`Repository`) para acceder a la base de datos.
- Convierte entre `Entity` y `VO' utilizando el converter

**Ejemplo:**

```java
@Service
public class ExpedienteServiceImpl implements ExpedienteService {

    @Autowired
    private ExpedienteRepository expedienteRepository;

    @Override
    public List<ExpedienteVO> findAll() {
        return expedienteRepository.findAll().stream()
            .map(this::toVO)
            .collect(Collectors.toList());
    }
}
```

---


## Controller

- Expone los endpoints de la API REST.
- Se anota con `@RestController` y `@RequestMapping`.
- Llama a los servicios (`Service`) para procesar las peticiones.
- Convierte datos de entrada (JSON → VO) y salida (VO → JSON).

**Ejemplo:**

```java
@RestController
@RequestMapping("/api/expedientes")
public class ExpedienteController {

    @Autowired
    private ExpedienteService expedienteService;

    @GetMapping
    public List<ExpedienteVO> getAll() {
        return expedienteService.findAll();
    }

    @GetMapping("/{id}")
    public ExpedienteVO getById(@PathVariable Long id) {
        return expedienteService.findById(id);
    }

    @PostMapping
    public ExpedienteVO create(@RequestBody ExpedienteVO expedienteVO) {
        return expedienteService.save(expedienteVO);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        expedienteService.deleteById(id);
    }
}
```


## Diferencias entre Repository y Service

| Capa         | Función principal                     | Ejemplo                            |
|--------------|----------------------------------------|------------------------------------|
| Repository   | Acceso a datos                         | `findByAnio(int anio)`             |
| Service      | Lógica de negocio                      | `getExpedientesDelAnio()`          |

---

## Ventajas de esta arquitectura

- Código más limpio y escalable
- Separación clara de responsabilidades
- Mayor seguridad (no se exponen Entities directamente)
- Reutilización de lógica de negocio
- Facilidad para testear
- Adaptabilidad para futuras mejoras

---

> Esta estructura sigue buenas prácticas y facilita el desarrollo de aplicaciones robustas en Spring Boot.

---

## Flujo de datos entre frontend (Angular) y backend (Spring Boot)

El flujo de datos entre el cliente y el servidor se basa en un modelo de comunicación **cliente-servidor** utilizando **peticiones HTTP** sobre el protocolo REST. Angular, desde el navegador del usuario, actúa como cliente que interactúa con el servidor desarrollado en Spring Boot a través de servicios específicos.

Este flujo se puede resumir en los siguientes pasos:

1. **Acción del usuario en el frontend:**
   El usuario interactúa con la interfaz (por ejemplo, al registrarse). Esta acción desencadena una llamada a un método en un componente Angular, que a su vez utiliza un servicio para comunicarse con el backend.

2. **Llamada HTTP desde Angular:**
   Angular realiza una petición HTTP (`GET`, `POST`, `PUT` o `DELETE`) a un endpoint definido en el backend. La información se envía habitualmente en formato JSON, ya sea como parámetros en la URL o como cuerpo de la petición.

3. **Intervención del interceptor:**
   Antes de que la petición llegue al servidor, el **interceptor HTTP** añade el token JWT (si el usuario está autenticado) en la cabecera de la petición. Esto permite que el backend identifique al usuario y verifique sus permisos.

4. **Recepción y procesamiento en el backend (Spring Boot):**
   El servidor recibe la petición y la redirige al controlador (`Controller`) correspondiente. Este llama al servicio (`Service`) que contiene la lógica de negocio, y este, a su vez, interactúa con los repositorios para consultar o modificar la base de datos.

5. **Transformación de datos (Entity → VO):**
   Antes de enviar la respuesta al cliente, los datos recuperados de la base de datos (entidades) se transforman en objetos VO (Value Object), que contienen solo la información necesaria para el frontend, evitando exponer detalles internos del modelo.

6. **Respuesta al frontend:**
   El backend devuelve una respuesta HTTP con un objeto JSON, que puede ser una lista de libros, el resultado de una inscripción, los datos del usuario, etc. En caso de error (por ejemplo, si el token no es válido), se devuelve un código 401 o 403 con un mensaje explicativo.

7. **Actualización del frontend:**
   Angular recibe los datos y actualiza dinámicamente la interfaz de usuario. Gracias al sistema reactivo de Angular, estos cambios se reflejan inmediatamente sin necesidad de recargar la página.

Este modelo de flujo de datos garantiza una **comunicación eficiente, segura y estructurada** entre ambas partes. Además, permite escalar la aplicación fácilmente y mantener separadas las responsabilidades de presentación (Angular) y lógica de negocio/datos (Spring Boot).

