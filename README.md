# Aplicación Laravel con Docker y Vite

Este repositorio contiene una aplicación Laravel configurada para ejecutarse en un entorno Docker con Vite como el agrupador de activos. Esto es con la finalidad de trabajar sin instalar nada más que Docker.

Probado en una MacBook Air M1.

## Prerrequisitos

- Docker
- Docker Compose

## Comenzando

### Clonar el Repositorio

```bash
git clone https://github.com/lonkonao/skeleton_laravel_vite_docker
```

### Construir e Iniciar los Contenedores

```bash
docker-compose up -d
```

### Instalar Dependencias e Iniciar el Servidor de Desarrollo de Vite

1. Accede al contenedor `myapp`:

   ```bash
   docker-compose exec myapp bash
   ```

2. Instala las dependencias e inicia el servidor de desarrollo de Vite:

   ```bash
   npm install
   npm run dev
   ```

### Acceder a la Aplicación

- La aplicación Laravel estará disponible en: `http://localhost:8000`
- El servidor de desarrollo de Vite estará disponible en: `http://localhost:5173`

## Configuración

### Configuración de Docker Compose

El archivo `docker-compose.yml` está configurado para establecer los siguientes servicios:

- **mariadb:** Servidor de base de datos MariaDB
- **myapp:** Servidor de la aplicación Laravel

```yaml
version: "3.8"

services:
  mariadb:
    image: docker.io/bitnami/mariadb:10.6
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_myapp
      - MARIADB_DATABASE=bitnami_myapp
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/bitnami/mariadb

  myapp:
    image: docker.io/bitnami/laravel:9
    ports:
      - "8000:8000"
      - "5173:5173"
    environment:
      - DB_HOST=mariadb
      - DB_PORT=3306
      - DB_USERNAME=bn_myapp
      - DB_DATABASE=bitnami_myapp
    volumes:
      - "./my-project:/app"
    depends_on:
      - mariadb
    working_dir: /app
    command: >
      /bin/bash -c "npm install && npm run dev"

volumes:
  mariadb_data:
```

### Configuración de Vite

### Configuración de Vite

El archivo `vite.config.js` está configurado para manejar los compilados de desarrollo y producción:

```javascript
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [
    laravel({
      input: ["resources/sass/app.scss", "resources/js/app.js"],
      refresh: true,
    }),
  ],
  server: {
    host: "0.0.0.0",
    port: 5173,
  },
  build: {
    manifest: true,
    outDir: "public/build",
    rollupOptions: {
      input: {
        app: "resources/js/app.js",
      },
    },
  },
});
```

### Integración de Laravel Blade

Asegúrate de que tu plantilla Blade utilice la directiva `@vite` para incluir los activos gestionados por Vite:

```blade
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Fuentes -->
    <link rel="dns-prefetch" href="//fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=Nunito" rel="stylesheet">

    <!-- Scripts -->
    @vite(['resources/sass/app.scss', 'resources/js/app.js'])
</head>
<body>
    <div id="app">
        <nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
            <div class="container">
                <a class="navbar-brand" href="{{ url('/') }}">
                    {{ config('app.name', 'Laravel') }}
                </a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="{{ __('Toggle navigation') }}">
                    <span class="navbar-toggler-icon"></span>
                </button>

                <div class="collapse navbar-collapse" id="navbarSupportedContent">
                    <!-- Lado Izquierdo del Navbar -->
                    <ul class="navbar-nav me-auto">

                    </ul>

                    <!-- Lado Derecho del Navbar -->
                    <ul class="navbar-nav ms-auto">
                        <!-- Enlaces de Autenticación -->
                        @guest
                            @if (Route::has('login'))
                                <li class="nav-item">
                                    <a class="nav-link" href="{{ route('login') }}">{{ __('Login') }}</a>
                                </li>
                            @endif

                            @if (Route::has('register'))
                                <li class="nav-item">
                                    <a class="nav-link" href="{{ route('register') }}">{{ __('Register') }}</a>
                                </li>
                            @endif
                        @else
                            <li class="nav-item dropdown">
                                <a id="navbarDropdown" class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
                                    {{ Auth::user()->name }}
                                </a>

                                <div class="dropdown-menu dropdown-menu-end" aria-labelledby="navbarDropdown">
                                    <a class="dropdown-item" href="{{ route('logout') }}"
                                       onclick="event.preventDefault();
                                                     document.getElementById('logout-form').submit();">
                                        {{ __('Logout') }}
                                    </a>

                                    <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
                                        @csrf
                                    </form>
                                </div>
                            </li>
                        @endguest
                    </ul>
                </div>
            </div>
        </nav>

        <main class="py-4">
            @yield('content')
        </main>
    </div>
</body>
</html>
```
