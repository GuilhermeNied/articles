# Colocando Micro Frontend em prática

Buenas!
Aqui nesse artigo venho trazer um pouco da minha experiência com micro frontend, vou contar um pouco de como foi
construir uma aplicação simples, como foi configurar, quais pontos positivos e negativos eu percebi, algumas dicas e
um pouco da minha visão.

## O que é esse raio de Micro Front-end

Bom pra começar com o pé direito, precisamos entender o que é e para que serve. Vamos começar o que é Micro Frontend.
Micro Frontend é um conceito(ou arquitetura) muito parecido com micro serviço, onde podemos separar uma grande aplicação
em pequenas partes onde cada micro frontend é responsável por uma funcionalidade específica de uma aplicação.
Uma boa imagem para ajudar a entender é esta, onde podemos ver times(Team Product, Team Checkout e Team Inspire) que
trabalham em diferentes partes de um produto e assim podendo dividir as funcionalidades:
<img src="https://micro-frontends.org/ressources/screen/three-teams.png">

## Para que serve

Normalmente o os micro frontends são utilizados para resolver problemas como escalabilidade, manutenção e desenvolvimento
de aplicações grande e complexas. Isso permite que times diferentes trabalhem em diferentes partes de uma grande aplicação
sem comprometer toda a aplicação e sem atrapalhar o desenvolvimento de outras funcionalidades.

## Vantagens

As principais vantagens do uso de micro frontend são:

- A modularidade de diferentes partes de uma aplicação;
- A utilização de diferentes frameworks para a criação de diferentes partes de uma aplicação;
- Uma autonomia das equipes, fazendo com que as equipes possam trabalhar em diversas partes de uma aplicaçao independentemente;

## Desvantagens

As principais desvantagens do uso de micro frontend são:

- A complexidade da impletação é um grande desafio, conseguirmos compartilhar estados e outros dados em diferentes micro frontends é uma grande barreira na hora de implementar uma solução utilizando essa arquitetura.
- O desafio de seguir o mesmo design é uma grande barreira, principalmente quando não temos um Design System.

## Minha experiência

Durante meu desenvolvimento de uma aplicação onde estudei todo o conceito e aplicação de um micro frontend, utilizei o
Module Federation e também já utilizei o Single SPA.
Todos os exemplo que vou utilizar são de um pequeno projeto que fiz utilizando React, Angular e Javascript Vanilla.
Vou dividir minha expêriencia de algumas partes, **Configuração**, **Desenvolvimento** e **Execução**

## Tutorial

Para começar precisamos criar 3 apliações, uma em React(que é a minha aplicação principal), outra em Angular e outra com JS.

### Na aplicação em React eu criei tudo do zero, então:

1. Primeiro eu iniciei uma aplicação com `npm init -y`
2. Depois fiz a instalação do react e react-dom: `npm i react react-dom`
3. Após isso instalei esta lista de depêndencias, todas elas como dev dependencies:
   - @babel/core
   - @babel/preset-env
   - @babel/preset-react
   - babel-loader
   - css-loader
   - html-webpack-plugin
   - style-loader
   - webpack
   - webpack-cli
   - webpack-dev-server
   - html-webpack-plugin
     Para facilitar `npm i @babel/core @babel/preset-env @babel/preset-react babel-loader css-loader html-webpack-plugin style-loader webpack webpack-cli webpack-dev-server html-webpack-plugin -D`
4. Agora criei todo o necessário para a aplicação funcionar o App.jsx e o index.js
   Exemplos:

   ```javascript
   import React from 'react'
   import ReactDOM from 'react-dom/client'
   import App from './App'

   const root = ReactDOM.createRoot(document.getElementById('root'))
   root.render(
     <React.StrictMode>
       <App />
     </React.StrictMode>
   )
   ```

   ```javascript
   import React from 'react'

   function App() {
     return (
       <div>
         <h1>Olá, Mundo!</h1>
         <p>Bem-vindo à minha aplicação React.</p>
       </div>
     )
   }

   export default App
   ```

5. Também é preciso criar um arquivo index.html assim como no exemplo a seguir:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

6. Após tudo isso eu criei o arquivo webpack.config.js
7. Para rodar o projeto React faça o build e depois rode o start

```javascript
const path = require('path')
const { ModuleFederationPlugin } = require('webpack').container
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './src/index.js',
  resolve: {
    extensions: ['.js', '.jsx', '.ts']
  },
  optimization: {
    minimize: false
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: 'auto'
  },
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react']
          }
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.jsx']
  },
  plugins: [
    new ModuleFederationPlugin({
      remotes: {
        angular_module: 'angular_module@http://localhost:4200/remoteEntry.js',
        javascript_module:
          'javascript_module@http://localhost:4300/remoteEntry.js'
      }
    }),
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ],
  devServer: {
    static: {
      directory: path.join(__dirname, 'dist')
    },
    port: 3000,
    hot: true
  }
}
```

6. Após isso adicionei esses dois scripts no meu arquivo package.json

```
"scripts": {
    "start": "webpack serve --mode development",
    "build": "webpack --mode production"
},
```

### Na aplicação JS

1. Primeiro eu iniciei uma aplicação com `npm init -y`
2. Após isso instalei esta lista de depêndencias, todas elas como dev dependencies:

   - @babel/core
   - @babel/preset-env
   - babel-loader
   - css-loader
   - html-webpack-plugin
   - style-loader
   - webpack
   - webpack-cli
   - webpack-dev-server
   - html-webpack-plugin
     Para facilitar `npm i @babel/core @babel/preset-env babel-loader css-loader html-webpack-plugin style-loader webpack webpack-cli webpack-dev-server html-webpack-plugin -D`

3. Fiz a criação do componente que desejo expor, no meu caso o `header.js`
   Ex:

```javascript
function createHeader() {
  const element = document.createElement('header')
  element.innerHTML = '<h1 class="header">MC</h1>'
  return element
}

export default createHeader
```

4. Após isso fiz a criação da função que vou expor para montar o meu componente no meu module pai, no meu caso criei como index.js, porém os nomes dos arquivos não importa é só cuidar para que seja exposto corretamente

```javascript
import createHeader from './header.js'
export function mountHeader(containerId) {
  const container = document.getElementById(containerId)
  if (container) {
    container.appendChild(createHeader())
  }
}
```

5. Depois de tudo isso fiz a criação do `webpack.config.js`

```javascript
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin')

module.exports = {
  entry: './src/index.js',
  mode: 'development',
  output: {
    publicPath: 'auto'
  },
  devServer: {
    static: {
      directory: path.join(__dirname, 'dist')
    },
    port: 4300,
    hot: true
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'javascript_module',
      filename: 'remoteEntry.js',
      exposes: {
        './Header': './src/index.js'
      }
    }),
    new HtmlWebpackPlugin({
      template: './index.html'
    })
  ]
}
```

6. Após tudo isso adicionei o script no meu `package.json`

```json
"scripts": {
    "start": "webpack serve --mode development --open"
},
```

### Na aplicação Angular

1. Criei um projeto com a CLI do Angular
   **É muito importante que o projeto seja criado com CSS**
   Estou utilizando esta versão da CLI:

   - Angular CLI: 17.0.8

2. Instalei as dependencias como desenvolvimento

- webpack
- webpack-cli
- webpack-merge
- html-webpack-plugin
  Para facilitar `npm i webpack webpack-cli webpack-merge html-webpack-plugin ngx-build-plus -D`

3. Instalei a dependencia

- @angular-architects/module-federation
  Para facilitar `npm i @angular-architects/module-federation`

4. Criei o arquivo `webpack.config.js` na raiz do projeto

```javascript
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin')
const mf = require('@angular-architects/module-federation/webpack')
const path = require('path')
const sharedMappings = new mf.SharedMappings()
sharedMappings.register(path.join(__dirname, 'tsconfig.json'))

module.exports = {
  output: {
    uniqueName: 'angular_module',
    publicPath: 'auto',
    scriptType: 'text/javascript'
  },
  optimization: {
    runtimeChunk: false
  },
  resolve: {
    alias: {
      ...sharedMappings.getAliases()
    }
  },
  experiments: {
    outputModule: true
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'angular_module',
      filename: 'remoteEntry.js',
      exposes: {
        './Component': './src/loadApp.ts'
      }
    }),
    sharedMappings.getPlugin()
  ]
}
```

5. Colei todo esse json no angular.json.

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "angular_module": {
      "projectType": "application",
      "schematics": {},
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "ngx-build-plus:browser",
          "options": {
            "outputPath": "dist/angular_module",
            "index": "src/index.html",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.css"],
            "scripts": [],
            "main": "src/main.ts",
            "extraWebpackConfig": "webpack.config.js",
            "commonChunk": false
          },
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "2kb",
                  "maximumError": "4kb"
                }
              ],
              "outputHashing": "all",
              "extraWebpackConfig": "webpack.prod.config.js"
            },
            "development": {
              "optimization": false,
              "extractLicenses": false,
              "sourceMap": true
            }
          },
          "defaultConfiguration": "production"
        },
        "serve": {
          "builder": "ngx-build-plus:dev-server",
          "configurations": {
            "production": {
              "buildTarget": "angular_module:build:production",
              "extraWebpackConfig": "webpack.prod.config.js"
            },
            "development": {
              "buildTarget": "angular_module:build:development"
            }
          },
          "defaultConfiguration": "development",
          "options": {
            "port": 4200,
            "publicHost": "http://localhost:4200",
            "extraWebpackConfig": "webpack.config.js"
          }
        },
        "extract-i18n": {
          "builder": "ngx-build-plus:extract-i18n",
          "options": {
            "buildTarget": "angular_module:build",
            "extraWebpackConfig": "webpack.config.js"
          }
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "polyfills": ["zone.js", "zone.js/testing"],
            "tsConfig": "tsconfig.spec.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.css"],
            "scripts": []
          }
        }
      }
    }
  }
}
```

6. Adicionei este script no package.sjon

```json
    "run:all": "node node_modules @angular-architects/module-federation/src/server/mf-dev-server.js"
```

7. Na pasta src criei o arquivo loadApp.ts

```typescript
import 'zone.js'

import { AppModule } from './app/app.module'
import { platformBrowser } from '@angular/platform-browser'

const mount = () => {
  platformBrowser()
    .bootstrapModule(AppModule)
    .catch(err => console.error(err))
}

export { mount }
```

8. Após isso configure os seguintes arquivos desta maneira

- app-routing.module.ts

```typescript
import { NgModule } from '@angular/core'
import { RouterModule, Routes } from '@angular/router'
import { AppComponent } from './app.component'

const routes: Routes = [
  {
    path: '',
    component: AppComponent
  }
]

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

- app.component.ts

```typescript
import { Component } from '@angular/core'
import { CommonModule } from '@angular/common'
import { RouterOutlet } from '@angular/router'

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  title = 'angular_module'
}
```

- app.module.ts

```typescript
import { NgModule } from '@angular/core'

import { AppComponent } from './app.component'
import { CommonModule } from '@angular/common'
import { AppRoutingModule } from './app-routing.module'
import { BrowserModule } from '@angular/platform-browser'
@NgModule({
  declarations: [AppComponent],
  imports: [CommonModule, AppRoutingModule, BrowserModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

- main.ts

```typescript
import { platformBrowser } from '@angular/platform-browser'

import { AppModule } from './app/app.module'

platformBrowser()
  .bootstrapModule(AppModule)
  .catch(err => console.error(err))
```

- tsconfig.app.json

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./out-tsc/app",
    "types": []
  },
  "files": ["src/main.ts", "src/loadApp.ts"],
  "include": ["src/**/*.d.ts"]
}
```

- tsconfig.json

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "./dist/out-tsc",
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "declaration": false,
    "experimentalDecorators": true,
    "moduleResolution": "node",
    "importHelpers": true,
    "target": "ES2022",
    "module": "ES2022",
    "useDefineForClassFields": false,
    "lib": ["ES2022", "dom"]
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

### Plugando tudo

1. Na aplicação container(React), crie uma pasta module e crie estes arquivos

- Header.jsx

```jsx
import React from 'react'
import { useEffect } from 'react'

export default function Header() {
  useEffect(() => {
    import('javascript_module/Header').then(module => {
      module.mountHeader('js-widget-container')
    })
  }, [])
  return <header id="js-widget-container"></header>
}
```

- Angular.jsx

```jsx
import React from 'react'

import { useEffect, useRef } from 'react'

const AngularModule = () => {
  const ref = useRef(null)
  useEffect(() => {
    import('angular_module/Component').then(module => {
      module.mount()
    })
  }, [])
  return (
    <div className="left-sidebar-module" ref={ref}>
      <app-root></app-root>
    </div>
  )
}

export default AngularModule
```

#### Para rodar utilize os comandos de start e run:all(para o Angular)

### Desenvolvimento

O desenvolvimento foi bastante tranquilo, principalmente por ele ser igual ao que fazemos hoje com React, Angular. Só o
Javascript tem uma peculiaridade, porque o Webpack que é quem faz o build da aplicação não exportar HTML, existem diversas
maneiras de exportar uma aplicação ou só um componente utilizando JS, eu utilizei está aqui principlamente pela sua simplicidade
e facilidade.

Aqui eu tenho meu header que é o componente que quero exportar:

```javascript
import './styles.css'

function createHeader() {
  const element = document.createElement('header')
  element.innerHTML = '<h1 class="header">MC</h1>'
  return element
}

export default createHeader
```

E aqui a função que exporto e utilizo para montar o componente

```javascript
import createHeader from './header.js'

export function mountHeader(containerId) {
  const container = document.getElementById(containerId)
  if (container) {
    container.appendChild(createHeader())
  }
}
```

### Execução

Para executar o React e JS eu utilizei o concurrently para não precisar executar três scripts separados. Como o Angular utiliza
uma o `angular-architects` deixei ele pra executar separado, principalmente por conta de que quando eu executavfa todos juntos
o Angular acabava dando um conflito e dava um crash na aplicação.
Toda a parte de execução foi bem simples e fácil, porque toda a parte de configuração já estava pronta e os problemas
já estavam corrigidos.

### Conclusão

Bom, aqui vai muito da minha opinião. É incrível o que podemos fazer com as ferramentas de micro frontend. Eu fiz algo muito simples porque a ideia era colocar em prática muitos dos estudos teóricos que eu já havia feito, mas realmente conseguimos fazer aplicações escaláveis e com uma boa separação de funcionalidades. Isso deixa as equipes de desenvolvimento muito mais confortáveis, pois estão correndo menos riscos ao mexer em apenas uma parte da aplicação, em vez de toda ela.

Agora fica a pergunta: Micro frontend serve para todos os casos?

A resposta é não. Mesmo que tenhamos aplicações escaláveis, ele se torna muito complexo e difícil de ser acoplado em aplicações legadas. Não quer dizer que não podemos fazer, mas será um grande trabalho que deve começar pelo estudo da aplicação que já está em produção. Para projetos menores, cujo intuito não seja estudar ou colocar em prática conceitos de micro frontend, não vejo sentido em usar essa abordagem. Como será um projeto pequeno, provavelmente não terá muitos módulos a serem separados.

Um possível problema que devemos ter em mente é que utilizar diferentes frameworks ou formas de desenvolver os módulos acaba deixando o desenvolvimento e a escalabilidade mais complexos.

Agora, se você quiser testar, vá em frente! Utilize meu projeto como exemplo, faça coisas simples e complexas! Caso não queira passar por toda a parte de configuração, recomendo utilizar outra biblioteca para isso. Você pode usar a que preferir, mas o **<u>[Single SPA](https://single-spa.js.org/)</u>** é uma das principais libs e tem uma boa comunidade. Então, eu a recomendaria. Mas, como eu disse, pesquise e veja a que melhor se encaixa ao seu projeto.

Obrigado por chegar até aqui, e até uma próxima!

## Referências:

[Micro Frontends - extending the microservice idea to frontend development](https://micro-frontends.org/)

[Micro Frontends](https://martinfowler.com/articles/micro-frontends.html)

[Module Federation](https://webpack.js.org/concepts/module-federation/)

[Single SPA](https://single-spa.js.org/)
