---
title: TypeScript Support
type: guide
order: 25
---

## Important 2.2 Change Notice for TS + webpack 2 users

<p class="tip">**Cette page est en cours de traduction française. Revenez une autre fois pour lire une traduction achevée ou [participez à la traduction française ici](https://github.com/vuejs-fr/vuejs.org).**</p>In Vue 2.2 we introduced dist files exposed as ES modules, which will be used by default by webpack 2. Unfortunately, this introduced an unintentional breaking change because with TypeScript + webpack 2, `import Vue = require('vue')` will now return a synthetic ES module object instead of Vue itself.

We plan to move all official declarations to use ES-style exports in the future. Please see [Recommended Configuration](#Recommended-Configuration) below on a future-proof setup.

## Official Declaration in NPM Packages

A static type system can help prevent many potential runtime errors, especially as applications grow. That's why Vue ships with [official type declarations](https://github.com/vuejs/vue/tree/dev/types) for [TypeScript](https://www.typescriptlang.org/) - not only in Vue core, but also for [vue-router](https://github.com/vuejs/vue-router/tree/dev/types) and [vuex](https://github.com/vuejs/vuex/tree/dev/types) as well.

Since these are [published on NPM](https://unpkg.com/vue/types/), and the latest TypeScript knows how to resolve type declarations in NPM packages, this means when installed via NPM, you don't need any additional tooling to use TypeScript with Vue.

## Recommended Configuration

``` js
// tsconfig.json
{
  "compilerOptions": {
    // ... other options omitted
    "allowSyntheticDefaultImports": true,
    "lib": [
      "dom",
      "es5",
      "es2015.promise"
    ]
  }
}
```

Note the `allowSyntheticDefaultImports` option allows us to use the following:

``` js
import Vue from 'vue'
```

instead of:

``` js
import Vue = require('vue')
```

The former (ES module syntax) is recommended because it is consistent with recommended plain ES usage, and in the future we are planning to move all official declarations to use ES-style exports.

In addition, if you are using TypeScript with webpack 2, the following is also recommended:

``` js
{
  "compilerOptions": {
    // ... other options omitted
    "module": "es2015",
    "moduleResolution": "node"
  }
}
```

This tells TypeScript to leave the ES module import statements intact, which in turn allows webpack 2 to take advantage of ES-module-based tree-shaking.

See [TypeScript compiler options docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html) for more details.

## Using Vue's Type Declarations

Vue's type definition exports many useful [type declarations](https://github.com/vuejs/vue/blob/dev/types/index.d.ts). For example, to annotate an exported component options object (e.g. in a `.vue` file):

``` ts
import Vue, { ComponentOptions } from 'vue'

export default {
  props: ['message'],
  template: '<span>{{ message }}</span>'
} as ComponentOptions<Vue>
```

## Class-Style Vue Components

Vue component options can easily be annotated with types:

``` ts
import Vue, { ComponentOptions }  from 'vue'

// Declare the component's type
interface MyComponent extends Vue {
  message: string
  onClick (): void
}

export default {
  template: '<button @click="onClick">Click!</button>',
  data: function () {
    return {
      message: 'Hello!'
    }
  },
  methods: {
    onClick: function () {
      // TypeScript knows that `this` is of type MyComponent
      // and that `this.message` will be a string
      window.alert(this.message)
    }
  }
// We need to explicitly annotate the exported options object
// with the MyComponent type
} as ComponentOptions<MyComponent>
```

Unfortunately, there are a few limitations here:

- __TypeScript can't infer all types from Vue's API.__ For example, it doesn't know that the `message` property returned in our `data` function will be added to the `MyComponent` instance. That means if we assigned a number or boolean value to `message`, linters and compilers wouldn't be able to raise an error, complaining that it should be a string.

- Because of the previous limitation, __annotating types like this can be verbose__. The only reason we have to manually declare `message` as a string is because TypeScript can't infer the type in this case.

Fortunately, [vue-class-component](https://github.com/vuejs/vue-class-component) can solve both of these problems. It's an official companion library that allows you to declare components as native JavaScript classes, with a `@Component` decorator. As an example, let's rewrite the above component:

``` ts
import Vue from 'vue'
import Component from 'vue-class-component'

// The @Component decorator indicates the class is a Vue component
@Component({
  // All component options are allowed in here
  template: '<button @click="onClick">Click!</button>'
})
export default class MyComponent extends Vue {
  // Initial data can be declared as instance properties
  message: string = 'Hello!'

  // Component methods can be declared as instance methods
  onClick (): void {
    window.alert(this.message)
  }
}
```

With this syntax alternative, our component definition is not only shorter, but TypeScript can also infer the types of `message` and `onClick` without explicit interface declarations. This strategy even allows you to handle types for computed properties, lifecycle hooks, and render functions. For full usage details, see [the vue-class-component docs](https://github.com/vuejs/vue-class-component#vue-class-component).
