# 🧾 Apuntes Spring Boot – Arquitectura en Capas (Ejemplo: Expedientes)

Este repositorio documenta la arquitectura en capas utilizada en un proyecto con Spring Boot, enfocada en el manejo de entidades como **Expedientes** y **Tipos de Expediente**, con el uso de VO (Value Objects), Repositorios, Servicios e Implementaciones.

---

## 📂 Estructura del proyecto

```
com.practicasSoltel.back
├── controller
├── entity
├── vo
├── repository
├── service
│   ├── ExpedienteService.java
│   └── impl
│       └── ExpedienteServiceImpl.java
```

---

## 🔸 Value Object (VO)

- Objeto de datos **no ligado a la base de datos**.
- Se usa para transferir datos entre capas o con el frontend (como un DTO).
- Evita exponer directamente las entidades (`Entity`).
- Permite transformación de datos, omisión de relaciones, formatos personalizados, etc.

**Ejemplo:**

```java
public class ExpedienteVO {
    private Long id;
    private Long idTipoExpediente;
    private int anio;
    private String codigo;
    private LocalDate fechaAlta;
    private int nEstado;
    private Long idUsuarioResponsable;
    private String asunto;
}
```

---

## 🔸 Repository

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

## 🔸 Service (Interfaz)

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

## 🔸 ServiceImpl (Implementación del Service)

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

## 🔍 Diferencias entre Repository y Service

| Capa         | Función principal                     | Ejemplo                            |
|--------------|----------------------------------------|------------------------------------|
| Repository   | Acceso a datos                        | `findByAnio(int anio)`             |
| Service      | Lógica de negocio y transformación    | `getExpedientesDelAnio()`          |

---

## ✅ Ventajas de esta arquitectura

- Código más limpio y escalable
- Separación clara de responsabilidades
- Mayor seguridad (no se exponen Entities directamente)
- Reutilización de lógica de negocio
- Facilidad para testear
- Adaptabilidad para futuras mejoras

---

> ✨ Esta estructura sigue buenas prácticas y facilita el desarrollo de aplicaciones robustas en Spring Boot.
