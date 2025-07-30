# ANOTACIONESSPRING

## @Configuration
  Declara una clase como fuente de definiciones de beans (como un archivo XML, pero en Java).
## @Bean
  Registra un bean manualmente en el contexto de Spring. Un Bean es objeto que es gestionado por el contenedor de Spring.
  Porque Spring utiliza el Inversión de Control (IoC), donde no creas objetos tú directamente con new, sino que Spring los crea por ti y te los inyecta donde los necesitas.
## @PropertySource
  Carga un archivo .properties adicional.
## @Import
  Importa una clase de configuración a otra.
## @Component
  Marca una clase como un bean administrado por Spring.
## @Service
  Variante especializada de @Component para la capa de servicio.
## @Repository
  Marca la clase de acceso a datos. Añade manejo automático de excepciones.
## @Controller
  Controlador MVC tradicional (devuelve vistas).
## @RestController
  Controlador REST (devuelve JSON). Equivale a @Controller + @ResponseBody.
## @Autowired
  Inyecta automáticamente una dependencia por tipo.
## @Qualifier("nombre")
  Se usa junto a @Autowired para especificar cuál bean usar si hay varios del mismo tipo.
## @Value("${...}")
  Inyecta valores desde propiedades o variables de entorno.
## @Lazy
  Retrasa la inicialización de un bean hasta que sea necesario.
## @Primary
  Marca un bean como la opción preferida cuando hay múltiples candidatos.
## @RequestMapping
  	Mapea solicitudes HTTP a métodos controladores. Se puede usar con GET, POST, etc.
## @GetMapping, @PostMapping, etc.
  Atajos de @RequestMapping para verbos HTTP.
## @PathVariable
  Extrae valores de la URL.
## @RequestParam
  Extrae parámetros de la query (?nombre=valor).
## @RequestBody
  Deserializa JSON del cuerpo de una solicitud a un objeto Java.
## @ResponseBody
  Serializa el objeto Java de retorno como JSON.
## @ModelAttribute
  Vincula parámetros de formulario a un objeto. (incluye archivos multipartfile con json)
## @CrossOrigin
  Habilita CORS para un endpoint.
## @Valid
  Habilita validación sobre un objeto (con Bean Validation).
## @NotNull, @Size, @Email, etc.
  Anotaciones de validación estándar de Java
## @ExceptionHandler
  Maneja excepciones dentro de controladores.
## @Entity
  Marca una clase como una entidad JPA (tabla).
## @Id
  Marca el campo como clave primaria.
## @GeneratedValue
  Especifica la estrategia de generación del ID.
## @Column
  Configura la columna de una entidad.
## @Table
  Configura la tabla asociada a la entidad.
## @OneToOne, @OneToMany, @ManyToOne, @ManyToMany
  Relaciones entre entidades.
## @JoinColumn
  Especifica la columna de unión en relaciones.
## @Repository
  Marca la interfaz DAO o repositorio.
## @Query
  Define consultas personalizadas.
## @Transactional
 Marca un método o clase como transaccional.
## @EnableWebSecurity
  Habilita la configuración de seguridad web en Spring. Se usa en una clase @Configuration.
## @EnableMethodSecurity (Spring 6 y Boot 3)
  Habilita la seguridad a nivel de métodos. 
##
##
##
##
##


# DETALLE DE ALGUNAS ANOTACIONES
  Anotaciones como @PreAuthorize
   Son anotaciones de seguridad que se colocan encima de métodos (o clases) para permitir o bloquear el acceso dependiendo     de quién esté usando la aplicación.
   Estas anotaciones funcionan cuando activas la seguridad a nivel de métodos con:
   ```
@EnableMethodSecurity(securedEnabled = true, prePostEnabled = true)
```

## @Secured("ROLE_ADMIN")
  📌 ¿Qué hace?
  Solo permite acceder al método si el usuario tiene el rol ADMIN.
  📌 Importante:
  El rol debe tener el prefijo ROLE_, es una convención en Spring.
   ```
  @Secured("ROLE_ADMIN")
  public void verUsuarios() {
      // Solo un usuario con rol ADMIN puede llegar aquí
  }
   ```
## @PreAuthorize("hasRole('ADMIN')")
  📌 ¿Qué hace?
  Evalúa la expresión antes de ejecutar el método. Es más flexible que @Secured.
  uedes usar condiciones como:
  - hasRole('ADMIN')
  - hasAuthority('READ_PRIVILEGE')
  - isAuthenticated()
  -authentication.name == 'carlos'
   ```
  @PreAuthorize("hasRole('ADMIN')")
  public void eliminarUsuario() {
      // Solo ADMIN puede
  }
  @PreAuthorize("hasAuthority('READ_PRIVILEGE')")
  public void verInformes() {
      // Solo quienes tengan ese permiso específico
  }
   ```
## @RolesAllowed("ADMIN")
  No usa el prefijo ROLE_.
   ```
  @RolesAllowed("ADMIN")
  public void crearInforme() {
      // Solo usuarios con el rol ADMIN (sin ROLE_)
  }
   ```
  Multiples roles
   ```
  @RolesAllowed({"USER", "MODERATOR"})
  public void verForo() {
      // USER o MODERATOR pueden entrar
  }
   ```
## @PostAuthorize("...")
📌 ¿Qué hace?
Evalúa una condición después de que el método se ejecuta, y puede bloquear la respuesta si no se cumple.
Ideal cuando:
No sabes si el usuario puede ver los datos hasta que los cargas.
Quieres comparar el resultado con el usuario actual.
```
@PostAuthorize("returnObject.owner == authentication.name")
public Documento obtenerDocumento(Long id) {
    Documento doc = documentoService.buscar(id);
    return doc;
}
```

# EJEMPLO
 ```
@RestController
@RequestMapping("/admin")
public class AdminController {

    @Secured("ROLE_ADMIN")
    @GetMapping("/config")
    public String verConfiguracion() {
        return "Configuración solo para admins";
    }

    @PreAuthorize("hasAuthority('READ_PRIVILEGE')")
    @GetMapping("/informe")
    public String verInforme() {
        return "Vista del informe con privilegio de lectura";
    }

    @PostAuthorize("returnObject.usuario == authentication.name")
    @GetMapping("/datos")
    public Datos obtenerDatos() {
        return new Datos("carlos"); // Solo si 'carlos' es el usuario actual
    }

    @RolesAllowed({"ADMIN", "MANAGER"})
    @GetMapping("/reporte")
    public String verReporte() {
        return "Reporte para admins o managers";
    }
}
```

## 📘 ¿Qué es authentication?
Es el objeto de Spring que representa al usuario actual. Tiene información como:

Campo	Descripción
- authentication.name	            Nombre de usuario
- authentication.authorities	    Lista de roles/autoridades
- authentication.principal	      El objeto completo del usuario (UserDetails u otro)
- authentication.authenticated	  true/false
Puedes hacer expresiones como:
```
@PostAuthorize("returnObject.usuario == authentication.name")
@PostAuthorize("authentication.authorities.contains('ROLE_ADMIN')")
@PostAuthorize("authentication.principal.email == returnObject.email")
```


