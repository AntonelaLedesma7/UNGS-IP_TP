# Introducción a la Programación - primer semestre del 2024.
## Trabajo práctico: galería de imágenes de la NASA 🚀
- **Alves Da Silva, Luis**
- **Ledesma, Antonela**

### Introducción
  - Este archivo tiene como objetivo mostrar el desarrollo del proyecto *fullstack* de la pagina web que nos permite  visualizar y gestionar imágenes de la NASA. El objetivo principal de la página será mostrar una amplia galería de imágenes, con las cuales el usuario podrá interactuar al guardarlas como favoritas. 
  El proyecto se centra en la implementación de varias funcionalidades, incluyendo la carga básica de imágenes desde la API de la NASA, la capacidad de buscar imágenes específicas por medio de un motor de búsqueda integrado, y la implementación de un mecanismo de autenticación -login- que permite a los usuarios guardar y gestionar sus imágenes favoritas de manera personalizada.
  Además, se ha integrado un spinner para mejorar la experiencia del usuario durante la carga de contenido, así como se han aplicado modificaciones visuales utilizando Tailwind CSS para optimizar el diseño de las vistas.
  A lo largo del informe, se detallará el código implementado en cada una de estas funcionalidades. 

### Funcionalidades Implementadas

#### **1. Carga de Imágenes** 

  - Esta vista de la aplicación está contenida en la ruta 'home' de nuestra página, anexada a el apartado de 'Galería' de nuestro header. 

  El código es el siguiente: 
  ```
    def home(request):
      lista_imagenes, lista_favoritos = getAllImagesAndFavouriteList(request)
      images = lista_imagenes
      favourite_list = lista_favoritos
      return render(request, 'home.html', {'images': images, 'favourite_list': favourite_list})
  ```
  El cuál pertenece al archivo `views.py`, utilizando otra función declarada en el mismo archivo: 

  ```
    def getAllImagesAndFavouriteList(request):
    images = services_nasa_image_gallery.getAllImages()# retorna todas las imagenes
    favourite_list = [] #retorna la lista de favoritos, en el caso de no desarrollar ese punto lo dejaremos como lista vacia
    return images, favourite_list
  ```
  La cuál utiliza la función del archivo `services_nasa_image_gallery`: 

  ```
    def getAllImages(input=None):
      json_collection = transport.getAllImages(input)
      images = []
      for object in json_collection:
          nasa_card = mapper.fromRequestIntoNASACard(object)
          images.append(nasa_card)
      return images
  ```
  La misma utiliza funciones ya definidas, desde el archivo `transport` y `mapper`. 

  Hasta este momento, en el desarrollo del proyecto, solo se visualizaban las imagenes con su título y descripción, por defecto buscando el término 'space'. 

  ![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/eb5a46a2-99bb-4b4e-bdcc-fd9b6b45914b)

#### **2. Búsqueda de Imágenes**

  - La implementación de esta funcionalidad requería agregar una función al archivo `views`, utilizando también `getAllImagesAndFavouriteList` y sus respectivas funciones mostradas arriba: 

```
  def search(request):
    images, favourite_list = getAllImagesAndFavouriteList(request)
    search_msg = request.POST.get('query', '')
    if search_msg=="":
        filtered_image = images
    else:
        filtered_image = services_nasa_image_gallery.getImagesBySearchInputLike(search_msg) 
    return render(request, 'home.html', {'images': filtered_image, 'favourite_list': favourite_list, 'search_query': search_msg })
```
  Además, utiliza una función definida previamente, `getImagesBySearchInputLike`, que a su vez utiliza una la función `getAllImages` mostrada en el punto anterior. 

  A partir de este punto, ya era posible visualizar distintas imágenes según el input ingresado al buscador, y en caso de no poseer ninguno la búsqueda sigue siendo por defecto 'space'.
  
![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/ec7f1928-ae86-4933-90de-7bb98ebca9f8)
![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/657b39b3-afcb-4a56-9a30-38a0c5046bae)

#### **3. Spinner de Carga**

  - El spinner de carga requirió un cambio en el template de `home.html`, agregando la etiqueta `<script>`, con código Js, para poder manejar la lógica de carga del navegador. En un principio se modificó para que se muestre en la vista que estaba predeterminada, y luego con el cambio de estilos sufrió unos cambios. Se adjunta el código como fueron modificados: 

  ```
    <div class="image-container" style="position: relative;">
      <img src="{% static 'images/loading.gif' %}" class="loading-gif" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); display: block;" alt="Loading...">
      <img src="{{ imagen.image_url }}" class="card-img-top nasa-image" alt="imagen" style="display: none;">
    </div>

    <script>
    document.addEventListener("DOMContentLoaded", function() {
        const images = document.querySelectorAll('.nasa-image');
        images.forEach((img) => {
            img.addEventListener('load', function() {
                const loadingGif = img.previousElementSibling;
                loadingGif.style.display = 'none';
                img.style.display = 'block';
            });

            img.addEventListener('error', function() {
                const loadingGif = img.previousElementSibling;
                loadingGif.style.display = 'none';
                img.style.display = 'block';
                img.src = '{% static 'images/placeholder.png' %}'; // Imagen de respaldo en caso de error
            });
        });

        const searchForm = document.getElementById('search-form');
        searchForm.addEventListener('submit', function() {
            const loadingGifs = document.querySelectorAll('.loading-gif');
            loadingGifs.forEach(function(gif) {
                gif.style.display = 'block';
            });

            const nasaImages = document.querySelectorAll('.nasa-image');
            nasaImages.forEach(function(img) {
                img.style.display = 'none';
            });
        });

        // Mostrar el spinner de carga inicialmente
        const initialLoadingGifs = document.querySelectorAll('.loading-gif');
        initialLoadingGifs.forEach(function(gif) {
            gif.style.display = 'block';
        });
    });
    </script>
  ```
  Luego de la modificación de estilos: 

  ```
    <div class="w-full h-64 bg-white flex items-center justify-center relative">
      <img src="{% static 'images/loading.gif' %}" class="loading-gif" alt="Loading...">
    </div>
    <img src="{{ imagen.image_url }}" class="w-full h-64 object-cover nasa-image hidden" alt="imagen">

    <script>
    document.addEventListener("DOMContentLoaded", function() {
        const images = document.querySelectorAll('.nasa-image');

        images.forEach((img) => {
            img.addEventListener('load', function() {
                const parent = img.parentElement;
                const loadingDiv = parent.querySelector('.w-full.h-64.bg-white');
                loadingDiv.style.display = 'none';
                img.classList.remove('hidden');
            });

            img.addEventListener('error', function() {
                const parent = img.parentElement;
                const loadingDiv = parent.querySelector('.w-full.h-64.bg-white');
                loadingDiv.style.display = 'none';
                img.src = "{% static 'images/placeholder.png' %}";
                img.classList.remove('hidden');
            });

            img.src = img.src;
        });

        const searchForm = document.getElementById('search-form');
        searchForm.addEventListener('submit', function() {
            const loadingDivs = document.querySelectorAll('.w-full.h-64.bg-white');
            loadingDivs.forEach(function(div) {
                div.style.display = 'flex';
            });

            const nasaImages = document.querySelectorAll('.nasa-image');
            nasaImages.forEach(function(img) {
                img.classList.add('hidden');
            });
        });
    });
    </script>
  ```

  Una vez modificado, el spinner de carga aparece en cada cuadro donde estará la imagen que se va a mostrar.

  ![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/a9cf1eb8-aae9-4816-9e7f-dfa54d25aebf)

#### **4. Inicio de Sesión**

  - Para implementar esa funcionalidad, se modificaron varios archivos, en primer lugar se modificaron las urls de la carpeta `main`: 

  ```
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('nasa_image_gallery.urls')),
        path('accounts/',include('django.contrib.auth.urls')) 
    ]
  ```

  Agregando, `accounts` de django para la gestión de autenticación. Además, al template de `login.html` se le agrego al formulario la ruta correspondiente: 

  ```action="{% url 'login' %}"```

  Del mismo modo, se agregó en el archivo `header.html` la referencia correspondiente al 'Iniciar Sesión' del mismo: 

  ```href="{% url 'login' %}"```

  Por último, se agregó la función que permite al usuario salir de la sesión: 

  ```
    @login_required
    def exit(request): #la funcion cierra sesion y redirige al usuario a la pagina principal
      logout(request)
      return redirect ('/')
  ```

  Hasta este punto de desarrollo, la página web ya tenía disponible el login de usuario, lo cuál modificaba las vistas, ya que en nuetra vista de inicio aparecía el nombre del usuario, y en la galería de imagenes nos aparecía el botón para añadir a favoritos, aunque aún sin funcionalidad. 

  ![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/3434a24a-8cc2-4950-91c5-4b82b985181c)

  ![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/268dd5f7-4770-4834-8ae6-05e03257e58d)

  ![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/c0cb240c-a9d3-4317-8558-e641c92889d3)

#### **5. Favoritos** 

  - Para llevar a cabo esta funcionalidad se desarrollaron funciones que estaban nombradas en el archivo ``views.py`: 

  En un principio se modificó la función `getAllImagesAndFavouriteList`: 

    ```
      def getAllImagesAndFavouriteList(request):
        images = services_nasa_image_gallery.getAllImages()# retorna todas las imagenes
        favourite_list = services_nasa_image_gallery.getAllFavouritesByUser(request) #retorna la lista de favoritos
        return images, favourite_list
    ```
  Dicha función toma la lista de favoritos, para poder mostrar en pantalla aquellas que ya han sido seleccionadas por el usuario logueado, como tal, modificando el botón 'Ya está añadida a favoritos'. 

  Luego, se modificaron las funciones que permiten al usuario agregar un favorito como también eliminarlo: 

  ```
    @login_required
    def saveFavourite(request):
      services_nasa_image_gallery.saveFavourite(request)
      return redirect ('home')


    @login_required
      def deleteFavourite(request):
      services_nasa_image_gallery.deleteFavourite(request)
      return redirect ('home')
  ```

  
  Por último, en esta implementación, se agregó el listado de favoritos con la siguiente función en el archivo `views.py`: 

  ```
    @login_required
    def getAllFavouritesByUser(request):
      favourite_list = services_nasa_image_gallery.getAllFavouritesByUser(request)
      return render(request, 'favourites.html', {'favourite_list': favourite_list})
  ```
  Se completaron las siguientes funciones en `services_nasa_image_gallery.py`:

  ```
    def saveFavourite(request):
      fav = mapper.fromTemplateIntoNASACard(request)  # transformamos un request del template en una NASACard.
      fav.user = request.user # le seteamos el usuario correspondiente.
      return repositories.saveFavourite(fav) # lo guardamos en la base.

    # usados en el template 'favourites.html'
    def getAllFavouritesByUser(request):
      if not request.user.is_authenticated:
        return []
      else:
        user = get_user(request)

        favourite_list = repositories.getAllFavouritesByUser(user) # buscamos desde el repositorio TODOS los favoritos del usuario (variable 'user').
        mapped_favourites = []

        for favourite in favourite_list:
            nasa_card = mapper.fromRepositoryIntoNASACard(favourite) # transformamos cada favorito en una NASACard, y lo almacenamos en nasa_card.
            mapped_favourites.append(nasa_card)

        return mapped_favourites
  ```
  Hasta acá, el cliente de la web tiene la posibilidad de ingresar con el usuario 'ADMIN', ver la galería de imágenes y realizar la búsqueda que desee, además de tener la posibilidad de agregar como favoritos a aquellos que desee, y poder verlos en una lista con más detalles como la fecha de dicha imagen, para también desde ahí poder eliminarlos de esa lista. 

![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/1483714e-cb69-4025-8227-adec3eed6c76)

![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/cb5985c3-f5f4-47cf-96fc-c68760e45991)

#### **6. Renovar interfaz gráfica**

  - Como última implementacíon, se llevo a cabo el cambio en la interfaz gráfica utilizando Tailwind CSS como medio para lograrlo, para eso se utilizó dicho framework desde un CDN para simplificar la configuración del mismo, agregado al `<head>` del archivo `header.html`

  ```
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  ```

  Luego, en cada template, se ajustaron los contenidos de los atributos `class` de cada elemento para modificar su posición, tamaño y color, según lo requerido en cada vista. El siguiente es un ejemplo de la modificación aplicada al `<body>` del `header.html`:

  ```
    <body>
      <nav class="bg-indigo-900 p-4">
        <div class="container mx-auto flex justify-between items-center">
            <a class="text-white text-xl font-bold hover:text-indigo-200" href="{% url 'index-page' %}">Proyecto TP</a>
            <button class="text-white md:hidden" onclick="toggleMenu()">
                <svg class="navbar-toggler hover:bg-indigo-200 w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16m-7 6h7"></path>
                </svg>
            </button>
            <div id="menu" class="hidden md:flex space-x-4">
                <a class="block px-4 py-2 mt-2 text-sm font-semibold text-white bg-transparent rounded-lg hover:text-indigo-200 md:inline md:mt-0 md:text-base md:bg-transparent md:rounded md:hover:text-indigo-200" href="{% url 'index-page' %}">Inicio</a>
                <a class="block px-4 py-2 mt-2 text-sm font-semibold text-white bg-transparent rounded-lg hover:text-indigo-200 md:inline md:mt-0 md:text-base md:bg-transparent md:rounded md:hover:text-indigo-200"  href="{% url 'home' %}">Galería</a>
                {% if request.user.is_authenticated %}
                <a class="block px-4 py-2 mt-2 text-sm font-semibold text-white bg-transparent rounded-lg hover:text-indigo-200 md:inline md:mt-0 md:text-base md:bg-transparent md:rounded md:hover:text-indigo-200"  href="{% url 'favoritos' %}">Favoritos</a>
                <!-- Agregamos la ruta para que el boton salir te saque de la sesion -->
                <a class="block px-4 py-2 mt-2 text-sm font-semibold text-white bg-transparent rounded-lg hover:text-red-700 hover:underline md:inline md:mt-0 md:text-base md:bg-transparent md:rounded md:hover:text-red-700 md:hover:underline"  href="{% url 'salir' %}">Salir</a>

                {% else %}
                <!-- Agregamos la ruta hacia la pagina de logueo en el boton iniciar sesion -->
                <a class="block px-4 py-2 mt-2 text-sm font-semibold text-white bg-transparent rounded-lg hover:text-indigo-200 md:inline md:mt-0 md:text-base md:bg-transparent md:rounded md:hover:text-indigo-200"  href="{% url 'login' %}">Iniciar sesión</a>

                {% endif %}
            </div>
        </div>
      </nav>

      {% block content %} {% endblock %}

      {% include "footer.html" %}
    </body>
  ```

![image](https://github.com/Luis-Alves-Da-Silva/TP-IP-Alves-Da-Silva_Ledesma/assets/128189587/464c6080-3b03-4d3a-9070-caa1a984ea14)
