# GymSystem

Sistema web de gestión para gimnasios, desarrollado en PHP con arquitectura MVC. Permite administrar socios, planes, suscripciones, asistencias, caja, gastos, usuarios y reportes visuales.

## 1) Ficha técnica del proyecto

### Nombre
- **GymSystem**

### Tipo de aplicación
- Aplicación web monolítica (backend + vistas renderizadas en servidor).

### Arquitectura
- Patrón **MVC** (Model-View-Controller).
- Front controller en `public/index.php`.
- Enrutamiento por URL amigable mediante `.htaccess` (Apache `mod_rewrite`).

### Stack tecnológico
- **Backend:** PHP (PDO para MySQL).
- **Base de datos:** MySQL 8.x.
- **Frontend:** HTML, Bootstrap 5, jQuery.
- **Librerías JS (CDN):** DataTables, SweetAlert2, Chart.js, Font Awesome.
- **Generación de documentos:** FPDF + QR (librerías incluidas en `app/lib`).

### Base de datos
- Script de respaldo y estructura: `bk_basededatos.sql`.
- Nombre de la BD esperada en código: `gym_system`.
- Conexión definida en: `app/config/Database.php`.

### Módulos funcionales
- **Autenticación y sesión:** login/logout por roles.
- **Dashboard:** KPIs (socios, ingresos, gastos, utilidad) y gráficos.
- **Socios:** alta, edición y estado.
- **Planes:** CRUD de planes y precios.
- **Suscripciones:** altas/renovaciones y control de vencimientos.
- **Asistencia:** validación por DNI y registro diario.
- **Caja:** apertura/cierre y movimientos.
- **Gastos:** registro de egresos.
- **Usuarios:** administración por rol (`admin`, `recepcionista`, `entrenador`).
- **Configuración:** datos del gimnasio (logo, RUC, moneda, etc.).
- **Documentos:** carnet con QR y comprobantes PDF.

### Control de acceso por rol (resumen)
- `admin`: acceso total (incluye configuración, usuarios, gastos, planes).
- `recepcionista`: operación diaria (asistencia, socios, caja, suscripciones).
- `entrenador`: foco operativo limitado (asistencia y gestión de socios/rutinas).

## 2) Estructura principal del proyecto

```
GymSystem/
├── app/
│   ├── config/            # Conexión y configuración
│   ├── controllers/       # Lógica de negocio por módulo
│   ├── models/            # Acceso a datos (PDO)
│   ├── views/             # Vistas PHP
│   └── lib/               # FPDF y librerías QR
├── public/                # Punto de entrada web + recursos públicos
├── bk_basededatos.sql     # Estructura y datos base
└── README.md
```

## 3) Requisitos para ejecutar

- **Sistema operativo:** Linux (también compatible con Windows/macOS con Apache + MySQL).
- **Servidor web:** Apache 2.4+ con `mod_rewrite` habilitado.
- **PHP:** 8.0+ (recomendado 8.1 o superior).
- **MySQL:** 8.0+.
- **Extensiones PHP recomendadas:**
	- `pdo`
	- `pdo_mysql`
	- `mbstring`
	- `gd` (útil para manejo de imágenes/QR)

## 4) Guía de inicialización (Levantar el sistema)

### Paso 1. Clonar o copiar el proyecto

Ubica el proyecto en tu carpeta de trabajo, por ejemplo:

`/home/usuario/Projects/GymSystem`

### Paso 2. Configurar Apache y rutas

Este proyecto usa URLs absolutas como `/auth/index`, `/home/index`, etc.  
Para que funcionen correctamente, se recomienda usar un **VirtualHost** apuntando a la raíz del proyecto (no directamente a `public`).

Ejemplo de VirtualHost:

```apache
<VirtualHost *:80>
		ServerName gym.local
		DocumentRoot /home/usuario/Projects/GymSystem

		<Directory /home/usuario/Projects/GymSystem>
				AllowOverride All
				Require all granted
		</Directory>
</VirtualHost>
```

Luego:
- Agrega `127.0.0.1 gym.local` en `/etc/hosts`.
- Habilita `mod_rewrite` y reinicia Apache.

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Paso 3. Crear e importar la base de datos

Desde terminal:

```bash
mysql -u root -p < bk_basededatos.sql
```

El script ya crea y usa la BD `gym_system`.

### Paso 4. Ajustar credenciales de conexión

Edita `app/config/Database.php` según tu entorno:

- `host`
- `db_name`
- `username`
- `password`

Por defecto está:
- host: `localhost`
- db: `gym_system`
- usuario: `root`
- contraseña: vacía

### Paso 5. Verificar permisos de escritura

Para subida de imágenes de socios y logo, valida permisos en:

- `public/img/`
- `public/img/socios/`

En Linux (si aplica):

```bash
sudo chown -R www-data:www-data public/img
sudo chmod -R 775 public/img
```

### Paso 6. Abrir el sistema

Accede en navegador:

- `http://gym.local/`

Si todo está correcto, el sistema redirige al login.

## 5) Acceso inicial y usuarios

### Opción A: usando datos del backup SQL

El backup incluye usuarios como:
- `admin@gym.com`
- `recepcion@gym.com`
- `entrenador@gym.com`

> Las contraseñas en el dump están hasheadas; si no recuerdas la clave, usa la opción B.

### Opción B: crear/resetear administrador

1. Si dejas vacía la tabla `usuarios`, al entrar a `/auth/index` se crea automáticamente:
	 - Email: `admin@irongym.com`
	 - Password: `123456`
2. También puedes usar `public/reset.php`, pero verifica que el email objetivo exista en tu BD.

## 6) Flujo básico de uso recomendado

1. Iniciar sesión como administrador.
2. Ir a **Configuración** y completar datos del negocio (incluyendo logo y moneda).
3. Crear/validar **Planes**.
4. Registrar **Socios**.
5. Registrar **Suscripciones**.
6. Operar **Asistencia** diaria por DNI.
7. Gestionar **Caja** (apertura/cierre) y **Gastos**.

## 7) Solución rápida de problemas

- **No cargan rutas (`/auth/index`, `/home/index`)**
	- Verifica VirtualHost + `AllowOverride All` + `mod_rewrite` activo.

- **Error de conexión a BD**
	- Revisa `app/config/Database.php` y que MySQL esté encendido.

- **No se ven imágenes o no sube foto/logo**
	- Revisa permisos en `public/img`.

- **Pantalla en blanco / errores PHP**
	- Revisa logs de Apache/PHP y habilita temporalmente `display_errors` en entorno local.

## 8) Notas de mantenimiento

- No se usa Composer actualmente; librerías PDF/QR están incluidas en el repositorio.
- Para producción, reemplaza credenciales por variables de entorno y desactiva scripts de reseteo como `public/reset.php`.
- Restringe acceso público directo a utilidades de mantenimiento.