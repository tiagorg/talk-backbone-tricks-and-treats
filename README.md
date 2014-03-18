<!--

WARNING!! DON'T EDIT THE FILE README.md on the root of the project, that one is a GENERATED FILE!

You should just edit the source file at src/README.md - the one which stars with ## Backbone.js tricks or treats

-->

## Backbone.js tricks or treats

<img src="img/cover.jpg" class="logo" />

Tiago Garcia @ [Avenue Code](http://www.avenuecode.com)

*tgarcia@avenuecode.com*

Mar 25th, 2014

---

## Agenda

 - The jQuery Way
 - Views and Memory leaks
 - Overwhelming the DOM
 - Nesting views
 - Application and Modules
 - Router vs Controller
 - Tight coupling
   - Pub/sub

----

## Agenda

 - Callback hell
   - Promises and deferring
 - Data binding
   - Epoxy.js
 - Slow tests
   - Sinon.JS

---

## Prerequisites

- Backbone.js
- Design patterns for large-scale javascript
- Jasmine

---

## The jQuery Way

- Backbone depends on jQuery, but it shouldn't mean abusing.
- Developers coming from strong jQuery background insist on *jQuerizing*, while Backbone provides structure to avoid that:
  - AJAX belongs to the Model and *SHOULD NOT* be coded like *`$.ajax()`*.
  - DOM events binding belongs to the View and *SHOULD NOT* be coded like *`$(el).click(...)`*.
- This is a common scenario in code migrations to Backbone, but simple to fix. Just have the Models and Views to do their work.
- Follow [Step by step from jQuery to Backbone](https://github.com/kjbekkelund/writings/blob/master/published/understanding-backbone.md) to better understand this process.

---

## Views and Memory leaks

- Backbone leaves much of the code structure for to the developer to define and implement.
- Bad designs easily lead to memory leaks.
```javascript
  var MyView = Backbone.View.extend({
    initialize: function() {
      this.model.on('change', this.render, this); // Data binding
    },

    render: function() {
      alert('Rendering the view');
    }
  });
```
- If instantiated twice, the 1st View will be never Garbage Collected, once model keeps a reference for it (*Zombie View*).
- This can cause *side-effects* - alert box will appear twice.

----

## Manual approach

- To fix it, we just need a method to *unbind* the View:
```javascript
    close: function() {
      // Unbind the events that this view is listening to
      this.stopListening();
    }
```
- However, we must remember to manually call this method whenever we destroy a View.
- Good practice: use a *Manager* to maintain the current View:
```javascript
    showView: function(view) {
      if (this.currentView) {
        this.currentView.close();
      }
      this.currentView = view;
      this.currentView.render();
      $("#mainContent").html(this.currentView.el);
    }
```

----

## Marionette.js

<img src="img/marionette.png" class="marionette" />

<ul class="full">
  <li>A Backbone.js composite application library to <br/>provide structure for large-scale Javascript.</li>
  <li>Includes good practices and design & <br/>implementation patterns.</li>
  <li>Reduces code boilerplate.</li>
  <li>Provides a modular architecture framework <br/>with a Pub/Sub implementation.</li>
  <li>And much more...</li>
</ul>

----

## Marionette's ItemView

- *Marionette.ItemView* extends *Backbone.View* and automates the rendering of a single item (Model or Collection).
- It implements *render()* for you, applying a given *template* to a Model/Collection.
- Using *listenTo()* instead of *on()* for binding events, you no longer need to manually invoke a *close* method.
```javascript
  var MyView = Marionette.ItemView.extend({
    template: '#my-ujs-template', // Underscore.js template
    template: Handlebars.compile($("#my-hbs-template").html()), // Handlebars.js template

    initialize: function() {
      this.listenTo(this.model, 'change', this.render);
    }

    // No render() anymore!! :)
  });
```

----

## Marionette's Region

- *Marionette.Region* is a Views container and manager.
- It manages their lifecycles and proper display on a DOM element and closing (no more Zombie Views).
```javascript
    var myRegion = new Marionette.Region({
      el: '#content'
    });

    var view1 = new MyView({ /* ... */ });
    myRegion.show(view1); // myRegion yields view and populates the DOM

    var view2 = new MyView({ /* ... */ });
    myRegion.show(view2); // myRegion yields view and populates the DOM
```

---

## Overwhelming the DOM

- In a View which renders a Collection, we normally use a child View for each item and append it to the parent, like:
```javascript
    var CollectionView = Backbone.View.extend({
      render: function() {
        _.each(this.collection.models, function(item) {
          var view = new MyView({
            model: item
          });

          this.$el.append(view.render().el); // Populating the DOM
        }, this);
      }
    });
```
- If the Collection has N items, this code makes N operations on the DOM, which is *expensive*. Imagine N = 1000?


----

## Manual approach

- A better approach is to append to a *document fragment* instead, and just add the fragment *once* to the DOM:
```javascript
    var CollectionView = Backbone.View.extend({
      render: function() {
        var fragment = document.createDocumentFragment();

        _.each(this.collection.models, function(item) {
          var view = new MyView({
            model: item
          });

          fragment.appendChild(view.render().el); // Appending to fragment
        }, this);

        this.$el.html(fragment); // Populating the DOM
      }
    });
```

----

## Marionette's CollectionView

- *Marionette.CollectionView* renders a Collection and uses a *Marionette.ItemView* for each item renderization. It doesn't need a template for itself.
- Uses a *document fragment* internally.
```javascript
  var MyView = Marionette.CollectionView.extend({
    itemView: MyView

    // No render() anymore!! :)
  });
```

----

## Marionette's CompositeView

- *Marionette.CompositeView* is similar to a *Marionette.CollectionView* but also takes a template for itself. Designed for parent-child relationships.
- Useful to build hierarchical and recursive structures like *trees*.
```javascript
  var MyView = Marionette.CompositeView.extend({
    itemView: MyView,
    template: "#node-template", // Template for the parent
    itemViewContainer: "tbody" // Where to put the itemView instances into

    // No render() anymore!! :)
  });
```

---

## Nesting views

- Usual view nesting:
```javascript
  var OuterView = Backbone.View.extend({
    render: function() {
      this.$el.append(template);

      // Inner view
      this.innerView = new InnerView();
      this.innerView.render();
      this.$('#some-container').append(this.innerView.$el);
    }
  });
```
- Every call to *render()* will instantiate again the inner view, and rebind the events.
- The previous inner views have potential to be Zombies.
- Inner view is manually created, but never manually disposed.

----

## Manual approach

- A better approach to improve performance & avoid Zombies.
```javascript
  var OuterView = Backbone.View.extend({
    initialize: function() {
      this.inner = new InnerView(); // Instantiated just once
    },

    render: function() {
      this.$el.append(template);
      this.$('#some-container').append(this.innerView.el);
      this.inner.render();
    },

    // Needs to be manually invoked before removing
    close: function() {
      this.inner.remove();
    }
  });
```

----

## Manual approach

- A better approach to improve performance & avoid Zombies.
```javascript
  var InnerView = Backbone.View.extend({
    render: function() {
      this.$el.html(template);

      // Needed to bind events to new DOM
      this.delegateEvents();
    }
  });
```
- This still can be a mess for too many inner views.

----

## Marionette's Layout

- *Marionette.Layout* extends from *Marionette.ItemView* but provides embedded *Marionette.Region*s which can be populated with other views.
```javascript
  var OuterView = Backbone.Marionette.Layout.extend({
    template: "#outer-template",

    regions: {
      inner: "#inner-template"
    }
  });

  var outer = new OuterView();
  outer.render();

  var inner = new InnerView();
  outer.inner.show(inner);
```

---

## Application and Modules

- We all know how bad it is to create global vars, so we came up with *namespaces* in JS object structures.
- Common practice: create a plain JS object that everything is attached to, which also initializes the Backbone application.
- *Marionette.Application* does that and much more:
```javascript
  var MyApp = new Marionette.Application();

  MyApp.addInitializer(function(options) {
    new MyRouter(options.initialRoute);
    Backbone.history.start();
  });

  MyApp.start({
    initialRoute: 'home'
  });
```

----

## Marionette's Application

- Can fire events on before, after and during initialization.
- Contain *Marionette.Region*s for global layout organization.
- Can define, access, start and stop *Marionette.Modules*:
```javascript
  MyApp.module('myModule', function() { // Defining
    var privateData = "I'm private"; // Private member

    this.publicFunction = function() { // Public member
      return privateData;
    }

    Foo.addInitializer(function() { // Initializer
      console.log(privateData);
    });
  });

  var myModule = MyApp.module('myModule'); // Accessing
  myModule.start(); // Starting
  myModule.stop(); // Stopping
```

---

## Router vs Controller

- Routers commonly violate the *Single Responsibility Principle* (SRP) when used to:
  - instantiate and manipulate views
  - load models and collections
  - coordinate modules
- A Router's main (and only) purpose is to define routes and delegate the flow to different parts of the application.
- According to MVC, *Controllers* should deal with such things as Models, Collections, Views and Modules.
- Even though Backbone.js is MV*, there is nothing wrong on creating Controllers just as any other Module.

----

## Router vs Controller

- One approach is to delegate the routes to controllers:
```javascript
  var MyRouter = Backbone.Router.extend({
    routes: {
      "": "home",
      "home": "home",
      "product/:id": "viewProduct",
    },

    home: function() {
      AppController.home();
    },

    viewProduct: function(productId) {
      ProductController.viewProduct(productId);
    }
  });
```
- Derick Bailey factors out even those functions: [Reducing Backbone Routers To Nothing More Than Configuration](http://lostechies.com/derickbailey/2012/01/02/reducing-backbone-routers-to-nothing-more-than-configuration/)

---

## Conclusion

- This talk template rocks!
- Your life should be easier now.

---

## Learn more

1. [Structuring jQuery with Backbone.js](http://www.codemag.com/Article/1312061)
1. [Step by step from jQuery to Backbone](https://github.com/kjbekkelund/writings/blob/master/published/understanding-backbone.md)
1. [Zombies! RUN! (Managing Page Transitions In Backbone Apps)](http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/)
1. [Developing Backbone.js Applications](http://addyosmani.github.io/backbone-fundamentals)
1. [Marionette.js](https://github.com/marionettejs/backbone.marionette)
1. [Reducing Backbone Routers To Nothing More Than Configuration](http://lostechies.com/derickbailey/2012/01/02/reducing-backbone-routers-to-nothing-more-than-configuration/)

---

## Challenge

1. Make your awesome talk based on this template.
1. Push it to a gh-pages branch on your GitHub account.
1. Share the URL with the world!