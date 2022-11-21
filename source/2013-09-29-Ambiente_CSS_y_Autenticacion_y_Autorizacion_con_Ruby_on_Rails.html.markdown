---
title: Ambiente_CSS_y_Autenticacion_y_Autorizacion_con_Ruby_on_Rails
date: 2013-09-29 12:00 UTC
tags:
---
Continuando {1}, presentamos como emplear en una aplicación Ruby on Rails 4.0 sobre adJ 5.4:

* Bootstrap de Twitter como ambiente CSS
* Devise para autenticar (verificar que un usuario es quien dice ser)
* !CanCan para autorizar (dar acceso a un usuario a lo que corresponde según su rol)

# 1. Ambiente CSS

El procedimiento para emplear una gema que brinde cierta funcionalidad a una aplicación típicamente es:
* Instalar y configurarla
** Instalar la gema con ```gem install``` para que todos los generados estén disponibles
** Agregar la gema al archivo ```Gemfile```  de la aplicación
** Ejecutar bundle update para actualizar paquetes e instalar nueva gema.  
* Generar archivos
** Iniciales
** Uno por cada modelo, controlador o vista dependiendo de la gema.
* Modificar archivos generados

En el caso de Bootstrap de Twitter como se explica en {2}, puede usarse con less para generar CSS bastante dinámicos (que requiere rubyracer que es problemático) o puede usarse con CSS estáticos pero compilados con SASS que es la configuración sugerida a continuación

##1.1 Instalación y configuración

* Ejecutar
<pre>
sudo gem install twitter-bootstrap-rails
sudo gem install bootstrap-sass
</pre>
* Incluir en ```aplicacion/Gemfile```
<pre>
gem "twitter-bootstrap-rails"
gem "bootstrap-sass"
</pre>
* Actualizar
<pre>
sudo bundle20 update
</pre>

##1.2 Generación de archivos

* Generar archivos estáticos CSS que usan por defecto los íconos de Font Awesome (ver {3}):
<pre>
rails generate bootstrap:install static
</pre>
* Generar una plantilla base (layout) para su sitio, la plantilla por defecto se llamará application queda en ```app/views/layouts/application.html.erb```  y será adaptativa (a computadores o smarphones o tabletas) en contraste con fija (sería con ```fixed``` en lugar de ```fluid```):
<pre>
rails g bootstrap:layout application fluid
</pre>
* Generar recurso ```Pais```, generando modelo (con una migración) y vistas que usan Bootstrap  --tenga en cuenta que para este ejemplo se requiere corregir la inflección plural como se explica en la sección 4.3 de {1}):
<pre>
rails g scaffold Pais nombre:string{500} latitud:float longitud:float
rake db:migrate
rails g bootstrap:themed Paises
</pre>

##1.3 Modificar archivos generados

!1.3.1 Localización

Las vistas generadas con el procedimiento descrito, en Rails 4.0.0 no están localizadas por lo que se recomienda seguir 4.3 de {1} y además:

1. Permitir jerarquía de directorios para localización en ```config/locales``` (como se recomienda en {4}, agregando a ```config/application.rb```:
<pre>
config.i18n.load_path += Dir![Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
</pre>

2. Editar las vistas y modificar las cadenas por localizar para convertirlas en variables como argumentos de la función ```I18n.t``` (o más breve ```t```).
Por ejemplo la vista ```app/views/paises/index.html.erb``` incluye:
<pre>
<h1>Listing paises</h1>
...
<td><%= link_to 'Destroy', pais, method: :delete, data: { confirm: 'Are you sure?' } %></td>
...
<%= link_to 'New Pais', new_pais_path %>
</pre>
líneas que se convierten a:
<pre>
<h1><%= t :listing %> paises</h1>
...
<td><%= link_to t(:destroy), pais, method: :delete, data: { confirm: t(:are_you_sure) } %></td>
...
<%= link_to t(:new)  + ' Pais', new_pais_path %>
</pre>

3. Crear traducciones para las cadenas empleando las variables introducidas en archivos localizados en ```config/locales``` por ejemplo para este ejemplo se recomienda crear ```config/locales/views/es.yml``` con el siguiente contenido:
<pre>
es:
  listing: "Listado de"
  show: "Mostrar"
  edit: "Editar"
  destroy: "Eliminar"
  are_you_sure: "¿Está seguro(a)?"
  new: "Nuevo"
</pre>

Al hacer esto podrá verse por completo en español la página ```http://127.0.0.1/paises/```


# 2. Autenticación

Suponiendo que tiene una nueva aplicación como se describe en sección 4.1 de {1}, 

##2.1 Instalar

<pre>
sudo gem install devise
sudo gem install devise-i18n
</pre>

En Gemfile agregue: 
<pre>
gem 'devise'
gem 'devise-i18n'
</pre>

Y ejecute
<pre>
sudo bundle20 update
</pre>

##2.2 Generar archivos

Ejecute 
<pre>
rails generate devise:install
</pre>
Para generar archivo de configuración ```config/initializer/devise.rb``` y localización inicial en in&#501;les (en otros idiomas será proveida por devise-i18n) en ```config/locales/devise.en.yml```


Ejecutando
<pre>
rails g devise:views
</pre>
 
genera archivos compartidos
     
| app/views/devise/shared/_links.erb  |

y formularios iniciales

| __Archivo__ | __Descripción__ |
|app/views/devise/confirmations/new.html.erb | Volver a enviar instrucciones de confirmación |
|app/views/devise/passwords/edit.html.erb | Cambiar clave |
|app/views/devise/passwords/new.html.erb | Reasigna clave cuando usuario olvida |
|app/views/devise/registrations/edit.html.erb | Editar registro (clave, correo, cancelar) |
|app/views/devise/registrations/new.html.erb| Registro nuevo usuario |
|app/views/devise/sessions/new.html.erb| Inicio de sesión |
|app/views/devise/unlocks/new.html.erb| Volver a enviar instrucciones para desbloquear |
|app/views/devise/mailer/confirmation_instructions.html.erb| Correo para confirmar cuenta |
|app/views/devise/mailer/reset_password_instructions.html.erb| Corre para cambiar clave |
|app/views/devise/mailer/unlock_instructions.html.erb| Correo para informar bloqueo |


Después genere tabla, modelo, controlador y vistas para manejar usuarios, puede comenzar con:

<pre>
rails g devise usuario
</pre>

que genera migración para crear tabla con nombre ```usuario```, modelo inicial y pruebas:

| __Archivo__ | __Descripción__  |
| db/migrate/20130929042748_devise_create_usuario.rb | Migración para crear tabla user |
| app/models/usuario.rb | Modelo para tabla user|
| test/models/usuario_test.rb | Pruebas |
| test/fixtures/usuarios.yml | Datos para pruebas |

Este procedimiento también agregara automáticamente rutas, agregando la línea ```devise_for: usuarios``` en ```config/routes.rb```

Aplique la migración para crear la tabla usuario con:

<pre>
rake db:migrate
</pre>

Tras esto, e iniciar el servidor podrá registrar un nuevo usuario desde ```http://127.0.0.1/usuario/sign_up/```
El nuevo usuario iniciará sesión de inmediato o puede iniciar sesión desde: ```http://127.0.0.1/usuario/sign_in/```.  Por defecto Devise almacena claves empleando bcrypt, si en su caso debe emplear métodos menos seguros configure en ```config/initializers/devise.rb``` y elimine la dependencia en la gema ```bcrypt```.

##2.3 Modificación de archivos y configuración

Para cerrar una sesión con ```http://127.0.0.1/sign_out/``` primero debe asociarse la ruta ```sign_out``` de usuarios autenticados con el método ```destroy``` del controlador ```session``` de devise.  Se logra editando ```config/routes.rb``` y agregando:
<pre>
devise_scope :usuario do
  get 'sign_out' => 'devise/sessions#destroy'
end
</pre>

Si queremos autorizar de forma muy simple (sin emplear CanCan) por ejemplo puede asegurarse que sólo usuario autenticados ingresen a una vista (digamos ```home```), agregando al comienzo de su controlador (e.g ```app/controller/hogar_controller.rb```) la línea:
<pre>
before_filter :authenticate_usuario!
</pre>

Rápidamente obtuvimos una aplicación con autenticación y registro de usuarios que son los módulos por defecto de devise.  Otros módulos se pueden configurar en ```app/models/usuario``` :
<pre>
# Include default devise modules. Others available are:
# :confirmable, :lockable, :timeoutable and :omniauthable
devise :database_authenticatable, :registerable,
       :recoverable, :rememberable, :trackable, :validatable
</pre>
la tabla usuario generada con la migración para los módulos tiene los siguientes campos:

| __Campo__ | __Tipo__ | __Atributos__ |
| id                     | integer                     | not null default nextval('usuarios_id_seq'::regclass) |
| email                  | character varying(255)      | not null default ''::character varying |
| encrypted_password     | character varying(255)      | not null default ''::character varying |
| reset_password_token   | character varying(255)      | |
| reset_password_sent_at | timestamp without time zone | |
| remember_created_at    | timestamp without time zone | |
| sign_in_count          | integer                     | not null default 0 |
| current_sign_in_at     | timestamp without time zone | |
| last_sign_in_at        | timestamp without time zone | |
| current_sign_in_ip     | character varying(255)      | |
| last_sign_in_ip        | character varying(255)      | |
| created_at             | timestamp without time zone | |
| updated_at             | timestamp without time zone | |


# 3. Autorización con !CanCan y roles con ```role_model```

Supongamos que quisieramos manejar usuarios con roles, por lo menos los roles usuario y administrador.  Y que queremos que el rol administrador pueda listar, crear , eliminar y editar usuarios mientras que el rol usuario no pueda hacerlo.   

## 3.1. Instalación

<pre>
sudo gem install cancan
sudo gem install role_model
</pre>

Agregar a ```Gemfile```:
<pre>
gem "cancan"
gem "role_model"
</pre>

## 3.2. Configuración

Podemos comenzar generando la clase ```app/models/ability.rb``` donde !CanCan centraliza autorización con:
<pre>
rails g cancan:ability
</pre>

Cancan por defecto requiere un dato ```current_user```, pero como nuestro modelo para usuarios es de nombre ```usuario```, Devise proporciona ```current_usuario```, así que debemos sobrecargar un método de !CanCan modificando la clase !ApplicationController (ubicada en ```app/controllers/application_controller.rb```) para agregar el siguiente método:
<pre>
  def current_ability
    @current_ability ||= Ability.new(current_usuario)
  end
</pre>


Para manejar roles, la gema ```role_model``` identifica cada rol posible con una potencia de dos diferente y los roles de un usuario como la suma de los roles que tiene.  Por esto debemos agregar un campo ```roles_mascara``` de tipo entero a la tabla Usuarios, puede hacerse con:
<pre>
rails g migration !AddRolToUsuario roles_mascara:integer
rake db:migrate
</pre>

además se requiere modificar el modelo Usuario para agregar métodos proveidos por ```role_model```, identificar el campo que tiene la mascara de roles y los roles posibles, para esto edite ```app/models/usuario.rb``` y agregue en medio de la clase ```Usuario```:
<pre>
  include !RoleModel
 
  roles_attribute :roles_mascara
 
  roles :admin, :usuario
</pre>

Generamos controlador para el modelo usuario y vistas para operaciones básicas con:
<pre>
rails g scaffold_controller usuario  nombre:string email:string encrypted_password:string \
reset_password_token:string reset_password_sent_at:datetime \
remember_created_at:datetime sign_in_count:integer \
current_sign_in_at:datetime last_sign_in_at:datetime \
current_sign_in_ip:string last_sign_in_ip:string roles_mascara:integer
rails g bootstrap:themed usuario
</pre>

que produce:

| __Archivo__ | __Descripción__ |
|app/controllers/usuarios_controller.rb| Controlador |
|app/views/usuarios/index.html.erb| Listado de usuarios |
|app/views/usuarios/edit.html.erb| Edita un usuario |
|app/views/usuarios/show.html.erb| Muestra un usuario |
|app/views/usuarios/new.html.erb| Nuevo usuario |
|app/views/usuarios/_form.html.erb| Formulario usado para editar y nuevo |
|test/controllers/usuarios_controller_test.rb| Pruebas |
|app/helpers/usuarios_helper.rb| Funciones auxiliares |
|test/helpers/usuarios_helper_test.rb| Pruebas a funciones auxiliares|
|app/views/usuarios/index.json.jbuilder| Listado JSON |
|app/views/usuarios/show.json.jbuilder| Muestra un en JSON |

Tras esto podemos indicar a !CanCan "quien tiene permiso para hacer que", modificando la función ```initialize(user)``` de ```app/models/ability.rb``` agregando:
<pre>
 if !user.nil? && user.has_rol?(:usuario) then
    can :read, :all
  elsif !user.nil? && user.has_rol?(:admin) then
    can :manage, :all
  end
</pre>

Con lo cual especificamos que los usuarios con rol ```:usuario``` pueden leer todo recurso, y que los que tengan rol ```:admin``` pueden hacer cualquier operación en todo recurso.  La función ```can``` recibe el permiso que se otorga y la clase a la cual se le otorga (los permisos predefinidos son ```:create```, ```:read```, ```:update```, ```:destroy``` y ```:manage``` que incluye cualquier otro).  Es posible definir nuevos permisos y filtrar más los objetos sobre los que se aplican (ver {7}).

Falta sólo indicar al controlador de Usuario que antes de realizar operaciones tenga en cuenta autorización, se hace agregando a la clase !UsuariosController (de ```app/controllers/usuarios_controller.rb```):
<pre>
load_and_authorize_resource
</pre>

Tras esto podrá probar y comprobar que el listado de usuarios ```http://127.0.0.1:3000/usuarios/``` sólo puede ser examinado por usuarios con el rol ```:admin```.  Para poner este rol a un usuario ponga 2 en el campo ```roles_mascara``` del registro del usuario en la tabla ```usuarios```, por ejemplo:
<pre>
rails dbconsole
UPDATE usuarios SET roles_mascara=2 WHERE email='correoadmin@miejemplo.org';
</pre>


#3. Referencias
* {1} [Ruby on Rails en OpenBSD]
* {2} https://github.com/seyhunak/twitter-bootstrap-rails
* {3} http://fortawesome.github.io/Font-Awesome/icons/
* {4} http://guides.rubyonrails.org/i18n.html
* {5} https://github.com/ryanb/cancan/wiki/changing-defaults
* {6} https://github.com/martinrehfeld/role_model
* {7} https://github.com/ryanb/cancan/wiki/Defining-Abilities

----
__Vive Jehova__
