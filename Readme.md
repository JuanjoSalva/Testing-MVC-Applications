### Testing MVC Applications


En este ejemplo, puede ver una prueba unitaria que aprovecha las fortalezas de un marco simulado. A diferencia de la prueba del controlador anterior, en esta prueba, no es necesario crear un doble de prueba para IProductRepository. En cambio, al usar el objeto simulado expuesto por el marco de Moq, puede configurar una clase falsa que devuelva una lista de productos que definió.
