---
title: Ruby_on_Rails_en_OpenBSD
date: 2012-07-03 12:00 UTC
tags:
---

-----
# ADVERTENCIA

Esta información se ha incorporado a la documentación de adJ, puede encontrarla más actualizada y completa en:

http://pasosdejesus.github.io/usuario_adJ/conf-programas.html#ruby

-----


En adJ 6.0 es sencillo usar ruby 2.4 con Ruby on Rails 5.  Lo básico se instala de paquetes de OpenBSD y lo más reciente en Ruby directamente como gemas.

# 1. Instalación y configuración

Asegúrese de tener instalados los paquetes  ruby 2.4.0, libv8 y node, incluidos en el DVD de adJ 6.0

Asegúrese de tener enlaces al interprete de ruby y herramientas (como describe el paquete ruby):
<pre>
doas sh
 ln -sf /usr/local/bin/ruby24 /usr/local/bin/ruby
 ln -sf /usr/local/bin/erb24 /usr/local/bin/erb
 ln -sf /usr/local/bin/irb24 /usr/local/bin/irb
 ln -sf /usr/local/bin/rdoc24 /usr/local/bin/rdoc
 ln -sf /usr/local/bin/ri24 /usr/local/bin/ri
 ln -sf /usr/local/bin/rake24 /usr/local/bin/rake
 ln -sf /usr/local/bin/gem24 /usr/local/bin/gem
</pre>

## 1.1. Límites amplios 

Asegurarse de tener limites amplios del sistema operativo para abrir
archivos y manejar memoria, por ejemplo superiores a los siguientes 
en ```/etc/systctl.conf```
<pre>
kern.shminfo.shmmni=1024
kern.seminfo.semmns=2048
kern.shminfo.shmmax=50331648
kern.shminfo.shmall=51200
kern.maxfiles=20000
</pre>

El usuario desde el cual desarrollará o ejecutará aplicaciones también
debe tener límites amplios, en particular su clase de login (por ejemplo 
```staff```).  Debe tener al menos los siguientes en ```/etc/login.conf```
<pre>
staff:\
        :datasize-cur=1536M:\
        :datasize-max=infinity:\
        :maxproc-max=4096:\
        :maxproc-cur=4096:\
        :openfiles-max=8090:\
        :openfiles-cur=8090:\
        :ignorenologin:\
        :requirehome@:\
        :tc=default:
</pre>
(Si modifica el archivo ```/etc/login.conf``` debe reconstruir su versińo binaria 
con  ```doas cap_mkdb /etc/login.conf```).

## 1.2. irb

Para facilitar su exploración del lenguaje ruby, puede usar ```irb``` (ver {4}),
pero antes verifique que su archivo ```~/.irbrc``` tenga las siguientes 
líneas (añadidas por defecto en adJ a la cuenta de administrador):
<pre>
# Configuración de irb
# Basado en archivo de comandos disponible en <http://girliemangalo.wordpress.com/2009/02/20/using-irbrc-file-to-configure-your-irb/>
require 'irb/completion'
require 'pp'
IRB.conf[:AUTO_INDENT] = true
IRB.conf[:USE_READLINE] = true

def clear
    system('clear')
end
</pre>

A continuación ingrese a irb y escriba  por ejemplo ```4.``` y presione la tecla [Tab] 2 veces para ver los métodos de la clase Integer.

## 1.3. Gemas

El paquete ```ruby``` incluye ```rubygems``` que maneja gemas (es decir
librerías) con el programa ```gem```. Puede actualizar a la versión 
más reciente las gemas globales con:

<pre>
doas gem update --system
QMAKE=qmake-qt5 make=gmake MAKE=gmake doas gem pristine --all
</pre>

Para facilitar compilación de algunas extensiones (como las de nokogiri) se recomienda instalar globalmente:
<pre>
doas gem install pkg-config 
</pre>

El directorio donde se instalan las gemas globales es ```/usr/local/lib/ruby/gems/2.4/``` donde sólo pueden
instalarse con ```doas```. Recomendamos iniciar un directoio para instalar gemas como usuario normal en  ```/var/www/bundler/ruby/2.4```, por 3 razones (1) evitar riesgos de seguridad al instalar gemas como root, (2) evitar problemas de permisos y la dificultad de programas como bundler para usar ```doas``` en lugar de ```sudo``` y (3) alista infraestructura para que sus aplicaciones corran en jaula chroot en ```/var/www```

Prepare ese directorio con:
<pre>
doas mkdir -p /var/www/bundler/ruby/2.4/
doas chown -R $USER:www /var/www/bundler
</pre>

Y cuando requiera instalar una gema allí emplee:
<pre>
gem install --install-dir /var/www/bundler/ruby/2.4/ json -v '2.0'
</pre>

O si llega a tener problemas de permisos con:
<pre>
doas gem install --install-dir /var/www/bundler/ruby/2.4/ bcrypt -v '3.1.11'
</pre>


## 1.4. Bundler

Para facilitar el manejo de varias gemas (y sus interdependencias) en un proyecto es típico emplear ```bundler``` que instala con:
<pre>
doas gem install bundler
doas ln -sf /usr/local/bin/bundle24 /usr/local/bin/bundle
</pre>

Configurelo para que instale gemas localmente en ```/var/www/bundler/ruby/2.4```:
<pre>
bundler config path /var/www/bundler/ruby/2.4/
</pre>

Puede experimentar descargando un proyecto para ruby ya hecho, seguramente verá
un archivo ```Gemfile```, donde ```bundler``` examina de que librerías depende la aplicación y genera
un archivo ```Gemfile.lock``` con las versiones precisas por instalar de cada gema.  
Una vez tenga un proyecto puede instalar las gemas de las que depende con ```bundle install```

Si eventualmente no logra instalar algunas --por problemas de permisos tipicamente--
puede instalar con ```doas gem install --install-dir /var/www/bundler```

## 1.5. Rails

Se trata de una popular gema que facilita mucho la creación de sitios dinámicos. 

Para instalar globalmente (en ```/usr/local/bin``` y ```/usr/local/lib/ruby/gems/```) la versión estable más 
reciente de Rails (5.0.1 en el momento de este escrito), ejecute
<pre>
doas gem install rails
</pre>

Rails requiere en el servidor un interprete de JavaScript, recomendamos ```node.js``` (ver {1}) incluido en 
el DVD de adJ 6.0 y que se configurará automáticamente.

La gran mayoría de gemas usadas por rails instalarán de la misma forma que se explicó. Algunos casos especiales son:

- ```nokogiri``` que puede requerir ```doas gem install --install-dir /var/www/bundler/ruby/2.4/ nokogiri -- --use-system-libraries --with-xml2-config=/usr/local/bin/xml2-config``` 

- ```capybara-webkit``` que podría requerir ```QMAKE=qmake-qt5 MAKE=gmake doas gem install capybara-webkit```

## 1.6 Coffescript

Si su aplicación rails emplea coffescript necesitará un programa que pueda ejecutar javascript al lado del servidor, recomendamos node.js incluido en adJ, que actualice a la versión más reciente de npm con:
``` 
doas npm install npm -g
``` 

y que instale coffeescript con:

``` 
doas npm install -g coffee-script
``` 

## 1.7. Editor ```vim```

Para emplear ```vim``` como editor se recomienda asegurarse de haber ejecutado:
<pre>
cd ~
mkdir -p .vim
cd .vim
cp -rf /usr/local/share/vim/vim74/* .
</pre>
y si no tiene archivo ~/.vimrc ejecutar:
<pre>
cp /usr/local/share/vim/vim74/vimrc_example.vi ~/.vimrc
</pre>
asi como agregar el archivo ```~/.vim/ftplugin/ruby.vim``` las siguientes líneas:
<pre>
setlocal shiftwidth=2
setlocal tabstop=2
set expandtab   
</pre>

# 2. Documentación

Puede aprender por ejemplo con los tutoriales interactivos de
<https://rubymonk.com>

En Internet puede ver la referencia oficial de las clases en:
<http://ruby-doc.org/core-2.1.5/Integer.html>

Y es buena referencia para Ruby, Rails y Rspec (incluidos cambios entre una versión y otra y comentarios) es:
<http://apidock.com/>

Podrá consultar documentación del núcleo, librería estándar y gemas instalando el paquete ```ruby-ri_docs``` y ejecutando ```ri``` seguido de la clase (por ejemplo ```ri Float```) o sin parámetro ingresa a una consola que da la posibilidad de autocompletar presionando Tab (por ejemplo pruebe tecleando ```Float``` y después Tab dos veces).

También podrá ver la documentación de las gemas en formato Rdoc ejecutando:
<pre>
gem server
</pre>
y con un navegador consultando <http://localhost:8808>

# 3. Uso

## 3.1 Nueva aplicación usando SQLite

Genere una nueva aplicación:
<pre>
rails new aplicacion
cd aplicacion
bundle install
</pre>

Esto creará una nueva aplicación de ejemplo e instalará todas sus dependencias.   
Las gemas que no logre instalar por falta de permisos, como se explicó anteriormente
instalelas con ```doas gem install``` y la opción ```--install-dir /var/www/bundler/ruby/2.4/```

Una vez haya logrado que ```bundle install``` se ejecute completo puede ejecutar:

<pre>
rails server
</pre>

Tras esto puede ver con un navegador la aplicación en el puerto 3000 del computador donde instaló:
<http://127.0.0.1:3000/>

Notará que se genera:

| __Archivo/Directorio__ | __Descripción__ |
|README.rdoc | Documentación en formato Rdoc (puede cambiarse por README.md en formato Markdown)|
|Rakefile | Tareas para ```rake``` (algo análogo a ```make```) |
|config.ru | Configurar servidor web por usar |
|.gitignore | Archivos por ignorar en control de versiones |
|Gemfile | Gemas requeridas |
| app/assets/javascripts/application.js| Plantilla de Javascript para aplicación |
| app/assets/stylesheets/application.css| Plantilla de CSS para aplicación |
| app/controllers/application_controller.rb| Plantilla del controlador de la aplicación |
| app/helpers/application_helper.rb| Ayudas para construir vistas (sin lógica del modelos). |
| app/views/layouts/application.html.erb| Plantilla por defecto para el sitio |
| app/assets/ | Datos estáticos de la página |
| app/assets/images/ | Gráficos de la aplicación (tipicamente que no se sirven estáticos) |
| app/mailers/ | Controlador para enviar correos |
| app/models/ | Modelos |
| app/channel/ | Controlador de websockets |
| app/jobs/ | Maneja cola de tareas programadas | 
| app/mailer/ | Facilita envío y recepción de correos |
| app/controllers/concerns/ | Código que se repite en controladores |
| app/models/concerns/| Código que se repite en modelos |
| bin/bundle| Maneja dependencias con el ambiente de la aplicación |
| bin/rails| Maneja posibilidades de generación y controles Rails|
| bin/rake| Maneja tareas definidas en ```Rakefile``` y ```lib/tasks```|
| bin/setup | Plantilla de un configurador de la aplicación |
| config/routes.rb| Rutas |
| config/application.rb| Configura aplicación |
| config/environment.rb| Configura ambiente |
| config/secrets.yml| Para verificar integridad de galletas firmadas |
| config/environments/development.rb| Configuración ambiente de desarrollo |
| config/environments/production.rb| Configuración ambiente de producción |
| config/environments/test.rb| Configuración ambiente de pruebas |
| config/initializers/assets.rb| Configura recursos |
| config/initializers/backtrace_silencers.rb| Inhibe trazas de algunas librerías |
| config/initializers/cookies_serializer.rb| Configura como serializar galletas |
| config/initializers/filter_parameter_logging.rb|  Parametros por filtrar (no dejar) en bitácoras |
| config/initializers/inflections.rb| Inflecciones singular/plural |
| config/initializers/mime_types.rb| Registra tipos MIME|
| config/initializers/session_store.rb| Configura donde se almacena sesión |
| config/initializers/wrap_parameters.rb| Configuración para ActionController::ParamsWrapper|
| config/locales/en.yml| Localización en inglés |
| config/boot.rb| Prepara rutas para encontrar gemas |
| config/database.yml| Configuración de base de datos |
| config/puma.rb| Configuración de servidor web puma |
| db/seeds.rb | Datos iniciales para base de datos |
| lib/tasks/ | Tareas para ```rake``` |
| lib/assets/ | "Activos" comunes para librerías |
| log/| Bitácoras |
| public | Archivos estáticos |
| public/404.html| Mensaje por defecto para páginas no encontradas |
| public/422.html| Mensaje por defecto para rechazar cambios |
| public/500.html| Mensaje por defecto cuando ocurren errores en servidor |
| public/favicon.ico| Icono |
| public/robots.txt| Puede evitar indexación por parte de motores de búsqueda |
| test/fixtures| Datos para pruebas |
| test/controllers | Pruebas a controladores |
| test/mailers| Pruebas a controladores de envio de correo|
| test/models|  Pruebas a modelos|
| test/helpers|  Pruebas a funciones para ayudar a construir vistas |
| test/integration| Pruebas de integración |
| test/test_helper.rb| Pruebas funciones auxiliares |
| tmp/ | Temporales y cache |
| vendor/assets/javascripts| Fuentes Javascript para código de terceros |
| vendor/assets/stylesheets| Hojas de estilo para código de terceros |



### 3.1.1 Generar un recurso

Puede crear un primer recurso (digamos ```Departamento```) con modelo, controlador simple y vistas para operaciones CRUD y RESTful con:
<pre>
rails g scaffold Departamento nombre:string{500} latitud:float longitud:float fechadeshabilitacion:date
rake db:migrate
</pre>

Tras esto y volver a iniciar el servidor (más breve con ```rails s```) podrá realizar las operaciones básicas sobre departamentos desde: <http://127.0.0.1:3000/departamentos/>

El scaffold genera tablas (con una migración), modelo, controlador, vistas y rutas iniciales que con una interfaz muy simple le permitirán listar departamentos, ver detalles de cada uno, crear nuevos, editar existentes y eliminar.  

Los archivos generados son:

| __Archivo__ | __Descripción__ |
|```db/migrate/20160320131555_create_departamentos.rb```| Migración para crear tabla |
|```app/models/departamento.rb```| Modelo |
|```test/models/departamento_test.rb```| Prueba al modelo|
|```test/fixtures/departamentos.yml```| Datos para pruebas|
|app/controllers/departamentos_controller.rb| Controlador |
|app/views/departamentos| Vistas |
|app/views/departamentos/index.html.erb| Lista de departamentos |
|app/views/departamentos/edit.html.erb| Formulario para editar |
|app/views/departamentos/show.html.erb| Muestra un departamento |
|app/views/departamentos/new.html.erb| Nuevo departamento |
|app/views/departamentos/_form.html.erb| Formulario usado por ```edit``` y ```new``` |
|test/controllers/departamentos_controller_test.rb| Pruebas al controlador |
|app/helpers/departamentos_helper.rb| Funciones auxiliares para construir vistas  |
|app/views/departamentos/index.json.jbuilder| Lista en JSON |
|app/views/departamentos/show.json.jbuilder| Muestra un departamento en JSON |
|app/assets/javascripts/departamentos.coffee| Donde agregar código Coffescript para vistas |
|app/assets/stylesheets/departamentos.scss| Para poner CSS de las vistas |
|app/assets/stylesheets/scaffolds.scss| Para poner CSS de las vistas |

Por convención de Ruby on Rails:

- Las fuentes del módelo quedan en ```app/models/departamento.rb```
- El nombre de la tabla será la forma plural del nombre del módelo (e.g ```departamentos```)
- La tabla incluirá automáticamente un campo id de tipo entero que se autoincrementa
- La tabla incluirá automáticamente los campos ```created_at``` y ```updated_at``` de tipo ```datetime```.  
- La tabla será creada con una migración (cuya fuente quedará en ```db/migrate/```) y el registro de las migraciones ya aplicadas se lleva en una tabla ```schema_migrations``` creada automáticamente en la base de datos.

### 3.1.2 Página de inicio

La página inicial de su aplicación puede crearla generando un controlador con vista, modificando la vista y especificando la ruta:

<pre>
rails g controller hogar
</pre>

Que genera:

| __Archivo__ | __Descripción__ |
|app/controllers/hogar_controller.rb| Controlador |
|app/views/hogar| Vistas relacionadas con el controlador |
|test/controllers/hogar_controller_test.rb| Pruebas |
|app/helpers/hogar_helper.rb| Funciones auxiliares para vistas |
|app/assets/javascripts/hogar.coffee| Coffescript para vistas |
|app/assets/stylesheets/hogar.scss| CSS para vistas |

Modifique el controlador ```app/controllers/hogar_controller.rb``` para indicar que tendrá un vista ```index```:
<pre>
class HogarController < ApplicationController
  def index
  end
end
</pre>

Cree la vista ```app/views/hogar/index.html.erb```:
<pre>
    &lt;article>
    &lt;h1>Mi aplicación&lt;/h1>
    &lt;p> Manejar &lt;a href="/departamentos">Departamentos&lt;/a>&lt;/p>
    &lt;/article>
</pre>

Y en el archivo de configuración de rutas ```config/routes.rb``` añada:

<pre>
root "hogar#index"
</pre>

Al modificar fuentes en general no necesita volver a lanzar el servidor de rails, pero si modifica ```config/routes.rb``` o cualquier archivo fuera del directorio ```app``` debe reiniciar el servicio.

Examine desde un navegador <http://127.0.0.1:3000/>.   Verá lo que escribió en ```app/views/hogar/index.html.erb``` en el ambiente HTML que tenga diseñado en ```app/views/layouts/application.html.erb```.

### 3.1.3 Interacción con la base de datos

Puede examinar la tabla creada e interactuar con la base de datos con la interfaz texto de SQLite como se ejemplifca a continuación:
<pre>
$ rails dbconsole
sqlite> .schema
CREATE TABLE "departamentos" ("id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, "nombre"
 varchar(500), "latitud" float, "longitud" float, "created_at" datetime, "updated_at" datetime);
CREATE TABLE "schema_migrations" ("version" varchar(255) NOT NULL);
CREATE UNIQUE INDEX "unique_schema_migrations" ON "schema_migrations" ("version"
);
</pre>

## 3.2 Ayudas para usar una base de datos PostgreSQL existente:

Se recomienda emplear UTF8 como codificación de PostgreSQL (si emplea otra codificación convierta sacando un respaldo eliminando la base, volviendola a crear con ```-E UTF8``` y restaurando los datos).

* Arregle la aplicación recién creada para que emplee PostgreSQL, modificando parámetros de base de datos en ```config/database.yml```. Si crea una nueva aplicación:
<pre>
cd ..
rails new prueba1 --database=postgresql
</pre>
* Si lo requiere en otra terminal cree un usuario para la base de datos que será el que empleará desde su aplicación ruby:
<pre>
doas su - _postgresql
$ createuser -Upostgres -h/var/www/tmp prueba1
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y 
Shall the new role be allowed to create more new roles? (y/n) n
$ psql -h/var/www/tmp -Upostgres
psql (9.3.6)
Type "help" for help.

postgres=# ALTER USER prueba1 WITH password 'miclave';
ALTER ROLE
postgres=# \q
$ exit
</pre>
* Vuelva a las fuentes de la aplicación y modifique los parámetros de la base de datos editando ```conf/database.yml```  Ponga el usuario recién creado, la clave y en la especificación de cada una de las 3 bases de datos (desarrollo, pruebas y producción) agregue la ruta de la conexión al motor de bases de datos:
<pre>
host: /var/www/tmp/
</pre>
* A manera de prueba de su configuración intente ingresar a un interprete de comandos para su base de datos con:
<pre>
rails dbconsole
</pre>
* Si las tablas de la base de datos no emplean la convención para nomenclatura de Ruby on Rails (nombres de tabla en plural, clases en singular)  agregue al archivo ```config/environment.rb```:
<pre>
ActiveRecord::Base.pluralize_table_names=false
</pre> 
Además de esto para que las pruebas puedan operar seguramente deberá renombrar los archivos con datos de prueba del directorio ```tests/fixtures```, y en lugar de dejar forma plural dejar forma singular.  Si el nombre de tabla es muy diferente en el modelo puede usar ```self.table_name = "cc"``` como se indica en {8}.
* Para generar un volcado inicial de la base de datos en ```db/structure.sql``` (incluyendo características no portables de PostgreSQL a otros motores de bases de datos), agregue al archivo ```config/application.rb``` la línea ```config.active_record.schema_format=:sql``` y después ejecute la tarea rake apropiada (ver {4}):
<pre>
rake db:structure:dump
</pre>
* Genere clases (módelos) en blanco en el directorio ```app/models``` para cada una de las tablas de su aplicación, por ejemplo para la tabla ```mitabla``` :
<pre>
rails generate model mitabla --migration=false --timestamp=false
</pre>
* Cuando la aplicación esté en operación ActiveRecord tomará la estructura directamente de la base de datos, no de los modelos en sus fuentes. Sin embargo puede ser útil poner comentarios en esos modelos con los campos de cada tabla.   Esto puede hacerse instalando la gema ```annotate```  con ```doas gem install annotate``` y ejecutando:
<pre>
annotate21
</pre>
* Si desea ayuda en la generación de controladores y vistas para sus tablas, instale ```doas gem install schema_to_scaffold```, genere (puede ser momentaneamente) el archivo ```db/schema.rb``` con ```rake db:schema:dump``` y ejecute ```scaffold21``` para ver las instrucciones que debe dar a rails para generar modelos, controladores y vistas para las tablas de la base de datos.


### 3.2.1 Detalles para tener en cuenta al migrar aplicaciones

RoR tiene sus propias convenciones respecto a SQL, nombres de tablas y demás, en ocasiones puede ser mejor intercambiar datos JSON con una aplicación ya existente que interactúe con la base de datos.  Sin embargo si desea emplear RoR tanto como pueda para interactuar con la base de datos, tenga en cuenta:

* Toda tabla creada y maneja por RoR se espera que tenga los campos ```created_at``` y ```updated_at``` de tipo ```datetime```.   Puede generar una migración que agregue estos campos digamos para la tabla ```intervalo``` con ```rails g migration addTiempoToIntervalo created_at:datetime updated_at:datetime``` tras lo cual debe editar la migración creada para asegurar que queda bien (por ejemplo pasando a singular la tabla si ese es su caso).
* Por defecto cada tabla debe tener una única llave primaria de nombre ```id``` y tipo entero autoincremental.  Esto puede cambiarse un poco --pero no del todo según {6}-- la sugerencia allí dada es definir la llave primaria en SQL en la migración, la cual debe incluir ```id:false``` y en el modelo agregar ```self.primary_key=:campo_llave_primaria```.  Por ejemplo si la llave primaria de una tabla usuario debe ```id``` pero de tipo ```string```, la migración será:
<pre>
  create_table "usuario", id: false, force: true do |t|
    t.string   "id",                     limit: 15,                   null: false
    t.string   "nombre",                 limit: 50
...
    t.datetime "created_at"
    t.datetime "updated_at"
  end

  add_index "usuario", ["id"], name: "index_usuario_on_id", unique: true, using: :btree
</pre>
Y en el modelo app/models/usuario.rb debe quedar:
<pre>
class Usuario < ActiveRecord::Base
...
  self.primary_key=:id
end
</pre>

* Aunque es posible insertar datos iniciales para algunas tablas en ```db/seeds.rb```, los métodos por defecto de RoR emplearán un campo autoincremental para la identificación, puede insertarse directamente en SQL, como se explica en {7}, con:
<pre>
connection = ActiveRecord::Base.connection()
connection.execute("INSERT INTO mitabla(id, nombre) VALUES ('id1', 'nom1')")
</pre>

##4.3 Localización

Si su público objetivo habla principalmente español puede:

* Instalar la gema ```rails-i18n``` que incluye traducción a español y otros idiomas de varias cadenas de Ruby on Rails
<pre>
doas gem install rails-i18n
</pre>
* Agregar  ```gem "rails-i18n"``` en su archivo ```Gemfile``` y ejecutar
<pre>
bundle update
</pre> 
* Configurar en ```config/application.rb```:
<pre>
 config.time_zone = 'America/Bogota'
 config.i18n.default_locale = :es
</pre>
* Eventualmente corregir inflexiones singular/plural en ```config/initializers/inflections.rb```, por ejemplo (ver {3}):
<pre>
ActiveSupport::Inflector.inflections do |inflect|
        inflect.irregular 'pais', 'paises'
end
</pre>


# 4. Depuración

Como se explica en {5}, desde la aplicación en rails puede entrar a depurar:
* Instale la gema 
```gem install byebug```
* Para activar el depurador en sitios en producción debe además inicar el servidor web con ```rails s --debugger```
* En el sitio de las fuentes donde quiere comenzar a depurar llamando la función ```debugger```

Ruby también ofrece facilidades para medir tiempos como se resume en {4}.


# 5. Referencias

* {1} <http://guides.rubyonrails.org/getting_started.html>
* {2} <http://stackoverflow.com/questions/7092107/rails-3-1-error-could-not-find-a-javascript-runtime>
* {3} <http://stackoverflow.com/questions/5285048/rails-3-routing-singular-option>
* {4} <http://www.caliban.org/ruby/rubyguide.shtml>
* {5} <http://guides.rubyonrails.org/debugging_rails_applications.html>
* {6} <http://stackoverflow.com/questions/1200568/using-rails-how-can-i-set-my-primary-key-to-not-be-an-integer-typed-column>
* {7} <http://stackoverflow.com/questions/6223803/execute-sql-script-inside-seed-rb-in-rails3>
* {8} <http://stackoverflow.com/questions/7542976/add-timestamps-to-an-existing-table>
* {9} <http://stackoverflow.com/questions/4613574/how-do-i-explicitly-specify-a-models-table-name-mapping-in-rails>
