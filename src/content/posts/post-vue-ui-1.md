---
title: 'Creamos una librería ui para vue basada en shadcn-vue CP 1'
pubDate: 2024-09-14
description: 'Este es la primera publicación de una serie de publicaciones en las que crearemos una librería ui de componentes para vue basada en shadcn-vue.'
author: 'Mario Garrido'
image:
    url: 'https://docs.astro.build/assets/full-logo-light.png'
    alt: 'El logotipo completo de Astro.'
tags: ["vue", "ui", "vue", "shadcn-vue"]
---

Voy a empezar una serie de publicaciones para documentar el proceso de creación de una librería ui de componentes usando vue 3 y shadcn vue. La idea es tener un storybook y poder publicarlo en npm.

Mi intención es documentar el proceso de creación mientras voy desarrollando la librería poco a poco.

En este primer post voy a documentar la creación del proyecto inicial con las instalaciones necesarias, un componente botón básico y su publicación a npm.

## Instalaciones e inicialización

1. **Iniciar el proyecto con vue**: En primer lugar he creado un proyecto nuevo en vue usando el comando `npm create vue@latest`. Le he dado soporte para typescript y he aañdido vitest, cypress, eslint, prettier y Vue DevTools.

2. **Tailwind y autoprefixer** Como shadcn-vue requiere de tailwind y autoprefixer hemos instalado las dos con `npm install -D tailwindcss autoprefixer` puedes verlo en detalle [aquí](https://www.shadcn-vue.com/docs/installation/vite.html).

3. **Git**: He inicializado el repositorio de git y hecho el primer commit con la configuración inicial.

4. **Ejecutar shadcn-vue**: Ejecutamos shadcn-vue con el comando `npx shadcn-vue@latest init`. Al iniciar se ejecuta un CLI para configurar shadcn-vue. Elegimos `typescript`, `vite`, estilo `neutral`, los colores `neutral`, le decimos que nuestro archivo de configuración es `./tsconfig.app.json`, nuestro archivo global de css es `src/assets/main.css`, `si` que vamos a usar css variables para los colores, especificamos la ruta de nuestro arrchivo de configuración de `tailwind.config.js`, especificamos alias para components `@/components` y utils `@/lib/utils`.

5. **Husky** Se me ha olvidado pero siempre en todos mis proyectos para asegurar la calidad de los commits y del repositorio me gusta instalar husky con conventional commits. `npm install --save-dev husky @commitlint/config-conventional @commitlint/cli`
    - Para configurar commitlint creamos el archivo `commitlint.config.js` con lo siguiente:
    ```js
    export default {
        extends: ['@commitlint/config-conventional']
    }
    ```
    - Añadimos los scripts siguientes al `package.json``
    ```json
    "release": "npm run test:unit:ci && git push --follow-tags --no-verify",
    "prepare": "husky install",
    "pre-commit": "npm run lint",
    "pre-push": "npm run release"
    ```
    - Ejecutamos husky en nuestra terminal con el comando:
    ```console
    npx husky init
    ```
    - Con esto se habrá creado la carpeta .husky y podremos crear nuestros script para el commit, pre-commit y pre-push

    - Commit
    ```console
    echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
    git add .husky/commit-msg
    ```
    - Pre-commit
    ```console
    echo "npm run pre-commit" > .husky/pre-commit
    git add .husky/pre-commit
    ```
    - Pre-push
    ```console
    echo "npm run pre-push" > .husky/pre-push
    git add .husky/pre-push
    ```
6. **Button** Ahora si, vamos a agregar el primer componente a nuestra librería. En este caso será un botón y lo harmos con el comando:
```console
npx shadcn-vue@latest add button
```
Para poder hacer commit he desactivado la regla de eslint de vue que impide crear componetes que tengan como nombre una sola palabra. Esto lo hacemos en el archivo `.eslint.cjs`:
```cjs
rules: {
    'vue/multi-word-component-names': 'off'
}
```

7. **Index** Creamos un archivo `src/index.ts` donde importaremos y exportaremos todos los componentes ui de nuestra librería.

## Configuración del build

1. Instalamos **vite-plugin-dts**, un complemento para generar definiciones de tipo para nuestra librería.

```console
npm install vite-plugin-dts --save-dev
```

2. Modificar **vite.config.ts** con lo siguiente.

```ts
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueDevTools from 'vite-plugin-vue-devtools'
import dts from 'vite-plugin-dts'
import { resolve } from 'node:path'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), dts(), vueDevTools()],
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      name: 'ExampleVueUIComponents',
      fileName: 'example-vue-ui-components'
    },
    rollupOptions: {
      external: ['vue'],
      output: {
        globals: {
          vue: 'Vue'
        }
      }
    }
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

En el `build` le estamos diciendo a vite que construya una librerría. 
En el objeto `lib` está la configuración para la librería con las siguientes propiedades:
- `entry`: El archivo de entrada a nuestra librería (en nuestro caso a `index.ts`).
- `name`: El nombre de la librería.
- `fileName`: El nombre del archivo de salida.

En el objeto `rollupOptions` se utiliza para personalizar la configuración de Rollup, que es el bundler subyacente que Vite utiliza para empaquetar los módulos.

- `external`: Esta propiedad le dice a Rollup que no incluya ciertas dependencias en el bundle final. En este caso, estamos diciendo que vue es una dependencia externa. Esto es útil cuando estás construyendo una librería y no quieres que ciertas dependencias se incluyan en el bundle, porque esperas que el consumidor de tu librería ya tenga esas dependencias instaladas.

- `output`: Este objeto contiene opciones relacionadas con la salida del bundle.
    - `globals`: Esta propiedad se utiliza para proporcionar nombres globales para las dependencias externas cuando se está construyendo un bundle en formato UMD o IIFE. En este caso, estamos diciendo que la dependencia `vue` debe estar disponible globalmente bajo el nombre `Vue`. Esto es necesario para que el bundle sepa cómo referirse a las dependencias externas cuando se ejecuta en un entorno donde `Vue` está disponible globalmente (por ejemplo, en un navegador).

En resumen, `rollupOptions` está configurando **Rollup** para que trate `vue` como una dependencia externa y use el nombre global `Vue` para ella en el bundle final. Esto es útil para evitar incluir `vue` en el bundle de la librería, reduciendo así el tamaño del bundle y evitando conflictos de versiones.

## Edición del archivo package.json

Editamos el archivo `package.json`:

```json
{
  "name": "example-vue-ui-components",
  "version": "0.0.0",
  "description": "A ui components library for Vue 3 based on shadcn-vue",
  "private": true,
  "type": "module",
  "files": [
    "dist",
    "src/components/"
  ],
  "main": "./dist/example-vue-ui-components.umd.cjs",
  "module": "./dist/example-vue-ui-components.js",
  "exports": {
    ".": {
      "import": "./dist/example-vue-ui-components.js",
      "require": "./dist/example-vue-ui-components.umd.cjs"
    }
  },
  "types": "./dist/index.d.ts",
  "license": "MIT",
  "peerDependencies": {
    "vue": "^3.4.29"
  },
  "scripts": {
    "dev": "vite",
    "build": "run-p type-check \"build-only {@}\" --",
    "preview": "vite preview",
    "test:unit": "vitest",
    "test:unit:ci": "vitest --run",
    "test:e2e": "start-server-and-test preview http://localhost:4173 'cypress run --e2e'",
    "test:e2e:dev": "start-server-and-test 'vite dev --port 4173' http://localhost:4173 'cypress open --e2e'",
    "build-only": "vite build",
    "type-check": "vue-tsc --build --force",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix --ignore-path .gitignore",
    "format": "prettier --write src/",
    "release": "npm run test:unit:ci && git push --follow-tags --no-verify",
    "prepare": "husky",
    "pre-commit": "npm run lint",
    "pre-push": "npm run release"
  },
  "dependencies": {
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.1",
    "lucide-vue-next": "^0.441.0",
    "radix-vue": "^1.9.5",
    "tailwind-merge": "^2.5.2",
    "tailwindcss-animate": "^1.0.7",
    "vue": "^3.4.29"
  },
  "devDependencies": {
    "@commitlint/cli": "^19.5.0",
    "@commitlint/config-conventional": "^19.5.0",
    "@rushstack/eslint-patch": "^1.8.0",
    "@tsconfig/node20": "^20.1.4",
    "@types/jsdom": "^21.1.7",
    "@types/node": "^20.16.5",
    "@vitejs/plugin-vue": "^5.0.5",
    "@vue/eslint-config-prettier": "^9.0.0",
    "@vue/eslint-config-typescript": "^13.0.0",
    "@vue/test-utils": "^2.4.6",
    "@vue/tsconfig": "^0.5.1",
    "autoprefixer": "^10.4.20",
    "cypress": "^13.12.0",
    "eslint": "^8.57.0",
    "eslint-plugin-cypress": "^3.3.0",
    "eslint-plugin-vue": "^9.23.0",
    "husky": "^9.1.6",
    "jsdom": "^24.1.0",
    "npm-run-all2": "^6.2.0",
    "prettier": "^3.2.5",
    "start-server-and-test": "^2.0.4",
    "tailwindcss": "^3.4.11",
    "typescript": "~5.4.0",
    "vite": "^5.3.1",
    "vite-plugin-dts": "^4.2.1",
    "vite-plugin-vue-devtools": "^7.3.1",
    "vitest": "^1.6.0",
    "vue-tsc": "^2.0.21"
  }
}
```

Añadimos `vue`a `peerDependencies` para indicar que debemos tener instalado `vue` para que la librería pueda funcionar.

Las propiedades `main`, `module` y `exports` en el archivo package.json son utilizadas para definir los puntos de entrada de tu paquete en diferentes entornos y configuraciones. Aquí te explico cada una de ellas:

- `main`:
    - Define el punto de entrada principal de tu paquete para entornos CommonJS.
    - Es la ruta al archivo que se cargará cuando alguien importe tu paquete usando require.
    - En nuestro caso, está configurado como `./dist/example-vue-ui-components.umd.cjs`, lo que significa que este archivo será el punto de entrada principal para usuarios que utilicen CommonJS.

    ```json
    "main": "./dist/example-vue-ui-components.umd.cjs"
    ```

- `module`:
    - Define el punto de entrada principal de tu paquete para entornos ES Module (ESM).
    - Es la ruta al archivo que se cargará cuando alguien importe tu paquete usando import.
    - En nuestro caso, está configurado como `./dist/example-vue-ui-components.js`, lo que significa que este archivo será el punto de entrada principal para usuarios que utilicen ES Modules.
    ```json
    "module": "./dist/example-vue-ui-components.js"
    ```
- `exports`:
    - Proporciona una forma más avanzada y flexible de definir los puntos de entrada de tu paquete.
    - Permite especificar diferentes puntos de entrada para diferentes tipos de importaciones (`import`, `require`, etc.).
    - En nuestro caso, está configurado para proporcionar diferentes archivos dependiendo de si el paquete se importa usando `import` o `require`.
    ```json
    "exports": {
        ".": {
            "import": "./dist/example-vue-ui-components.js",
            "require": "./dist/example-vue-ui-components.umd.cjs"
        }
    }
    ```
    - Esto significa que:
        - Si alguien importa tu paquete usando import, se utilizará el archivo `./dist/example-vue-ui-components.js`.
        - Si alguien importa tu paquete usando require, se utilizará el archivo `./dist/example-vue-ui-components.umd.cjs`.

En resumen, estas propiedades permiten que tu paquete sea compatible con diferentes sistemas de módulos y entornos, asegurando que los usuarios puedan importar tu librería de la manera que mejor se adapte a su configuración.


