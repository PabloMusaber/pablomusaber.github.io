---
name: Blog con Ruby on Rails
tools: [Ruby, Rails, HTML, CSS]
image: ../assets/img/projects/blog_rails/portada2.jpg
description: Plataforma web desarrollada con el objetivo de aprender la tecnología. Permite a usuarios registrarse, comentar y publicar artículos.
---

# Blog con Ruby on Rails<a href="https://github.com/PabloMusaber/blog-rails" style="color: #6c757d" onMouseOver="this.style.color='#333333'" onMouseOut="this.style.color='#6c757d'" target="githubWindow"><i class="fab fa-github"></i></a>

Este proyecto es muy sencillo, y fue desarrollado para aprender y practicar con **Ruby on Rails**. Utiliza las versiones:

- **Ruby**: 2.7.0
- **Rails**: 6.1.4

Además, las gemas utilizadas son:

- **simple_form**: Simplifica la creación de formularios y la visualización de errores en los campos correspondientes.
- **devise**: Provee autenticación de usuarios con registro, inicio de sesión, recuperación de contraseñas, manejo de sesiones y eliminación de cuentas.
- **bootstrap**: Framework de diseño.
- **font-awesome-sass**: Provee una amplia colección de iconos, utilizados principalmente en los botones de la aplicación.
- **wicked_pdf**: Permite la generación de archivos PDF a partir de vistas HTML.
- **rubocop**: Analizador y formateador de código Ruby.

<br>

Al levantar el proyecto, y luego de iniciar sesión, el usuario puede observar la lista de sus artículos creados en forma de tarjetas. Se aprecia el título, las categorías a las que pertenece dicho artículo, la fecha de creación y la foto de perfil del autor.
![blog-rails](../assets/img/projects/blog_rails/blog1.jpg)
<br>

El presioanr sobre el botón de "Exportar artículos como PDF", un usuario puede generar un archivo de dicho formato con todo el listado de sus artículos publicados.
![blog-rails](../assets/img/projects/blog_rails/blog2.jpg)
<br>

Además, el autor de un artículo es el único usuario capaz de editar o eliminar dicho elemento, y por ende es el único capaz de visualizar los botones que permiten estas funciones.
![blog-rails](../assets/img/projects/blog_rails/blog3.jpg)
<br>

Los artículos pueden ser comentados por cualqueir usuario, como se observa a continuación.
![blog-rails](../assets/img/projects/blog_rails/blog4.jpg)
<br>

Por último quiero destacar, entre otras funcionalidades, la gestión de la cuenta de usuario, la cual se realiza gracias a la implementación de la gema **Devise**. Permite realizar cambios de datos personales y de foto de perfil, utilizando la contraseña para confirmar los cambios. Además, también facilita las opciones de cambio de contraseña y de eliminación de cuenta.

<script src='https://cdn.jsdelivr.net/gh/eddymens/markdown-external-link-script@v2.0.0/main.min.js'></script>
