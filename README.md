### 📂 Estructura de Paquetes y Código Creado                                                                                                                                                     
                                                                                                                                                                                                    
  Los archivos se han distribuido en el directorio  ms-catalogo/src/main/java/cl/bookpointchile/catalogo  de acuerdo al patrón CSR:                                                                 
                                                                                                                                                                                                    
    ms-catalogo/                                                                                                                                                                                    
    ├── pom.xml                                     # Dependencias del proyecto (Lombok, Validation, JPA)                                                                                           
    └── src/                                                                                                                                                                                        
        └── main/                                                                                                                                                                                   
            ├── java/                                                                                                                                                                               
            │   └── cl/                                                                                                                                                                             
            │       └── bookpointchile/                                                                                                                                                             
            │           └── catalogo/                                                                                                                                                               
            │               ├── config/                                                                                                                                                             
            │               │   └── DataInitializer.java  # Sembrador de catálogo base y valoraciones iniciales                                                                                     
            │               ├── controller/                                                                                                                                                         
            │               │   └── CatalogoController.java # Endpoints REST con paginación y filtros                                                                                               
            │               ├── dto/                                                                                                                                                                
            │               │   ├── ResenaRequestDTO.java  # Validaciones JSR 380 (1 a 5 estrellas)                                                                                                 
            │               │   ├── ProductoRegistroRequestDTO.java # Estructura para registrar productos                                                                                           
            │               │   ├── ResenaResponseDTO.java  # DTO para comentarios de clientes                                                                                                      
            │               │   └── ProductoResponseDTO.java # DTO con promedio de estrellas calculado                                                                                              
            │               ├── exception/                                                                                                                                                          
            │               │   ├── ResourceNotFoundException.java                                                                                                                                  
            │               │   ├── ProductoNoEncontradoException.java # HTTP 404                                                                                                                   
            │               │   ├── ErrorResponse.java          # Payload JSON limpio de errores                                                                                                    
            │               │   └── GlobalExceptionHandler.java  # @RestControllerAdvice interceptor                                                                                                
            │               ├── model/                                                                                                                                                              
            │               │   ├── Producto.java               # Entidad catálogo (libros, papelería, material)                                                                                    
            │               │   └── Resena.java                 # Entidad para las opiniones y estrellas                                                                                            
            │               ├── repository/                                                                                                                                                         
            │               │   ├── ProductoRepository.java     # JPQL dinámico avanzado para filtros y páginas                                                                                     
            │               │   └── ResenaRepository.java                                                                                                                                           
            │               ├── service/                                                                                                                                                            
            │               │   ├── CatalogoService.java                                                                                                                                            
            │               │   └── CatalogoServiceImpl.java    # Lógica de cálculo de promedios y logs                                                                                             
            │               └── CatalogoApplication.java        # Bootstrap de la aplicación Spring Boot                                                                                            
            └── resources/                                                                                                                                                                          
                └── application.properties                  # Configuración de puerto, base de datos y logs                                                                                         
  ──────                                                                                                                                                                                            
  ### 🛠️ Decisiones de Desarrollo y Diseño                                                                                                                                                           
                                                                                                                                                                                                    
  1. Evitar el Problema de Consultas N+1 con Paginación:                                                                                                                                            
      • En ProductoRepository.java se ha diseñado una consulta JPQL dinámica avanzada que realiza un  LEFT JOIN FETCH  de las reseñas del producto en conjunto con un filtrado flexible de campos (
autor ,  
      editorial ,  categoria , rango de precios). Esto minimiza los viajes de red, retornando un objeto  Page<Producto>  óptimo.                                                                    
  2. Cálculo Dinámico de Calificaciones en Capa de Negocio:                                                                                                                                         
      • El cálculo del promedio de estrellas no está hardcodeado en la base de datos. CatalogoServiceImpl.java procesa dinámicamente las reseñas asociadas a cada producto usando la API de Streams de
Java 17,
      obteniendo el promedio aritmético, redondeándolo a un decimal e inyectándolo en el campo  calificacionPromedio  del DTO de respuesta.                                                         
  3. Garantía Bidireccional de Relaciones JPA:                                                                                                                                                      
      • Producto.java gestiona la sincronización con Resena.java usando los métodos utilitarios  addResena  y  removeResena , asegurando que la relación bidireccional se mantenga coherente en 
      el contexto de persistencia de Hibernate.                                                                                                                                                     
  4. Validaciones Beans Estrictas (JSR 380):                                                                                                                                                        
      • En ResenaRequestDTO.java se limita la calificación de estrellas estrictamente mediante  @Min(1)  y  @Max(5) . Además, se agregaron validaciones de longitud para evitar inyección de cadenas      
      excesivamente largas en los comentarios.                                                                                                                                                      
  5. Sembrado Automático del Catálogo (Data Seeder):                                                                                                                                                
      • He programado DataInitializer.java. En el primer arranque, registrará de forma automática libros de programación clásicos, sets de papeleríaStabilo y kits robóticos Arduino, insertando
además   
      reseñas reales de compradores para que los cálculos de promedios de estrellas e interfaces funcionen inmediatamente en tus pruebas.                                                           
                                                                                                                                                                                                    
  ──────                                                                                                                                                                                            
  ### ⚙️ Propiedades del Entorno                                                                                                                                                                     
                                                                                                                                                                                                    
  En application.properties se han ajustado los siguientes parámetros base:                                                                                                                                
                                                                                                                                                                                                    
  • Puerto:  server.port=8084  (independiente de ms-ventas  8081 , ms-inventario  8082  y ms-usuarios  8083 ).                                                                                      
  • Base de datos: MySQL esquema  bookpoint_catalogo  (con  createDatabaseIfNotExist=true ).                                                                                                        
  • Logging: Activado a nivel  INFO  para el rastreo de búsquedas y advertencias en el registro de reseñas.                                                                                         
  ──────                                                                                                                                                                                            
  ### 🔍 Endpoints REST Expuestos                                                                                                                                                                   
                                                                                                                                                                                                    
  •  GET /api/catalogo/productos : Retorna el catálogo con soporte de paginación ( page  y  size ) y parámetros de búsqueda opcionales ( autor ,  editorial ,  categoria ,  precioMin ,  precioMax ).
  •  GET /api/catalogo/productos/{id} : Detalle de un producto específico (calculando su valoración promedio y listando sus opiniones).                                                             
  •  POST /api/catalogo/productos : Registra un nuevo producto en vitrina.                                                                                                                          
  •  POST /api/catalogo/productos/{id}/resenas : Permite a un cliente calificar con estrellas y comentar sobre un libro (dispara  @Valid ).
