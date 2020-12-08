### Testing MVC Applications

Una prueba unitaria es un procedimiento que instancia un componente que ha escrito, llama a un aspecto de las funcionalidades del componente y verifica que el componente responde correctamente de acuerdo con el diseño. En la programación orientada a objetos, las pruebas unitarias suelen crear una instancia de una clase y llamar a uno de sus métodos. En una aplicación web ASP.NET Core MVC, las pruebas unitarias pueden crear instancias de clases de modelo o controladores y llamar a sus métodos y acciones. Como desarrollador web, necesita saber cómo crear un proyecto de prueba, agregar pruebas al proyecto de prueba y ejecutar pruebas en el proyecto de prueba. Las pruebas verifican aspectos individuales de su código sin depender de servidores de bases de datos, conexiones de red y otra infraestructura. Esto se debe a que las pruebas unitarias se centran en el código que escribe.


Una prueba de una sola unidad generalmente consta de código que se ejecuta en tres fases:

1. Arrange. En esta fase, la prueba crea una instancia de la clase que probará. También asigna las propiedades necesarias y crea los objetos necesarios para completar la prueba. Solo se crean propiedades y objetos que son esenciales para la prueba.

2. Act. En esta fase, la prueba ejecuta la funcionalidad que debe verificar. Por lo general, en la fase Act, la prueba llama a un solo procedimiento y almacena el resultado en una variable.

3. Assert. En esta fase, la prueba compara el resultado con el resultado esperado. Si el resultado coincide con lo esperado, la prueba pasa. De lo contrario, la prueba falla.

Para ello:
Agregar y configurar un proyecto de prueba
En Visual Studio, puede probar un proyecto de aplicación web ASP.NET Core MVC agregando un nuevo proyecto a la solución, basado en la plantilla MSTest Test Project (.NET Core). Debe agregar una referencia del proyecto de prueba al proyecto de la aplicación web MVC para que el código de prueba pueda acceder a las clases en el proyecto de la aplicación web MVC. También debe instalar el paquete Microsoft.AspNetCore.Mvc en el proyecto de prueba.

En un proyecto de prueba de Visual Studio MSTest, los accesorios de prueba son clases marcadas con el atributo TestClass. Las pruebas unitarias son métodos marcados con el atributo TestMethod. Las pruebas unitarias generalmente devuelven void pero llaman a un método de la clase Assert, como Assert.AreEqual para verificar los resultados en la fase de prueba Assert.


El siguiente código es un ejemplo de una prueba para el modelo de Producto creado anteriormente:

Prueba de una clase de modelo

<pre> <code>
[TestClass]
public class ProductTest
{
    [TestMethod]
    public void TaxShouldCalculateCorrectly()
    {
        // Arrange
        Product product = new Product();
        product.BasePrice = 10;
 
        // Act
        float result = product.GetPriceWithTax(10);
 
        // Assert
        Assert.AreEqual(11, result);
    }
}
</code> </pre>



Crear un Repository Service
Un repositorio es un patrón de diseño común que se utiliza para separar el código de lógica empresarial de la recuperación de datos. Esto se hace comúnmente creando una interfaz de repositorio que define propiedades y métodos que MVC puede usar para recuperar datos. Por lo general, el repositorio maneja las diversas operaciones CRUD (crear, leer, actualizar y eliminar) en los datos. En las aplicaciones ASP.NET Core MVC, el repositorio se maneja creando un servicio. En general, querrá un repositorio por entidad. Los repositorios son muy útiles para probar controladores porque los modelos generalmente no usan datos externos.

El siguiente código es un ejemplo de una interfaz de repositorio:

Una interfaz de repositorio
<pre> <code>
public interface IProductRepository
{
    IQueryable<Product> Products { get; }
    Product Add(Product product);
    Product FindProductById(int id);
    Product Delete(Product product);
}
</code> </pre>



Implementación y uso de un repositorio en la aplicación
La interfaz del repositorio define métodos para interactuar con los datos, pero no determina cómo se configurarán y almacenarán esos datos. Debes proporcionar dos implementaciones del repositorio:

- Una implementación de servicio del repositorio que se utilizará dentro de la aplicación. Esta implementación utilizará datos de la base de datos o algún otro mecanismo de almacenamiento.

- Una implementación del repositorio para su uso en pruebas. Esta implementación utilizará datos del conjunto de memoria durante la fase de Arreglo de cada prueba.

El siguiente ejemplo de código muestra una implementación de servicio de un repositorio:

Implementación de un repositorio en el proyecto ASP.NET Core MVC
<pre> <code>
public class ProductRepository : IProductRepository
{
    StoreContext _store;
    public ProductRepository(StoreContext store)
    {
        _store = store;
    }
 
    public IQueryable<Product> Products
    {
        get { return _store.Products; }
    }
 
    public Product Add(Product product)
    {
        _store.Products.Add(product);
        _store.SaveChanges();
        return _store.Products.Find(product);
    }
 
    public Product Delete(Product product)
    {
        _store.Products.Remove(product);
        _store.SaveChanges();
        return product;
    }
 
    public Product FindProductById(int id)
    {
        return _store.Products.First(product => product.Id == id);
    }
}
</code> </pre>


Implementación de un doble de prueba de repositorio
La segunda implementación de la interfaz del repositorio es la implementación que utilizará en las pruebas unitarias. Esta implementación utiliza datos en memoria y una colección de objetos con clave para funcionar como un contexto de Entity Framework, pero evita trabajar con una base de datos.

El siguiente ejemplo de código muestra cómo implementar una clase de repositorio para pruebas:
Implementing a Fake Repository

<pre> <code>
public class FakeProductRepository : IProductRepository
{
    private IQueryable<Product> _products;
 
    public FakeProductRepository()
    {
        List<Product> products = new List<Product>();
        _products = products.AsQueryable();
    }
 
    public IQueryable<Product> Products
    {
        get { return _products.AsQueryable(); }
        set { _products = value; }
    }
 
    public Product Add(Product product)
    {
        List<Product> products = _products.ToList();
        products.Add(product);
        _products = products.AsQueryable();
        return product;
    }
 
    public Product Delete(Product product)
    {
        List<Product> products = _products.ToList();
        products.Remove(product);
        _products = products.AsQueryable();
        return product;
    }
 
    public Product FindProductById(int id)
    {
        return _products.First(product => product.Id == id);
    }
}
</code> </pre>

Ejecución de pruebas unitarias

Una vez que escribe una prueba, puede ejecutarla haciendo clic con el botón derecho en una clase de prueba y luego seleccionando la opción Ejecutar prueba (s) en el menú contextual. Alternativamente, puede abrir el explorador de prueba desde el menú de prueba dentro de las subopciones de Windows. Desde el explorador del ejecutor de pruebas, puede elegir clases o métodos de prueba específicos y ejecutarlos según sea necesario. También hay opciones para ejecutar todas las pruebas, o ejecutar pruebas en modo de depuración, lo que le permite observar fallas específicas y manejarlas en consecuencia.


En este ejemplo, puede ver una prueba unitaria que aprovecha las fortalezas de un marco simulado. A diferencia de la prueba del controlador anterior, en esta prueba, no es necesario crear un doble de prueba para IProductRepository. En cambio, al usar el objeto simulado expuesto por el marco de Moq, puede configurar una clase falsa que devuelva una lista de productos que definió.
