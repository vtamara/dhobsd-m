---
title: Formularios_Anidados_en_Rails_con_Cocoon
date: 2014-01-14
tags:
---
Al manejar formularios anidados sobre Ruby on Rails 4.1 (ver {6}), Simple Form (ver {8}) y Cocoon (ver {7}), por ejemplo con una relación muchos a muchos, hemos identificado dos formas de manejar la tabla combinada (traducción de {4} de ''join table''):

* Cuando tiene llave primaria ```id```
* Cuando no tiene llave primaria o la llave primaria es múltiple (digamos las llaves de las 2 tablas que relaciona).

Con la primera forma es más sencillo implementar los formularios anidados, pero la segunda es la que Rails genera por defecto al crear tablas combinadas.

A continuación explicamos ambos casos. 

#1. Contexto y modelos

En el contexto de SIVeL (ver {2}), un caso (modelo ```Caso```) puede tener diversos presuntos responsables (modelo ```Presponsable```), y un mismo presunto responsable puede aparecer en diversos casos.

La relación muchos a muchos se implementa con la tabla ```Caso_Presponsable```.   

Así que las tablas tiene entre otros los siguientes campos (para este ejemplo sólo emplearemos estos):

* Caso
  + id
  + titulo
  + fecha

* Presponsable
  + id
  + nombre

* Caso_Presponsable
  + id_caso
  + id_presponsable
  + otro

Como nota, los nombres de las tablas legadas de esa aplicación están en singular (mientras que en la convención de Ruby on Rails deberían estar en plural) y las llaves foráneas son de la forma ```id_tabla``` (en lugar de seguir la convención de Rails que sería ```tabla_id```).   Estas diferencias pueden forzarse en Rails así:
* Poniendo ```!ActiveRecord::Base.pluralize_table_names = false``` en ```config/environment.rb```
* Al declarar asociaciones (ver {5}) en los modelos con ```has_many```, ```has_one``` y ```belongs_to``` especificar la llave foranea de manera explícita empleando la opción ```foreign_key``` (ver ejemplos a continuación).

Los 3 modelos incluyendo la declaración requeridas para formularios anidados (ver {9}) son:

```app/models/caso.rb```:
<pre> 
  class Caso < !ActiveRecord::Base
      has_many :presponsable,
        :through => :caso_presponsable
      has_many :caso_presponsable,
        foreign_key: :id_caso,
        dependent: :destroy,
        validate: true
      accepts_nested_attributes_for :caso_presponsable,
        allow_destroy: true,
        reject_if: :all_blank
  end
</pre>

```app/models/presponable.rb```
<pre>
  class Presponsable < !ActiveRecord::Base
    has_many :caso, through: :caso_presponsable
    has_many :caso_presponsable, foreign_key: "id_presponsable", validate: true
  end
</pre>

```app/models/caso_presponsable.rb```
<pre>
  class !CasoPresponsable < !ActiveRecord::Base
    belongs_to :caso, foreign_key: "id_caso", validate: true
    belongs_to :presponsable, foreign_key: "id_presponsable", validate: true
  end
</pre>

Para anidar los formularios en el controlador de caso (```app/controllers/casos_controllers.rb```) además de permitir recibir datos del formulario padre (```Caso```), deben permitirse datos del formulario hijo o anidado (```!Caso_Presponsable```) así como el parámetro especial ```:_destroy``` que permite eliminar registros de ```!Caso_Presponsable```:
<pre>
    def caso_params
      params.require(:caso).permit(:fecha, :titulo,
        :caso_presponsable_attributes => [
          :id_presponsable, :otro, :_destroy
        ]
      )
    end
</pre>

En la vista parcial ```app/views/casos/_form.html.erb``` utilizada para crear y actualizar casos, además de los campos de la tabla ```Caso``` es necesario incluir el formulario anidado como parcial y un botón para permitir añadir Presuntos Responsables:
<pre>
  <ul>
  <%= f.simple_fields_for :caso_presponsable do |caso_presponsable| %>
      <%= render 'caso_presponsable_fields', :f => caso_presponsable %>
  <% end %>
  </ul>
  <div class="links">
    <%= link_to_add_association 'Añadir Presunto Responsable', f,
      :caso_presponsable, :class => 'btn-primary' %>
  </div>
</pre>

El formulario parcial ```app/views/casos/_caso_presponsable_fields.html.erb``` incluirá los campos de ```caso_presponsable``` dentro de una sección div con clase ```nested-fields``` y un botón para eliminar un registro de esta tabla:
<pre>
<li>
  <div class='control-group nested-fields'>
    <div class="controls">
    <%= f.association :presponsable,
      label: "Presunto Responsable",
      label_method: :nombre,
      value_method: :id %>
    <%= f.input :otro %>
    <%= link_to_remove_association "Eliminar Presunto Responsable", f,
      :class => 'btn-danger' %>
    </div>
  </div>
</li>
</pre>

Para que se agreguen y eliminen campos dinamicamente Cocoon provee la lógica para Rails y Javascript que se activa agregando en ```app/assets/javascripts/casos.js.coffee```:
<pre>
//= require cocoon
</pre>

#2. Implementaciones posibles para la tabla combinada


##2.1 Con llave primaria ```id```

Puede ver las fuentes de un ejemplo que implementamos en:
https://github.com/vtamara/cocoon-caso-presponsable
Lo hemos probado en adJ versión 5.4alfa (ver {10}) que incluye Ruby 2.0 (ver {1}).  También puede verlo operando sobre heroku en: http://cocoon-caso-presponsable.herokuapp.com/

Para este caso nos ha resultado necesario incluir el campo ```id``` escondido en el formulario parcial ```app/views/casos/_caso_presponsable_fields.html.erb```

<pre>
<%= f.input :id, as: :hidden %>
</pre>

Se ha preferido asegurar que no pueda relacionarse más de una vez el mismo presunto responsable a un caso con un índice en ```db/schema.rb```:

<pre>
add_index "caso_presponsable", ["id_caso", "id_presponsable"], 
  name: "index_caso_presponsable_on_caso_id_and_presponsable_id", 
  unique: true
</pre>

##2.2 Sin llave primaria

Puede ver fuentes en la rama ```sin-indice``` del mismo repositorio:
https://github.com/vtamara/cocoon-caso-presponsable/tree/sin-indice
o ya desplegada en heroku en: http://cocoon-caso-presponsable-sin-i.herokuapp.com/

Comparando ambas posibilidades (ver comparación de los repositorios en 
https://github.com/vtamara/cocoon-caso-presponsable/compare/sin-indice )
 notará las siguientes diferencias:

* En db/schema.rb (o cuando fuere el caso en la migración) debe agregar el parámetro ```id: false``` cuando se crea la tabla ```caso_presponsable``` y por lo mismo no debe crearse índice.  Esto ocurre  por defecto cuando genera la migración como se explica en la documentación oficial (ver {3}), e.g: ```rails g migration !CreateJoinTableCasoPresponsable``` 
* Entre los parametros por definir en el controlador ya no se requiere ```id``` en ```caso_parametro_attributes``` en el método ```caso_params``` de ```app/controllers/casos_controllers.rb```.
* Al actualizar o eliminar un caso es necesario eliminar todas las entradas de la tabla ```caso_presponsable``` relacionadas (con ```@caso.caso_presponsable.clear```) en los métodos ```update``` y ```destroy``` de ```app/controllers/casos_controllers.rb```.
* En la vista parcial ```app/views/casos/_caso_presponsable_fields.html.erb``` tampoco se requiere el campo ```id```


#3. Conclusión

Para hacer formulario anidados es más directo con Rails 4.1 y Cocoon 1.2.6 emplear tablas combinadas que tengan una llave  primaria ```id```.  

Sin embargo en aplicaciones que se estén migrando, así como en las tablas combinadas generadas por Rails, no habrá llave primaria ```id``` en tablas combinadas.  Aún así y al menos para casos como el aquí ejemplificado es posible emplear formularios anidados y Cocoon, siempre y cuando se eliminen en el controlador del formulario papá los campos relacionados en la tabla combinada antes de actualizar o eliminar un registro de la tabla papá.  Es un método un poco más riesgoso, pues en caso de fallas al actualizar se perderán los registros que se eliminan.

#4. Referencias

* {1} [Ruby on Rails en OpenBSD]
* {2} http://sivel.sf.net
* {3} http://guides.rubyonrails.org/migrations.html
* {4} http://www.interglot.com/dictionary/en/es/translate/outer%20join
* {5} http://guides.rubyonrails.org/association_basics.html
* {6} http://edgeguides.rubyonrails.org/index.html
* {7} https://github.com/nathanvda/cocoon
* {8} https://github.com/plataformatec/simple_form
* {9} http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html
* {10} http://aprendiendo.pasosdeJesus.org

----
<pre>
Respondió Rut: No me rueges que me aleje y me aparte de ti, 
porque a donde quiera que tu fuerés, iré yo,
y dondequiera que vivieres, viviré.
Tu pueblo será mi pueblo y tu Dios será mi Dios.
</pre>

Rut 1:16
