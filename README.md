# Permx

Iniciando con un escaneo de puertos y servicios me encontre con los 2 siguientes resultados\


<figure><img src=".gitbook/assets/{66CC1E6A-1857-4E34-BE79-1E30F667527F}.png" alt=""><figcaption></figcaption></figure>

En el escaneo se encuentra un dominio: permx.htb\
El cual prosigo a agregar al archivo /etc/hosts para poder acceder al mismo\


<figure><img src=".gitbook/assets/{EE742878-6D8E-4A39-9244-C7FE359C8346} (1).png" alt=""><figcaption></figcaption></figure>

Parece que el sitio web en el puerto 80 es bastante simple y estático, sin elementos interactivos como enlaces, formularios o páginas de inicio de sesión. Incluso al revisar el código fuente no se encuentra nada relevante. Dado que no hay un exploit público para la versión de SSH, tal vez valga la pena profundizar en otras áreas del servidor o intentar identificar vulnerabilidades no evidentes en el sitio.

Comienzo a hacer fuzzing de directorios, reviso los resultados y veo si aparece algo interesante que pueda investigar más a fondo

<figure><img src=".gitbook/assets/{831CCFB9-131B-47CE-815A-857AEAB2463C}.png" alt=""><figcaption></figcaption></figure>

No encontre nada relevante por lo que prosegui a realizar fuzzing de subdominios, logrando dar con un resultado posiblemente util

<figure><img src=".gitbook/assets/{89979DFC-AD8C-4E00-85D9-A809C480A38C}.png" alt=""><figcaption></figcaption></figure>

Prosigo a agregar el resultado del fuzzing  de subdominios a mi archivo /etc/hosts y repito los pasos realizados con el primer dominio y me encontré con un panel de login&#x20;

<figure><img src=".gitbook/assets/{AB449261-6B8E-481F-8AE1-E2B562861C53}.png" alt=""><figcaption></figcaption></figure>

Mientras dejaba corriendo un fuzzing de directorios en el dominio, me puse a probar diferentes métodos para vulnerar el panel de login. Intenté con algunas credenciales predeterminadas, ataques de fuerza bruta e inyecciones SQL, entre otros, pero no tuve éxito con ninguno. Decidí entonces revisar nuevamente los resultados del fuzzing

<figure><img src=".gitbook/assets/{3FD6EF1B-C91B-4DFE-97D1-E64FD712D8F0}.png" alt=""><figcaption></figcaption></figure>

Encontrandome con el directorio robots.txt expuesto

<figure><img src=".gitbook/assets/{096B375A-3FAC-469B-874C-870D15D0BE76}.png" alt=""><figcaption></figcaption></figure>

Despues de revisar los directorios, en /documentation me encontre con la version de CMS:\
Chamilo 1.11

<figure><img src=".gitbook/assets/{8D97B597-194C-477D-B63F-1722D5CB6BBD}.png" alt=""><figcaption></figcaption></figure>

Busqué el exploit en Google y encontré un repositorio\


<figure><img src=".gitbook/assets/{B0A666D3-A58A-4067-9B1D-C5B2B8839CDB}.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/{1C642062-1CF9-42C3-B286-E04148E67B6D}.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/{BE1F6253-D758-41D2-9C8F-DEB03BECB888}.png" alt=""><figcaption></figcaption></figure>

Logrando así obtener acceso al usuario www-data.\
Prosigo a hacer que mi shell sea mas estable

script /dev/null -c bash\
ctrl +z\
stty raw -echo; fg\
reset xterm\
export TERM=xterm\
export SHELL=bash

Una vez realizado el tratamiento de la TTY\
Después de investigar un poco, encontre las credenciales de la base de datos\
db\_user: chamilo\
db\_password: 03F6lY3uXAP2bkW8

<figure><img src=".gitbook/assets/{F6F3CE8C-C9D7-4E4E-B9F1-7C4A0486D3B0}.png" alt=""><figcaption></figcaption></figure>

Continue investigando y dentro del directorio /home se ve la existencia del directorio del usuario mtz

<figure><img src=".gitbook/assets/{A001C1FA-C0FA-42D3-8CD2-45EFAB977F08}.png" alt=""><figcaption></figcaption></figure>

Al intentar acceder me solicita la credencial del usuario, inicialmente probe con el contenido que encontre al inicio del acceso&#x20;

<figure><img src=".gitbook/assets/{0FFA9B7E-B90B-4A95-946F-88AF43E3A252}.png" alt=""><figcaption></figcaption></figure>

Pero no tuve exito con esa posible contraseña.\
Se me ocurrio evaluar la posibilidad de un descuido muy comun por parte de los usuarios el cual es la reutilizacion de contraseñas, asi que decidi optar por probar la contraseña encontrada en el archivo configuration.php con el usuario mtz

<figure><img src=".gitbook/assets/{F2BBE73F-764E-49F5-97D2-960CD5A174B8} (1).png" alt=""><figcaption></figcaption></figure>

Logrando asi tener exito y pudiendo entrar al directorio mtz, donde pude encontrar la primer flag, la flag de usuario.

Ahora toca realizar la escalacion de privilegios

<figure><img src=".gitbook/assets/{393B342E-F858-468F-8E09-1E6751C1C91F}.png" alt=""><figcaption></figcaption></figure>

Intenté verificar qué permisos tenía y vi un archivo en la carpeta /opt, por lo que me puse a ver que hacia el archivo

\


<figure><img src=".gitbook/assets/{B0D10746-2E07-4DD8-AB66-FC85DAEE49A9}.png" alt=""><figcaption></figcaption></figure>

Es un programa que tiene 3 permisos donde dice que para el usuario **mtz** puede agregar cualquier permiso pero la ruta tiene que ser y el objetivo debe ser un archivo. Pero en mtz no habia nada que pudiese utilzar.\
Si creamos un **enlace simbólico** con el archivo, podria editarlo desde el directorio mtz

<figure><img src=".gitbook/assets/{C3BBB277-5191-4DA5-8711-01ED5E13B069}.png" alt=""><figcaption></figcaption></figure>

Logrando asi, la escalacion de privilegios y obteniendo la ultima flag del usuario root

<figure><img src=".gitbook/assets/{39D550EF-A95F-48F9-B4EA-EBC4DAE4C577}.png" alt=""><figcaption></figcaption></figure>
