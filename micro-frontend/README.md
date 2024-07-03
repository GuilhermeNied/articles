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

## Quando devemos utilizar

Aqui vem um grande **depende**. Na minha opinião esse tipo de arquitetura deve ser aplicada

## Vantagens

## Desvantagens

## Minha experiência

Durante meu desenvolvimento de uma aplicação onde estudei todo o conceito e aplicação de um micro frontend, utilizei o
Module Federation e também já utilizei o Single SPA.
Todos os exemplo que vou utilizar são de um pequeno projeto que fiz utilizando React, Angular e Javascript Vanilla.
Vou dividir minha expêriencia de algumas partes, **Configuração**, **Desenvolvimento** e **Execução**

### Configuração

A configuração foi uma das partes mais complexas e dificieis para mim principalmente porque a maioria dos exemplos que
pesquisei pra usar como referência ou estavam desatualizados para a versão do Angular ou do React ou não tinha a
configuração que eu precisava. Outro motivo por ser mais complicado é porque quando ocorre erro de build em alguma das aplicações, na maioria das vezes o erro não é muito explícito e isso deixa um pouco mais complicado de corrigi-lo.
Então pesquisando um pouco mais e lendo a configuração montei as três configurações para utilizar JS, Angular e React.

#### Angular

```javascript
...
  plugins: [
      new ModuleFederationPlugin({
          name: "angular_module",
          filename: "remoteEntry.js",
          exposes: {
              './Component':'./src/loadApp.ts'
          },
      }),
      sharedMappings.getPlugin()
    ],
```

##### Javascript

```javascript
...
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
```

##### React

```javascript
...
  plugins: [
      new ModuleFederationPlugin({
        remotes: {
          angular_module: 'angular_module@http://localhost:4200/remoteEntry.js',
          javascript_module:
            'javascript_module@http://localhost:4300/remoteEntry.js'
        }
      })
    ],
    devServer: {
      static: {
        directory: path.join(__dirname, 'dist')
      },
      port: 3000,
      hot: true
    }
```

Vocês podem notar que a configuração do React é diferente das outras, isto é porque ele é que acopla os outros dois micro
frontends.

### Desenvolvolvimento

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

Agora, se você quiser testar, vá em frente! Utilize meu projeto como exemplo, faça coisas simples e complexas! Caso não queira passar por toda a parte de configuração, recomendo utilizar outra biblioteca para isso. Você pode usar a que preferir, mas o **<u>[Single SPA](https://single-spa.js.org/)</u>** é uma das principais libs e tem uma boa comunidade. Então, eu a recomendaria. Mas, como eu disse, pesquise e veja a que melhor se encaixa ao seu projeto.

Obrigado por chegar até aqui, e até uma próxima!

## Referências:

[Micro Frontends - extending the microservice idea to frontend development](https://micro-frontends.org/)

[Micro Frontends](https://martinfowler.com/articles/micro-frontends.html)

[Module Federation](https://webpack.js.org/concepts/module-federation/)

[Single SPA](https://single-spa.js.org/)
