# manul-app-manifest

This manifest defines the way how we create modern, modular and maintainable web-applications. It has focus on meteor&react, but can be extended to other technologies like magnolia or ruby on rails as well.

## TL;DR:

- take a short look at this [The%20Way%20of%20the%20Manul.pdf](presentation)

- We program in ES2015 (aktuelle javascript-version), following styleguide: https://github.com/airbnb/javascript, sample .eslintrc can be found here: [.eslintrc](.eslintrc)

- For meteor-apps, please read the official [http://guide.meteor.com](guide) (focus on react)

- UI is written in [https://facebook.github.io/react/](react), we prefer [https://facebook.github.io/react/docs/reusable-components.html#stateless-functions](functional and stateless components).

- We folow the [https://github.com/kadirahq/mantra](mantra)-specification. 

- mantra gives us depencency-injection and a clean architecture. It enforces distinguish between [https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.g3judhufh](*component* and *container*).

- For a sample mantra app, take a look here: [https://github.com/mantrajs/mantra-sample-blog-app](sample app)

- We believe that *inline-styling* solves a lot of problem with styling. [https://github.com/FormidableLabs/radium](radium) helps with that. We also have our own [https://git.panter.ch/manul/manul-utils/blob/master/with_theme.js](*Higher Order Component*) that defines a pattern to caluculate styles based on a theme-object and the component's properties. 



## Mantra

For Meteor we adopt the **Mantra*-Specification: https://kadirahq.github.io/mantra/

Mantra is an application architecture for Meteor and its main goals are Maintainability and Future Proof.

the Mantra-Specification is the baseline for this manifest, but we want to highlight some of its principles and show how we can use these principles in other frameworks as well:


## Everything is a component, every component works stand-alone.


Try to encapsulate UI in small components. Components should only depend on their properties and should avoid inner state.

In **React** its therefore best to use stateless components:

````

BlogPost = ({title, text, author, onEdit}) => (
    <article className="BlogPost">
      <h1>{title}</h1>
      <h2>Written by {author}</h2>
      <p>{text}</p>
      <button click={onEdit}>Edit this post</button>
    </article>
)

````
This components are also known as "pure" or "dumb" components.

With **Blaze** (Meteors default ui-library) this works similar. You create templates that only depend on their passed data-property:

````
<template name="BlogPost">
  <article className="BlogPost">
    <h1>{{post.title}}</h1>
    <h2 className="author">Written by {{post.author}}</h2>
    <p>{{post.text}}</p>
    <button>Edit this post</button>
  </article>
</template>

Template.BlogPost.events({
  ["click button"]() {
    this.onEdit();
  }
})

````

This works also similary in **Magnolia** where you can use freemarker-makros to encapsulate Parts of the UI. Magnolia has its own kind of "component", but that is more like a container (see below)

## Injecting data with Containers

Pure UI-Components should neither know how to fetch data nor have application-logic. You therefore have to inject these properties with so called **Containers**.

A Container is a Component that encapsulates a pure UI-Component and injects data and actions.

**Mantra**: use react-komposer for this. https://kadirahq.github.io/mantra/#sec-Containers

**Meteor without Mantra**: use react-komposer or meteor's createContainer (https://guide.meteor.com/react.html#using-createContainer)

**DRAFT: Magnolia**: A Magnolia-Component ususally has a dialog, a model-class or Spring-like Class in a Blossom-Component, as well as a freemarker-file. The properties of the dialog are stored in a jcr-node that is usually exposed as `content` in the freemarker-file. Because it knows how to load data, we can consider this as a container.

We did not yet establish a best-practise for this, but an idea would be to compose all your "magnolia-components" with freemarker-makros that never access a node-directly, but get there properties by arguments to the makro.



## Styling

Thinking in Components also helps with organizing styles.

### Always scope your selectors

In the sample above we wrapped the component in an element with a class "BlogPost". We can use this to prefix any styles that we define for this component. It's best to place the styles near the UI of this component:

...
BlogPost.jsx
BlogPost.scss
...

`````
//BlogPost.scss

.BlogPost {
  border: 1px dashed #333;
  padding: 10px;
  & > h1: {
    font-weight: bold;
    font-size: 22px;
    margin-bottom: 10px;
  }
  & > .author {
    font-style: italic;
  }
}

`````

Using the component-name as the base-selector prevents the styles from leaking into other components.
Using child-selectors (>) prevents also leakage into nested components.

It's also a good idea to combine this BEM (http://getbem.com/introduction/)

### React & Inline-Styles

Because we tend to address every element as explicit as possible, it's also possible to use inline-styles with react:


`````

const styles = {
  container: {
    border: `1px dashed ${Theme.bordercolor}`,
    padding: 10
  },
  title: {
    fontWeight: "bold";
    fontSize: 22,
    marginBottom: 10
  },
  author: {
    fontStyle: "italic";
  }
}

BlogPost = ({title, text, author, onEdit}) => (
    <article style={styles.container}>
      <h1 style={styles.title}>{title}</h1>
      <h2 style={styles.author}>Written by {author}</h2>
      <p>{text}</p>
      <button click={onEdit}>Edit this post</button>
    </article>
)


`````

#### Advantages:

- Highly expressive
- no more dead-code
- organize styles like code
- makes it easy to compute values
- style depending on state or properties
- you can change styles during runtime
- no more cascading :-)

#### Caveats:

- You need to learn new ways of organizing styles
- :hover and :focus does not work, however there is a good library for this: https://github.com/FormidableLabs/radium
- No more cascading :-( Makes styling child-components harder, in particular if they are not built for inline-styles.

Of course you can mix both, e.g. by using CSS for general styles like fonts, colors, etc. and inline-styles for component-specific styling.
