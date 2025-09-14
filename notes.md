# rendering a component on demand

\<ember is many things, but I mainly talk about state and rendering in this talk>

\<reference to nick's talk>

Now that we know how to integrate any library with ember, let's have a look at
the opposite.

When we say we integrate a library with Ember, typically we mean that Ember
stays "in control" and calls on the library to do a certain specialized task.
In particular, this means that no matter what the library does, it's ember that
takes care of the rendering to the dom, and the state management of its
components.

There are however, a number of very useful libraries that are specifically made
for rendering things to the dom, in ways that would simply be impractical with
ember, or are otherwise so universal that it wouldn't make sense to tie them
into a frontend framework.

Examples include things like leaflet, d3, and editors like CodeMirror, Monaco,
and ProseMirror. These libraries deal with such complex rendering that they
require the host app to give them complete control over a DOM element, much like
Ember itself requires. They also often manage their own bespoke state.
What sets those libraries apart from frontend frameworks is their specialized nature, whereas frontend frameworks try to provide tools to
build "anything".

Giving a library control over a DOM element is easy to do in Ember, that's
exactly what modifiers are for, and indeed it's a standard feature in any
frontend framework.
However, where things get interesting is when those libraries provide hooks or
interfaces to customize their specialized rendering. It's a very powerful and
common pattern, because you can gradually expand the capabilities of those
libraries and tailor the UX to your needs, building on a solid default UX
experience.

d3:

```js
d3.selectAll("div").append(() => document.createElement("p"));
```

leaflet:

```js
const el = document.createElement("p");
layer.bindPopup(el)
```

codemirror tooltips:

```js

import {hoverTooltip} from "@codemirror/view"

export const wordHover = hoverTooltip((view, pos, side) => {
    /** bunch of positioning calculations omitted **/
  return {
    pos: start,
    end,
    above: true,
    create(view) {
      let dom = document.createElement("div")
      dom.textContent = text.slice(start - from, end - from)
      return {dom}
    }
  }
})
```

prosemirror nodeviews:

```js

class SectionView {
  constructor(prosemirrorNode, view, getPos, deco) {
    this.dom = document.createElement("section")
    this.header = this.dom.appendChild(document.createElement("header"))
    this.header.contentEditable = "false" 
    this.foldButton = this.header.appendChild(document.createElement("button"))
    this.foldButton.title = "Toggle section folding"
    this.foldButton.onmousedown = e => this.foldClick(view, getPos, e)
    this.contentDOM = this.dom.appendChild(document.createElement("div"))
    this.setFolded(deco.some(d => d.spec.foldSection))
  }

  setFolded(folded) {
    this.folded = folded
    this.foldButton.textContent = folded ? "▿" : "▵"
    this.contentDOM.style.display = folded ? "none" : ""
  }

  update(prosemirrorNode, deco) {
    if (prosemirrorNode.type.name != "section") return false
    let folded = deco.some(d => d.spec.foldSection)
    if (folded != this.folded) this.setFolded(folded)
    return true
  }

  foldClick(view, getPos, event) {
    event.preventDefault()
    setFolding(view, getPos(), !this.folded)
  }
}
```

What all these systems have in common is, essentially, their interface. They
all, some way or another, ask you to construct a DOM element, which they will
then render.

That's of course, obvious when you think about it. What else could they do? Ask
for a React component? That would make their library tied to react.
Some libraries actually do provide first-class support for certain popular
frameworks, but even those will usually have a fallback to a plain old DOM
element.

However, that poses a bit of a problem. As frontend developers, we are all
of course deathly allergic to constructing DOM elements by hand. No, we want to
use ember!

So how do we get the html that Ember renders when it renders a component? And,
before you get any wild ideas, how do we get this html _without_ actually
rendering it?

## Before ember 6.8

I think it's safe to assume most of us do not use bleeding edge ember versions
in production, so I will outline the method we've found which works in just
about every ember version you can think of (probably even pre-octane, as you will see). However, as \<you've probably noticed in Nullvox's talk> 6.8 introduces a native tool for this, which I will also cover.

<todo>

## After ember 6.8

<todo>

# Shipping an ember app as an installable npm package

## before vite

Building an ember app with broccoli generated an AMD bundle, which was designed to be run by
the browser. Standard bundlers could not do much with this bundle. 
<todo>

## with vite

Thanks to embroider, we can now transform ember-specific source code into
javascript which bundlers can understand. Combined with vite, we get a very
powerful build system which not only comes with an excellent dev server, but,
more importantly, also wraps and pre-configures Rollup and ESbuild.

By default, vite is set up to produce browser "apps". That is to say, bundles
with an index.html file, which are meant to be directly hosted.

However, it can also create libraries with its "library mode". This informs vite
that you're intending to ship code which should be imported from, either
directly in the browser using esmodules or umd, or in node, through commonjs or
likewise esmodules (in modern node). (Typically, the consuming node program is
then also bundled for the browser, but that's not our concern anymore).

```
Running vite build with this config uses a Rollup preset that is oriented towards shipping libraries and produces two bundle formats:

    es and umd (for single entry)
    es and cjs (for multiple entries)
```



