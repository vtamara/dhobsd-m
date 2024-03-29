---

title: Integración entre gitlab y github
date: 2022-03-16 13:18 UTC
tags: 

---

Es posible tener un repositorio principal en gitlab y que mantenga copia 
sincronizada automáticamente en github, o viceversa.

Ambas posibildades requieren que primero cree un testigo de acceso personal
en github con el siguiente procedimiento:

* Diríjase a <https://github.com/settings/apps>
* Elija `Personal access tokens -> Tokens (classic)`
* Cree uno nuevo con un nombre que le facilite identificarlo, el tiempo
  de expiración que requiera y por lo menos con los permisos `repo` 
  (y todos los de este) y `workflow`
* Cuando vea el testigo guárdelo pues no volverá a verlo y lo requerirá con
  cada repositorio que quiera replicar con este token.


# 1. Mantener repositorio principal en gitlab

En github cree un nuevo repositorio en blanco que recibirá la copia.

Configure el repositorio de gitlab para empujar hacía github así:

* Diríjase al repositorio de gitlab que va a replicar
* Elija `Configuración -> Repositorio`
* Expanda la sección `Replicando repositorios`
* Como URL del repositorio utilice
  <https://testigo@github.com/usuario/repositorio.git>
* La dirección de la replica debe ser `push`
* Sin ingresar contraseña alguna presione el botón `Replicar Repositorio`
* Verá que se agrega el repositorio al listado `Repositorios replicados` y podrá
  presionar el icono de `Actualizar` para iniciar la replica (la cual podrá
  confirmar en github).

# 2. Mantener repositorio principal en github

En gitlab:

* Cree un nuevo repositorio donde recibirá la copia de github, elija 
  `Ejecutar CI/CD para un repositorio externo`
* Desde allí presione el botón de github
* En `Token de acceso personal` pegue el testigo que creó en github y que guardó.
* A continuación gitlab presentará los diversos repositorios que tiene
  en github y que podría replicar en gitlab para correr integración continua.
* Elija el que va a replicar y presione `Conectar`.


# Referencias

* https://stackoverflow.com/questions/30268549/mirroring-from-gitlab-to-github
* https://docs.gitlab.com/ee/user/project/repository/mirror/index.html


