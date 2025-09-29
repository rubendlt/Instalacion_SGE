# Instalacion_SGE
# Instalación y Configuración de WordPress en Ubuntu Server

## 1️⃣ Instalar PHP, Apache y MySQL

Actualiza la lista de paquetes e instala Apache, MySQL, PHP y extensiones necesarias:

```bash
sudo apt update
sudo apt install apache2 \
                 ghostscript \
                 libapache2-mod-php \
                 mysql-server \
                 php \
                 php-bcmath \
                 php-curl \
                 php-imagick \
                 php-intl \
                 php-json \
                 php-mbstring \
                 php-mysql \
                 php-xml \
                 php-zip
```

---

## 2️⃣ Descargar e instalar WordPress desde WordPress.org

Usaremos la versión oficial de WordPress desde WordPress.org, ya que es el método recomendado y evita problemas inesperados.

Crea el directorio de instalación y descarga WordPress:

```bash
sudo mkdir -p /srv/www
sudo chown www-data: /srv/www
curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```

**Explicación de los comandos:**

- `mkdir -p /srv/www` crea el directorio donde se instalará WordPress.
- `chown www-data: /srv/www` asigna la propiedad del directorio al usuario y grupo `www-data` usado por Apache.
- `curl ... | sudo -u www-data tar zx -C /srv/www` descarga la última versión de WordPress y la descomprime directamente en el directorio con permisos de Apache.

---

## 3️⃣ Configurar Apache para WordPress

Crea el archivo de configuración del sitio de Apache:

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Agrega el siguiente contenido:

```apache
<VirtualHost *:80>
    DocumentRoot /srv/www/wordpress
    <Directory /srv/www/wordpress>
        Options FollowSymLinks
        AllowOverride Limit Options FileInfo
        DirectoryIndex index.php
        Require all granted
    </Directory>
    <Directory /srv/www/wordpress/wp-content>
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```

---

### Habilitar el sitio y la reescritura de URLs

Habilita el sitio de WordPress:

```bash
sudo a2ensite wordpress
```

Habilita la reescritura de URLs:

```bash
sudo a2enmod rewrite
```

Deshabilita el sitio por defecto “It Works”:

```bash
sudo a2dissite 000-default
```

Recarga Apache para aplicar los cambios:

```bash
sudo service apache2 reload



---

## Configurar la base de datos MySQL para WordPress

Para configurar WordPress, necesitamos crear una base de datos en MySQL y un usuario con los permisos necesarios.

1️⃣ Accede a MySQL como usuario root:

```bash
sudo mysql -u root
```

2️⃣ Dentro de MySQL, crea la base de datos para WordPress:

```sql
CREATE DATABASE wordpress;
```

3️⃣ Crea un usuario de MySQL para WordPress y establece su contraseña:

```sql
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY '<tu-contraseña>';
```

4️⃣ Concede los permisos necesarios al usuario sobre la base de datos:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER ON wordpress.* TO 'wordpress'@'localhost';
```

5️⃣ Aplica los cambios de privilegios:

```sql
FLUSH PRIVILEGES;
```

6️⃣ Sal de MySQL:

```sql
quit;
```

*Explicación:*  
- `CREATE DATABASE` crea la base de datos donde WordPress almacenará su información.  
- `CREATE USER` define un usuario con contraseña para conectarse a la base de datos.  
- `GRANT` otorga todos los permisos necesarios para que WordPress funcione correctamente.  
- `FLUSH PRIVILEGES` aplica los cambios de permisos inmediatamente.

## Configurar WordPress para usar la base de datos

Ahora configuraremos WordPress para usar la base de datos que creamos anteriormente.

### 1️⃣ Copiar el archivo de configuración de ejemplo

```bash
sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
```

### 2️⃣ Establecer las credenciales de la base de datos

No reemplaces `database_name_here` o `username_here` en los comandos; reemplaza `<tu-contraseña>` con la contraseña de tu base de datos.

```bash
sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/password_here/<tu-contraseña>/' /srv/www/wordpress/wp-config.php
```

### 3️⃣ Editar las claves de seguridad

Abre el archivo de configuración en nano:

```bash
sudo -u www-data nano /srv/www/wordpress/wp-config.php
```

Busca las siguientes líneas:

```php
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
```

Elimina estas líneas (Ctrl+K en nano) y reemplázalas con las claves generadas automáticamente desde [WordPress Salt API](https://api.wordpress.org/secret-key/1.1/salt/). Esto asegura que tu sitio no sea vulnerable a ataques conocidos.

Guarda y cierra el archivo con `Ctrl+X`, luego `Y` y presiona Enter.

*Explicación:*  
- Copiamos el archivo de configuración de ejemplo para crear `wp-config.php`.  
- Configuramos la base de datos, usuario y contraseña para que WordPress pueda conectarse.  
- Reemplazamos las claves de seguridad con valores aleatorios para proteger el sitio contra ataques de fuerza o de secretos conocidos.

## Configurar WordPress desde el navegador

Abre tu navegador y visita:

```
http://localhost/
```

Serás solicitado para:

- Título de tu nuevo sitio.
- Nombre de usuario para WordPress.
- Contraseña para WordPress.
- Dirección de correo electrónico.

> Nota: El nombre de usuario y la contraseña que elijas aquí son **solo para WordPress**. No proporcionan acceso a ninguna otra parte de tu servidor. Asegúrate de que sean diferentes de:
> - Tus credenciales de MySQL.
> - Tu usuario y contraseña para iniciar sesión en tu máquina o en la terminal.

También podrás elegir si quieres que tu sitio sea indexado por los motores de búsqueda.

*Explicación:*  
- Esta configuración finaliza la instalación de WordPress y permite que accedas al panel de administración desde tu navegador.
