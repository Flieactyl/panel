![Flieactyl](https://cdn.discordapp.com/attachments/1063585626022223892/1065304573058764850/PylexPlus_2.png)

<hr>

# Flieactyl • El panel moderno de clientes para Pterodactyl

Todas las funciones:
- Gestión de recursos (Úsala para crear servidores, regalarlos, etc.)
- Monedas (Obtención en la página AFK, obtención vía Linkvertise, posibilidad de regalarlas)
- Renovación (Requiere monedas para la renovación)
- Cupones (Otorga recursos y monedas a un usuario)
- Servidores (Crear, ver y editar servidores)
- Pagos (Comprar a través de Stripe)
- Cola de inicio de sesión (Evita la sobrecarga del sistema)
- Sistema de usuarios (Autenticación, recuperación de contraseña, etc.)
- Tienda (Comprar recursos con monedas)
- Panel de control (Visualizar recursos)
- Unirse para obtener recompensas (Únete a servidores de Discord para ganar monedas)
- Administración (Establecer/añadir/eliminar monedas y recursos; crear/revocar cupones)
- API (Para bots y otros fines)

# Advertencia

No podemos obligarle a mantener la mención «Powered by Flieactyl» en el footer de página; sin embargo, le pedimos que considere conservarla. Esto contribuye a dar mayor visibilidad al proyecto y, por ende, a su mejora continua. No ofreceremos soporte técnico para aquellas instalaciones que no incluyan dicha mención en el footer de la página. Asimismo, bajo ciertas circunstancias, podríamos presentar una reclamación DMCA contra el sitio web.
No obstante, le insistimos: por favor, mantenga el footer de página.

<hr>

# Guía de instalación
## 1. Configurando Flieactyl
### Pterodactyl egg
Advertencia: Necesitas tener Pterodactyl ya configurado para que esto funcione.

<strong>1.1</strong> Sube el archivo anterior a un servidor Pterodactyl NodeJS [Descarga el egg desde el repositorio de GitHub de Parkervcp](https://github.com/parkervcp/eggs/blob/master/generic/nodejs/egg-node-js-generic.json)

<strong>1.2</strong> Descomprime el archivo y configura el servidor para utilizar NodeJS 16.

### Método directo
<strong>1.1</strong> Instale nodejs 16; se recomienda instalarlo con nvm:
- `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`
- Vuelva a abrir una nueva sesión SSH (por ejemplo, reinicie Termius).
- `nvm install 16`
- Verifica la versión de Node con `node -v` y cambia entre versiones con `nvm use <version>`.

<strong>1.2</strong> Descarga los archivos de Heliactyl en /var/www/flieactyl:
- `git clone https://github.com/Flieactyl/panel.git /var/www/flieactyl`

<strong>1.3</strong> Instalando los módulos de Node requeridos (y las dependencias de compilación para evitar errores):
- `apt-get update && apt-get install libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev build-essential`
- `cd /var/www/flieactyl && npm i`

Tras configurar settings.json, para iniciar el servidor utilice `node index.js`</br>
Para ejecutarlo en segundo plano, utilice pm2 (consulte la sección de pm2) o screen</br>
`screen -S flieactyl node index.js`</br>
Para desvincularse de la sesión de screen, presione Ctrl + A + D</br>
Para volver a vincularse: `screen -R flieactyl`</br>
Para detenerlo: Ctrl + C

## 2. Configurando el servidor web

<strong>2.1</strong>  Configura settings.json (específicamente el dominio/clave API del panel y los ajustes de autenticación de Discord para que funcione).

<strong>2.2</strong>  Inicia el servidor (ignora los 2 errores extraños que podrían aparecer).

<strong>2.3</strong>  Inicia sesión en tu gestor de DNS y apunta el dominio en el que deseas alojar tu panel de control hacia la dirección IP de tu VPS. (Ejemplo: dashboard.domain.com 192.168.0.1)

<strong>2.4</strong>  Ejecuta `apt install nginx && apt install certbot` en el VPS.

<strong>2.5</strong>  Ejecuta `ufw allow 80` y `ufw allow 443` en el VPS

<strong>2.6</strong>  Ejecuta `certbot certonly -d <Tu dominio de Flieactyl>`, luego selecciona la opción 1 e introduce tu correo electrónico.

<strong>2.7</strong>  Ejecuta `nano /etc/nginx/sites-enabled/flieactyl.conf`

<strong>2.8</strong> Pega la configuración al final de este archivo y reemplaza la IP por la del servidor de Pterodactyl (incluyendo el puerto), así como el dominio en el que deseas alojar tu panel de control.

<strong>2.9</strong> Ejecuta `systemctl restart nginx` e intenta abrir tu dominio.

# Nginx Proxy Config
```Nginx
server {
    listen 80;
    server_name <domain>;
    return 301 https://$server_name$request_uri;
}
server {
    listen 443 ssl http2;
location /afkwspath {
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_pass "http://localhost:<port>/afkwspath";
}
    
    server_name <domain>;
ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
location / {
      proxy_pass http://localhost:<port>/;
      proxy_buffering off;
      proxy_set_header X-Real-IP $remote_addr;
  }
}
```

<hr>


# Ejecución en segundo plano / al inicio, en un servidor en lugar de dentro de Pterodactyl.

Instalando [pm2](https://github.com/Unitech/pm2):
- Ejecuta `npm install pm2 -g` en el VPS

Iniciar el panel en segundo plano:
- Cambia el directorio a tu carpeta de Heliactyl utilizando el comando `cd`. Ejemplo: `cd /var/www/flieactyl`
- Para ejecutar Flieactyl, utiliza `pm2 start index.js --name "Flieactyl"`
- Para ver los registros, ejecuta `pm2 logs Flieactyl`

Configurar el panel para que se inicie automáticamente:
- Asegúrese de que su panel se esté ejecutando en segundo plano con la ayuda de [pm2](https://github.com/Unitech/pm2).
- Puede verificar si Flieactyl se está ejecutando en segundo plano mediante el comando `pm2 list`.
- Una vez que haya confirmado que Flieactyl se está ejecutando en segundo plano, puede crear un script de inicio ejecutando `pm2 startup` y `pm2 save`.
- Nota: Los sistemas de inicio compatibles son `systemd`, `upstart`, `launchd` y `rc.d`.
- Para detener la ejecución de Flieactyl en segundo plano, utilice el comando `pm2 unstartup`.

Para detener una instancia de Flieactyl que se esté ejecutando actualmente, utiliza `pm2 stop flieactyl`.
