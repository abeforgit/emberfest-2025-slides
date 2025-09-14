---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# some information about your slides (markdown enabled)
title: ember() adventures with Ember in an imperative world
info: |
  ## ember(): adventures with Ember in an imperative world
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
seoMeta:
  # By default, Slidev will use ./og-image.png if it exists,
  # or generate one from the first slide if not found.
  ogImage: auto
  # ogImage: https://cover.sli.dev
---

# ember(): adventures with Ember in an imperative world

---

# About me

---

# Redpencil

---

# When we talk about ember...

Ember is many things:

<ul>
<li v-mark="{ color: '#FF7700', type: 'circle' }"> a DOM rendering framework (glimmer components, template tag syntax) </li>
</ul>

- a reactivity system (`@tracked`)
- a router (router.ts)
- a data-fetching framework (route classes)
- a state management system (services)
- a dependency injection framework (`owner`, `@service`, etc)

---


# The project: say-editor

A specialized wysiwyg rich-text editor, with a heavy focus on creating annotated
text.

building blocks:

- UI: ember
- text editing engine: Prosemirror
- data-parsing: combo of standard parsers and bespoke logic

---

# Prosemirror




--- 

# The project: say-editor

short demo
<Transform :scale="0.5">
<Counter />
</Transform>

---

# Integrating frameworks in Ember

See Nick's talk - potentially add some things

---

# What I mean with framework

"Things that render to the DOM"

## General-purpose

- react 
- vue   
- svelte
- ember 

## Specialized
- leaflet    
- d3         
- codemirror 
- prosemirror

---

# Frameworks can render your code

d3:

```js
d3.selectAll('div').append(() => document.createElement('p'));
```

---

# Frameworks can render your code

leaflet:

```js
const el = document.createElement('p');
layer.bindPopup(el);
```


---

# Frameworks can render your code

prosemirror nodeviews:

```js
class SectionView {
  constructor(prosemirrorNode, view, getPos, deco) {
    this.dom = document.createElement('section');
    this.header = this.dom.appendChild(document.createElement('header'));
    this.header.contentEditable = 'false';
    this.foldButton = this.header.appendChild(document.createElement('button'));
    this.foldButton.title = 'Toggle section folding';
    this.foldButton.onmousedown = (e) => this.foldClick(view, getPos, e);
    this.contentDOM = this.dom.appendChild(document.createElement('div'));
    this.setFolded(deco.some((d) => d.spec.foldSection));
  }
}
```

---

# But who wants to write plain html?

---


# Using ember to generate DOM elements

`in-element`

```gjs
destinationElement = document.createElement('div');

<template>
    {{#in-element this.destinationElement}}
      <div>Some content</div>
    {{/in-element}}
</template>

```

---

# Situations where in-element is not enough

TODO

---

# Before ember 6.8

```ts {*}{maxHeight:'400px'}
this.template = hbs`<this.component
                      @getPos={{this.getPos}}
                      @node={{this.node}}
                      @updateAttribute={{this.updateAttribute}}
                      @controller={{this.controller}}
                      @view={{this.view}}
                      @selected={{this.selected}}
                      @contentDecorations={{this.contentDecorations}}
                      @selectNode={{this.selectNode}}
                    >
                      {{#unless this.atom}}
                        {{! @glint-expect-error: not typesafe yet }}
                        <EmberNode::Slot @contentDOM={{this.contentDOM}}/>
                      {{/unless}}
                    </this.component>`;

function emberComponent(
  owner: Owner,
  name: string,
  inline: boolean,
  props: EmberNodeArgs & {
    atom: boolean;
    component: ComponentLike<{ Args: EmberNodeArgs }>;
    contentDOM?: HTMLElement;
  }
): { node: HTMLElement; component: EmberInlineComponent } {
  const componentName = `${name}-${uuidv4()}`;
  owner.register(
    `component:${componentName}`,
    // eslint-disable-next-line ember/no-classic-classes
    Component.extend({
      layout: this.template,
      tagName: '',
      ...props,
    })
  );
  const component = owner.lookup(
    `component:${componentName}`
  ) as EmberInlineComponent;
  const node = document.createElement(inline ? 'span' : 'div');
  node.classList.add('ember-node');
  component.appendTo(node);
  return { node, component };
}
```

---

# After ember 6.8

```gts {*}{maxHeight:'400px'}
function emberComponent(
  owner: Owner,
  inline: boolean,
  comp: ComponentLike<{ Args: EmberNodeArgs }>,
  props: EmberNodeArgs & {
    atom: boolean;
    component: ComponentLike<{ Args: EmberNodeArgs }>;
    contentDOM?: HTMLElement;
  },
): { node: HTMLElement; renderResult: ReturnType<typeof renderComponent> } {
  const node = document.createElement(inline ? 'span' : 'div');
  node.classList.add('ember-node');
  const renderResult = renderComponent(EmberNodeContainer, {
    into: node,
    args: trackedObject({
      ...props,
      emberNodeComponent: comp,
    }),
  });

  return { node, renderResult };
}
const EmberNodeContainer: TOC<ContainerSig> = <template>
  <@emberNodeComponent
    @getPos={{@getPos}}
    @node={{@node}}
    @updateAttribute={{@updateAttribute}}
    @controller={{@controller}}
    @view={{@view}}
    @selected={{@selected}}
    @contentDecorations={{@contentDecorations}}
    @selectNode={{@selectNode}}
  >
    {{#if @atom}}
      {{#if @contentDom}}
        <EmberNodeSlot @contentDOM={{@contentDom}} />
      {{/if}}

    {{/if}}
  </@emberNodeComponent>
</template> satisfies TOC<ContainerSig>;
```

---

# Packaging ember as a library

Making ember apps is fun, but....

"Hey that's a cool thing you've build, how can I use it in my angular/react/vue app?"

---

# Pre-vite

Building with broccoli

- `ember build` -> AMD bundle

---

# Pre-vite

Amd bundles are hard to share

```js
import vendor from './ember-build/assets/vendor.js';
import vendorBundle from './ember-build/assets/@lblod/my-app-vendor-bundle.js';
import app from './ember-build/assets/@lblod/my-app-app.js';
import myApp from './ember-build/assets/@lblod/my-app.js';
import editorCss from './ember-build/assets/@lblod/my-app.css';
import vendorCss from './ember-build/assets/vendor.css';
```

Bundlers don't understand these js files

---

# Pre-vite

Ugly solutions for ugly problems

```js
// webpack.config.js

  module: {
    rules: [
      {
        test: /ember-build\/assets\/.*\.js$/,
        type: 'asset/source',
        exclude: /node_modules/,
        // use: {
        //   loader: "babel-loader"
        // }
      },
```

---

# Pre-vite

Ugly solutions for ugly problems

```js {*}{ maxHeight: '400px' }
function renderApp() {
 const editorFrame = document.createElement('iframe');
  editorFrame.setAttribute('srcdoc', srcDoc);
  editorFrame.setAttribute('width', width);
  editorFrame.setAttribute('height', height);
  editorFrame.setAttribute('title', title ?? 'say-editor');
  editorFrame.style.overflow = 'auto';

  element.replaceChildren(editorFrame);

  // wait for the iframe to render
  await new Promise((resolve) =>
    editorFrame.addEventListener('load', resolve, { once: true }),
  );
  const frameDoc = editorFrame.contentDocument!.body;

  // append styles
  const vendorStyle = document.createElement('style');
  vendorStyle.textContent = vendorCss;
  const editorStyle = document.createElement('style');
  editorStyle.textContent = editorCss;
  frameDoc.append(editorStyle, vendorStyle);

  // append scripts
  const vendorScript = document.createElement('script');
  vendorScript.text = vendor;
  frameDoc.appendChild(vendorScript);

  const vendorBundleScript = document.createElement('script');
  vendorBundleScript.text = vendorBundle;
  frameDoc.appendChild(vendorBundleScript);

  const appScript = document.createElement('script');
  appScript.text = app as string;
  frameDoc.appendChild(appScript);

  const embeddableScript = document.createElement('script');
  embeddableScript.text = embeddable as string;
  frameDoc.appendChild(embeddableScript);
}
```

---

# With embroider and vite

Much easier

Thanks to embroider, we can now transform ember-specific JS into standard ES
modules, allowing us to use all the features of the build-system

---

# Vite configuration

vite is just rollup + esbuild, but also a lot more

By default, vite is set up to build a browser bundle, just like broccoli...

```js
import { defineConfig } from 'vite';
import { extensions, classicEmberSupport, ember } from '@embroider/vite';
import { babel } from '@rollup/plugin-babel';

export default defineConfig({
  plugins: [
    classicEmberSupport(),
    ember(),
    // extra plugins here
    babel({
      babelHelpers: 'runtime',
      extensions,
    }),
  ],
});
```
---

# Vite configuration
build output

```shellsession
$ tree dist
dist
├── assets
│   ├── main-Br2rsYWF.css
│   └── main-DEN-EFOq.js // minified
├── ember-welcome-page
│   └── images
│       └── construction.png
├── @embroider
│   └── virtual
│       ├── app.css
│       ├── vendor.css
│       └── vendor.js
├── index.html
└── robots.txt

6 directories, 8 files
```
---

# Vite configuration

...but unlike broccoli, it can easily be configured to do something else

```js {*}{ maxHeight: '400px' }
import { dirname, resolve } from 'node:path';
import { fileURLToPath } from 'node:url';
import { defineConfig } from 'vite';
import { extensions, classicEmberSupport, ember } from '@embroider/vite';
import { babel } from '@rollup/plugin-babel';

const __dirname = dirname(fileURLToPath(import.meta.url));

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'app/app.ts'),
      name: 'my-app',
      fileName: 'my-app',
    },
    rollupOptions: {
      input: 'app/app.ts',
    },
  },
  plugins: [
    // hmm... what's this?
    tweakedClassicEmber,
    ember(),
    babel({
      babelHelpers: 'runtime',
      extensions,
    }),
  ],
});
```

---

# Vite configuration

build output

![](./images/vite-dist.png)

By default, vite outputs both an es module and an umd bundle, which is perfect
for compatibility with just about anything.

---

# Package.json edits

Pointing node to the right files

```js
  "main": "./dist/my-app.umd.js",
  "module": "./dist/my-app.mjs",
```

---

# Manually booting the ember app

---

# Setting up the test-app for development

---

# Things to look out for

We break an assumption

---

# Combining with renderComponent

---
