# 🧾 Apuntes Spring Boot – Arquitectura en Capas (Ejemplo: Expedientes)

Este repositorio documenta la arquitectura en capas utilizada en un proyecto con Spring Boot, enfocada en el manejo de entidades como **Expedientes** con el uso de VO (Value Objects), Repositorios, Servicios e Implementaciones.

## Índice

- [Entity](#entity)
- [Value Object (VO)](#value-object-vo)
- [Repository](#repository)
- [Converter](#converter)
- [Service (Interfaz)](#service-interfaz)
- [ServiceImpl (Implementación del Service)](#serviceimpl-implementación-del-service)
- [Controller](#controller)
- [Diferencias entre Repository y Service](#diferencias-entre-repository-y-service)
- [Ventajas de esta arquitectura](#ventajas-de-esta-arquitectura)

---


## Value Object (VO)

- Objeto de datos **no ligado a la base de datos**.
- Se usa para transferir datos entre capas o con el frontend (como un DTO).
- Evita exponer directamente las entidades (`Entity`).
- Permite transformación de datos, omisión de relaciones, formatos personalizados, etc.

---


## Entity

- Representa una tabla de la base de datos.
- Se anota con `@Entity` y se mapea con `@Table`.
- Utiliza anotaciones como `@Id`, `@GeneratedValue`, `@Column`, `@ManyToOne`, etc.
- No debe usarse directamente en el frontend.


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


## ServiceImpl (Implementación del Service)

- Implementa la lógica real definida en la interfaz `Service`.
- Usa el repositorio (`Repository`) para acceder a la base de datos.
- Convierte entre `Entity` y `VO`.

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

    private ExpedienteVO toVO(ExpedienteEntity entity) {
        // Conversión manual
    }

    private ExpedienteEntity toEntity(ExpedienteVO vo) {
        // Conversión manual
    }
}
```

---


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
| Repository   | Acceso a datos                        | `findByAnio(int anio)`             |
| Service      | Lógica de negocio y transformación    | `getExpedientesDelAnio()`          |

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
