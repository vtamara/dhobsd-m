---
title: Motores_Rails
date: 2014-11-12
tags:
---
Es posible separar en motores una aplicación y reusar cada motor en diversas aplicaciones.  Presentamos a continuación algunas ejemplos empleando Rails 4.2.0.beta4 sobre adJ 5.5 (puede ver una introducción al uso de Ruby on Rails sobre la plataforma adJ en {3}).

# 1. MOTOR 

Como se explica en {1} pueden crearse motores aislados (o montables) o no aislados en Ruby on Rails 

# 2. MOTOR NO AISLADO 


# 3. MOTOR AISLADO

Para generar un motor aislado que use rspec para pruebas como se explica en {2}:
<pre>
  rails plugin new s2g -T --mountable --full --dummy-path=spec/dummy
</pre>

Esto además del motor generará una aplicación de prueba en ```spec/dummy```.


## 3.1 Configuración inicial con rspec y uso automático de migraciones en aplicaciones

Complete la configuración de rspec agregando  al archivo ```Gemfile```
<pre>
  gem "rspec-rails"
  gem "factory_girl_rails"
</pre>

Modifique ```lib/s2g/engine.rb``` para indicar el uso de rspec, factory-girls y migraciones automáticas en aplicaciones.
<pre>
module S2g
  class Engine < ::Rails::Engine
    isolate_namespace S2g
    config.generators do |g|
      g.test_framework :rspec
      g.fixture_replacement :factory_girl, :dir => 'spec/factories'
    end
    initializer :append_migrations do |app|
      unless app.root.to_s ``` root.to_s
        config.paths["db/migrate"].expanded.each do |expanded_path|
          app.config.paths["db/migrate"] << expanded_path
        end
      end
    end
  end
end
</pre>
Prepare archivos de rspec
<pre>
  rails generate rspec:install
</pre>

y modifique el archivo generado ```spec/rails_helper.rb``` cambiando la línea:
<pre>
require File.expand_path("../../config/environment", __FILE__)
</pre>
por
<pre>
require File.expand_path("../dummy/config/environment", __FILE__)
</pre>

Además para que la aplicación de pruebe opere bien, agrege en ```spec/dummy/config/application.rb``` después de ```Bundler.require(*Rails.groups)```:
<pre>
require 's2g'
</pre>


## 3.2 Generar un recurso

Ahora puede por ejemplo generar un recurso con:
<pre>
  rails g scaffold actividad minutos:integer nombre:string
</pre>

que genera:
| Contenido | Archivo(s) |
| Migración | db/migrate/!20141117194927_create_s2g_actividads.rb |
| Modelo | app/models/s2g/actividad.rb |
| Prueba a modelo | spec/models/s2g/actividad_spec.r |
| Modelos de ejemplo | spec/factories/s2g_actividads.rb |
| Ruta a actividad | La agrega a config/routes.rb |
| Controldor | app/controllers/s2g/actividads_controller.rb |
| Vistas |      app/views/s2g/actividads/index.html.erb, app/views/s2g/actividads/edit.html.erb, app/views/s2g/actividads/show.html.erb, app/views/s2g/actividads/new.html.erb app/views/s2g/actividads/_form.html.erb |
| Pruebas a controlador |  spec/controllers/s2g/actividads_controller_spec.rb |
| Pruebas a vistas |  spec/views/s2g/actividads/edit.html.erb_spec.rb,  spec/views/s2g/actividads/index.html.erb_spec.rb,  spec/views/s2g/actividads/new.html.erb_spec.rb,  spec/views/s2g/actividads/show.html.erb_spec.rb, spec/routing/s2g/actividads_routing_spec.rb |
| Pruebas a enrutador | spec/routing/s2g/actividads_routing_spec.rb |
| Ayudas para vistas y controladores | app/helpers/s2g/actividads_helper.rb |
| Prueba para ayudas | spec/helpers/s2g/actividads_helper_spec.rb |
|
| Recursos para vistas | app/assets/javascripts/s2g/actividads.js, app/assets/stylesheets/s2g/actividads.css, app/assets/stylesheets/scaffold.css |

Notará que no usa inflección correcta en español --ni aún si la agrega en  ```config/initializers/inflections.rb```. Si prefiere la inflección correcta, para las aplicaciones que usen en plural agregue a ```config/initializers/inflections.rb```:
<pre>
!ActiveSupport::Inflector.inflections do |inflect|
  inflect.irregular 'actividad', 'actividades'
end
</pre>
Y renombre manualmente archivos y cambien nombres erradamente generados, por ejemplo con:
<pre>
for i in `find . -name "*ctividads*"`; do echo $i; n=`echo $i | sed "s/ctividads/ctividades/g"`; echo $n; mv $i $n; done
for i in `find . -exec grep -l "ctividads" {} ';'`; do echo $i; cp $i /tmp/t; sed -e "s/ctividads/ctividades/g" /tmp/t > $i; done
</pre>

## 3.3 Aplicación de prueba

Para ver  el recurso operando en la aplicación de prueba, pase al directorio de la misma e inicie la base:
<pre>
  cd spec/dummy
  rake db:setup
  rake db:migrate
</pre>

Note que si no configuró uso automático de migraciones del motor como se explicó en la sección 3.1, tendrá que ejecutar: ```rake s2g:install:migrations```

Tras esto puede iniciar el servidor de desarrollo (```rails s```) y examinar el recurso en la ruta s2g  (```https://127.0.0.1:3000/s2g/actividades```)

Si prefiere que la aplicación de prueba no emplee ```/s2g/``` en las rutas sin o ```/``` edite ```spec/dummy/config/routes.rb``` y cambie
<pre>
  mount S2g::Engine => "/s2g"
</pre>
por
<pre>
  mount S2g::Engine => "/"
</pre>

## 3.4 Pruebas con rspec

Prepara base para pruebas con:
<pre>
cd spec/dummy
RAILS_ENV=test rake db:setup
RAILS_ENV=test rake db:migrate
</pre>

Debería bastar que después desde el directorio del motor ejecute
<pre>
  rspec
</pre>
pues como se explica en https://www.relishapp.com/rspec/rspec-rails/docs/routing-specs/engine-routes rspec ahora tiene soporte para probar motores.  Sin embargo en el momento de este escrito al probar con Rails 4.2.0.beta3 y rspec 3.1.7 vemos 12 fallas porque los generadores no dejan todo perfecto, deben realizarse los siguientes cambios.

* En la prueba al controlador ```spec/controllers/s2g/actividades_controller_spec.rb``` después de ```RSpec.describe !ActividadesController, :type => :controller do``` agregue:
<pre>
  routes { S2g::Engine.routes }
</pre>
* En las pruebas a rutas de ```spec/routing/s2g/actividades_routing_spec.rb``` también agrege ```routes { S2g::Engine.routes }``` y además modifique los nombres de controladores para añadir el motor, por ejemplo cambie ```route_to("actividades#create")``` por ```route_to("s2g/actividades#create")```
* Para que pasen las pruebas a vistas debe hacer cambios tanto en las pruebas como en las vistas:
** show: En ```spec/views/s2g/actividades/show.html.erb_spec.rb```  cambie rutas como ```"actividades/show"``` por ```"s2g/actividades/show"```, cambie ```Actividad.create``` por ```S2g::Actividad.create```.  En la vista ``` app/views/s2g/actividades/show.html.erb``` cambie llamados ```edit_actividad_path``` y ```actividades_path``` por ```s2g.edit_actividad_path``` y ```s2g.actividades_path```
** new: En ```spec/views/s2g/actividades/new.html.erb_spec.rb``` cambie como en la prueba a show y ademas cambie ```actividades_path``` por ```s2g.actividades_path```.  En ```app/views/s2g/actividades/_form.html.erb``` cambie ```form_for(@actividad)``` por ```form_for(@actividad, :url => s2g.actividades_path(@actividad).sub(/\./,"/"))```  que servirá tanto para los casos new como edit. En ```app/views/s2g/actividades/new.html.erb``` cambie ```actividades_path``` por ```s2g.actividades_path```.
** edit: En =spec/views/s2g/actividades/new.html.erb_spec.rb``` haga los mismos cambios del caso new. Y en ```app/views/s2g/actividades/edit.html.erb``` los mismos cambios del caso new y además ```@actividad``` por ```s2g.actividad_path(@actividad)``` En la prueba a edit cambiar como en la prueba a new, y en la prueba a la vista cambiar 
** index: En ```spec/views/s2g/actividades/index.html.erb_spec.rb``` cambie ```actividades/index``` por ```s2g/actividades/index```, cambie Actividad.create! por ```S2g::Actividad.create!```. En ```app/views/s2g/actividades/index.html.erb``` cambie ```link_to 'Show', actividad``` por ```link_to 'Show', s2g.actividad_path(actividad)```, ```link_to 'Edit', edit_actividad_path(actividad)``` por ```link_to 'Edit', s2g.edit_actividad_path(actividad)``` y ```link_to 'Destroy', actividad``` por ```link_to 'Destroy', s2g.actividad_path(actividad)``` y ```new_actividad_path``` por ```s2g.new_actividad_path```


## 3.5 Una aplicación que usa el motor

Puede iniciarla con:
<pre>
  rails new s2 -T
</pre>
Añada una referencia al motor en Gemfile:
<pre>
  gem 's2g', path: '../s2g'
</pre>
Configure rspec añadiendo a Gemfile en la sección ```development```:
<pre>
  gem 'rspec-rails'
</pre>
Instale gemas necesarias:
<pre>
  bundle install
</pre>
Configure rutas en ```config/routes.rb```:
<pre>
Rails.application.routes.draw do
  mount S2g::Engine => "/", as: 's2g'
end
</pre>


##3.6. Decoradores para modificar clases de un motor

Como se explica en {1}, otros detalles:
* Los decoradores se cargan como parte de la inicialización
* Con la instrucción de {1} se cargan sólo los decoradores de la aplicación, no de los motores.  Los decoradores definidos en motores se deben cargar desde los decoradores de la aplicación.


# 4. PASANDO DE UN MOTOR AISLADO A UNO NO AISLADO

Se presentarán ejemplos tomados de sivel2_gen:

En ```lib/sivel2_gen/engine.rb``` agregar
<pre>
isolate_namespace !Sivel2Gen
</pre>


## 4.1 Modelos

Crear una migración que renombre tablas
<pre>
class !RenombraAislasjr < !ActiveRecord::Migration
  def change
    rename_table :pais, :sivel2_gen_pais  
...
  end
end
</pre>


Mover modelos de ```app/models``` a ```app/models/sivel2_gen```

Agregar módulo a modelos del motor. Por ejemplo pasar de 
<pre>
# encoding: UTF-8
class Pais < !ActiveRecord::Base
  include Basica
end
</pre>
a
<pre>
# encoding: UTF-8
module !Sivel2Gen
  class Pais < !ActiveRecord::Base
    include Basica
  end
end
</pre>
El siguiente script para awk ayuda
<pre>
/.*/ {
        if (lin ``` 2) {
                print "module !Sivel2Gen";
        }
        if (lin >= 2) {
                print "  " $0;
        } else {
                print $0;
        }
        lin++;
}

BEGIN { 
        lin = 1;
}

END {
        if (lin > 2) {
                print "end"; 
        }
}
</pre>

Poner la clase explicita en cada relacion ```has_many```, ```has_one``` y ```belongs_to```, por ejemplo pasar de
<pre>
    has_many :aslegal_respuesta, foreign_key: "id_aslegal", validate: true, dependent: :destroy
    has_many :respuesta, :through => :aslegal_respuesta
</pre>
a
<pre>
    has_many :aslegal_respuesta, class_name: '!Sivel2Sjr::!AslegalRespuesta',
      foreign_key: "id_aslegal", validate: true, dependent: :destroy
    has_many :respuesta, :through => :aslegal_respuesta, class_name: '!Sivel2Sjr::Respuesta'
</pre>  

## 4.2 Controladores

Debe hacerse un cambio análogo al de modelos pasandolos de
app/controllers/  a app/controllers/sivel2_gen y agregando en cada clase
```module !Sivel2Gen```

Es posible que en algunos casos deba referenciar modelos usando el espacio de nombre, e.g ```!Sivel2Gen::Actividadoficio```

Si utiliza !CanCan en algunos contralodores puede requerir indicar la clase explicitamente para cargar y autorizar recursos automáticamente. Por ejemplo en el controlador ```sivel2_gen/app/controllers/sivel2_gen/admin/actividadareas_controller.rb``` cambiar 
<pre>
  load_and_authorize_resource
</pre>
por
<pre>
  load_and_authorize_resource class: !Sivel2Gen::Actividadarea
</pre>

## 4.3 Vistas

Igualmente deben moverse de ```app/views``` a ```app/views/motor```

En las vistas las referencias a clases deben incluir el espacio de nombres,
por ejemplo en lugar de:
<pre>
<%= f.association :pais,
          collection: Pais.habilitados,
          label_method: :nombre,
          value_method: :id
        %>
</pre>
usar
<pre>
<%= f.association :pais,
          collection: !Sivel2Gen::Pais.habilitados,
          label_method: :nombre,
          value_method: :id
        %>
</pre>

Los nombres de clases en los archivos de localización con traducciones deben renombrarse, pasar en ```config/locales/es.yml``` de
<pre>
actividadarea:
        Actividadarea: Área de Actividad
        Actividadareas: Áreas de Actividades
</pre>
a
<pre>
"sivel2_gen/actividadarea":
        Actividadarea: Área de Actividad
        Actividadareas: Áreas de Actividades
</pre>

## 4.4 Pruebas con rspec

Deben pasarse de spec/{models,controllers,routing,views,request}/ a spec/sivel2_gen/{models,controllers,routing,views,ruquests}

Deben renombrarse las "fabricas" de Factory Girls, del directorio ```spec/factories```, por ejemplo
<pre>
mv caso.rb sivel2_gen_caso.rb
</pre>
En cada una de estas "fabricas" debe cambiarse el nombre y añadir la clase explicitamente, por ejemplo pasando de
<pre>
factory :caso do
</pre>
a
<pre>
factory :sivel2_gen_caso, class: "Sivel2Gen::Caso" do
</pre>

Aplican los mismos comentarios de la sección 3.5 y además.
* En las pruebas a modelos cambiar nombre en uso de FactoryGirl, como se acaba de describir


# 5. MOTORES QUE USAN MOTORES

* Pueden usarse decoradores en la aplicación final, pero no en motores intermedios por el "desorden" al cargarlos.  Puede ocurrir que se cargue de último el decorador de un motor y no el de la aplicación.

* Si un motor debe modificar un modelo o un controlador de otro, es mejor usar Concerns.


# 6. REFERENCIAS

* {1} http://guides.rubyonrails.org/engines.html
* {2} http://www.andrewhavens.com/posts/27/how-to-create-a-new-rails-engine-which-uses-rspec-and-factorygirl-for-testing/
* {3} http://dhobsd.pasosdejesus.org/index.php?id=Ruby+on+Rails+en+OpenBSD
* {4} http://pivotallabs.com/leave-your-migrations-in-your-rails-engines/
