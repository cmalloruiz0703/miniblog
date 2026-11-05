# README #

---

Este repositorio contiene una pequeña aplicación web de prueba realizada originalmente con Silex (hoy migrada a Symfony 7) que usé como demostración en la [Cuarta BetaBeers de Córdoba](http://betabeers.com/event/4-betabeers-cordoba-626/). Trata de un blog muy simple, con un frontend y un backend sencillo para la creación de nuevas entradas.

La aplicación está basada en la aplicación [Bilbostack](https://github.com/javiereguiluz/bilbostack) realizada por [Javier Eguiluz](https://twitter.com/javiereguiluz), al cual agradezco que me haya dejado usar su material como apoyo a mi charla. Podéis encontrar [más material sobre Silex](http://symfony.es/noticias/2013/01/30/silex-desarrollo-web-%C3%A1gil-y-profesional-con-php/) en su página web.

**Requisitos**: PHP >= 8.2 con las extensiones `pdo_sqlite`, `ctype` e `iconv`, y [Composer](https://getcomposer.org).

**Cómo instalar el proyecto**

```
$ git clone https://github.com/mpgprofe/miniblog.git
$ cd miniblog
$ composer install
$ php bin/console create-schema
$ php bin/console app:create-user admin "Administrador"   # pide la contraseña (mín. 8 caracteres)
$ php -S localhost:8000 -t public
```

`create-schema` también actualiza bases de datos de versiones anteriores (las entradas
existentes quedan como publicadas). La base de datos SQLite se guarda en `var/schema.sqlite`
(el directorio `var/` debe ser escribible por el servidor web). Cambia `APP_SECRET` en un
`.env.local` antes de desplegar en producción.

**Usuarios y roles** (acceso en `/login`, panel en `/admin`)

| Rol | Puede |
|---|---|
| Alumno | Crear entradas como borrador o enviarlas a revisión; editar/borrar solo las suyas mientras no estén publicadas. |
| Docente | Todo lo anterior sobre cualquier entrada, y publicarlas o despublicarlas. |
| Administrador | Todo lo anterior, más gestionar usuarios y categorías. |

Para crear más usuarios desde la consola: `php bin/console app:create-user NOMBRE "Nombre completo" --role=ROLE_TEACHER`
(roles: `ROLE_ADMIN`, `ROLE_TEACHER`, `ROLE_STUDENT`). El administrador también puede hacerlo desde `/admin/users/`.

**Datos del centro y tutorial de acceso**

La dirección y el teléfono que aparecen en el pie y en `/contacto` se editan en
`config/packages/twig.yaml` (clave `school`). La primera vez que un visitante entra, un
minitutorial de 3 pasos explica cómo acceder; se puede reabrir con «¿Cómo accedo?».

**Editor, imágenes, páginas, buscador y RSS (v2.2)**

- El editor es CKEditor 5 (copia local en `public/vendor/ckeditor5`, licencia GPL-2+; ver `LICENSE.md`
  de esa carpeta). Las imágenes se suben desde el propio editor a `public/uploads/` (JPG, PNG, GIF o WebP,
  máx. 3 MB; si el servidor limita menos con `upload_max_filesize`/`post_max_size`, aplica el límite del servidor).
  **`public/uploads/` debe ser escribible por el servidor web**, y conviene incluirla en las copias de seguridad.
- Todo el HTML que se guarda se sanea en el servidor (se eliminan scripts, `onclick`, `javascript:`, iframes…).
- El administrador crea páginas fijas (horarios, normas…) en `/admin/pages/`; se muestran en `/p/<dirección>` y,
  si se marca, en el menú.
- Buscador (`/buscar?q=`), paginación de 10 en 10 y feed RSS en `/rss`.
- Tras actualizar desde una versión anterior, ejecuta `php bin/console create-schema`.

**Usuarios de demostración (para probar)**

Crea un alumno y un docente de prueba desde la consola. El comando pide la contraseña
(mínimo 8 caracteres, por eso no sirve `demo`; aquí se usa `demo1234`):

```
$ php bin/console app:create-user alumno.demo "Alumno Demo" --role=ROLE_STUDENT
$ php bin/console app:create-user docente.demo "Docente Demo" --role=ROLE_TEACHER
```

Para crearlos sin que pregunte (por ejemplo, en un script):

```
$ printf 'demo1234\n' | php bin/console app:create-user alumno.demo "Alumno Demo" --role=ROLE_STUDENT
$ printf 'demo1234\n' | php bin/console app:create-user docente.demo "Docente Demo" --role=ROLE_TEACHER
```

Con ellos puedes probar el flujo completo: entra como `alumno.demo`, escribe una entrada y envíala a
revisión; después entra como `docente.demo` (o como administrador), revísala y publícala.

> **Importante:** borra estos usuarios (en `/admin/users/`, como administrador) o cambia su contraseña
> antes de publicar el blog en internet. Contraseñas conocidas en un servidor público son un riesgo.

## Cómo he desplegado el proyecto

Entorno: máquina virtual con Debian 13 («trixie»), a la que me conecto por SSH desde Windows.

### Comandos que he usado

```bash
# Instalar dependencias del sistema
sudo apt update
sudo apt install git openssh-client php-cli php-sqlite3 php-xml php-mbstring php-intl php-curl unzip composer

# Configurar Git
git config --global user.name "Carlos Mallo Ruiz"
git config --global user.email "mi-correo@ejemplo.org"
git config --global init.defaultBranch master

# Clave SSH para GitHub
ssh-keygen -t ed25519 -C "mi-correo@ejemplo.org"

# Clonar mi fork y preparar el proyecto
git clone git@github.com:cmalloruiz0703/miniblog.git
cd miniblog
composer install
php bin/console create-schema
php bin/console app:create-user admin "Administrador"
php -S localhost:8000 -t public
```

### Problemas que me han surgido

- **No veía la web desde Windows:** el servidor de PHP corre en la máquina virtual, así que `localhost:8000` del navegador de Windows no llegaba. Lo resolví con un túnel SSH: `ssh -L 8000:localhost:8000 usuario@IP-de-la-VM`.
- **`ERR_CONNECTION_CLOSED` en el navegador:** Chrome abría la dirección con `https://` y el servidor de pruebas de PHP solo habla `http://`. Lo resolví escribiendo la URL completa con `http://`.
- **Pull request al destino equivocado:** GitHub propone por defecto el repositorio del profesor. Lo cerré y lo volví a crear cambiando `base repository` a mi propio fork.
