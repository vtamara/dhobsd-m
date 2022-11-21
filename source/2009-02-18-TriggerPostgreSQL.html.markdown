---
title: TriggerPostgreSQL
date: 2009-02-18 12:00 UTC
tags:
---

Es posible crear procedimientos almacenados y triggers en PostgreSQL usando varios lenguajes, entre otros PLpgSQL que es una ampliación a SQL.

Presentamos un breve ejemplo de como mantener actualizado el promedio de los números de la tabla ```numeros``` en la tabla ```promedio```.

!Creación de tablas,  función y trigger

<pre>
CREATE TABLE numeros (num integer);

CREATE TABLE promedio (tabla VARCHAR(20),  prom FLOAT);

INSERT INTO TABLE promedio VALUES ('numeros', '0');

CREATE LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION calcular_promedio()
RETURNS "trigger" AS
 '
 BEGIN
 UPDATE promedio SET prom=(SELECT AVG(num) FROM numeros);
return NEW;
END;
'
 LANGUAGE 'plpgsql' VOLATILE;


CREATE TRIGGER recalcula_promedio AFTER INSERT OR UPDATE OR DELETE ON
numeros FOR EACH STATEMENT
EXECUTE PROCEDURE calcular_promedio();
</pre>

!Prueba de uso
<pre>
INSERT INTO numeros VALUES ('5');

SELECT * FROM promedio ;
  tabla  | prom 
---------+------
 numeros |    5
(1 row)
</pre>


------

Este escrito se dedica a Dios por su Belleza. Dominio público. 2009. vtamara@pasosdeJesus.org
