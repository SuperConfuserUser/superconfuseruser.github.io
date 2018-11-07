---
layout: post
title:      "Semantic CSS Frameworks with SASS"
date:       2018-11-07 16:48:41 -0500
permalink:  semantic_css_frameworks_with_sass
---


CSS frameworks are a quick and easy way to style your content. Basic styles and behaviors are taken care of, so you're free to focus more on design and customization.

I've shied away from them though because adding styling classes to your HTML elements isn't very semantic. A basic page can become littered with extra classes. 

```
     <ul class="collection with-header">
        <li class="collection-header"><h4>First Names</h4></li>
        <li class="collection-item">Alvin</li>
        <li class="collection-item">Alvin</li>
        <li class="collection-item">Alvin</li>
        <li class="collection-item">Alvin</li>
      </ul>
```

Oof, almost everything is classed. This feels very redundant when the HTML tags should be enough for differentiation.

## SASS to the Rescue
SASS is a CSS preprocessor that turns it's SCSS into regular CSS as the end. SCSS has a bunch of features like variables, nesting, importing, and operators. I've barely explored all the ways that these can be used to write more efficient and modular styling.

We'll be using few of the SASS features.

### Case Study
A main style.scss sheet will hold our styling. Regular CSS can be used along with SASS specific things. Import the CSS framework of your choice with the proper path. Files don't need the .css or .scss extension. I'll be using Materialize CSS for this example.

```
\\ style.scss


@import '/path/to/materialize-css'

```

Let's change the original html to something like this. Now, there's only the content, html tags, and one descriptive class.

```
     <ul class="first-names-list">
        <li><h4>First Names</h4></li>
        <li>Alvin</li>
        <li>Alvin</li>
        <li>Alvin</li>
        <li>Alvin</li>
      </ul>
```

To target our new ul class, let's use nesting. This is really similar to the way your html is actually nested. Levels of nesting will be up to you. I like to keep nesting shallow unless it makes sense to get more specific.

```
\\ style.scss

.first-names-list {
  
     li {

     }
	
     li:nth-child(1) {

     }
}

```

So how do we leverage the Materialize classes for style? SASS extend will let you share CSS properties from one selector to another. We can use it to add Materialize to these selectors. It's taking the classes from the materialize-css file that was imported. You can extend more than one thing, and it could also be a custom class or extension. .first-names-list is extending two classes.

```
.first-names-list {
     @extend collection, with-header;
	
     li {
        @extend collection-item;
     }
	
     li:nth-child(1) {
        @extend collection-header;
     }
}

```


### Extra
In my final project, I used partials along with a mixin to create a materialize helper. `@extend framework-class` becomes `@include materialize(framework-class)` It's a little bit more writing, but now I know that those classes are specifically from Materialize.

```
@mixin materialize($classes...) {
     @each $class in $classes {
        @extend .#{$class};
     }
}
```


### Conclusion
You should be able to use this technique with any class-based CSS framework. Most of the other popular CSS preprocessors have the same features, so any of those should be fine too. 

This mehod was shown with a very simple example but can be very useful throughout the entire app. Styling can be very DRY and is separated from the content.

