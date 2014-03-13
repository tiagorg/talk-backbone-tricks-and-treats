<!--

WARNING!! DON'T EDIT THE FILE README.md on the root of the project, that one is a GENERATED FILE!

You should just edit the source file at src/README.md - the one which stars with ## @@title

-->

## @@title

<img src="img/cover.jpg" class="logo" />

@@author @ [Avenue Code](http://www.avenuecode.com)

*@@email*

@@date

---

## Agenda

 - The jQuery Way
 - Views and Memory leaks
 - Overwhelming the DOM
 - View management
   - Nesting views
 - Bloated code
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

- Backbone depends on jQuery, however it must not be abused.
- Developers coming from strong jQuery background insist on *jQuerizing*, while Backbone provides structure to avoid that:
  - AJAX belongs to the Model and *SHOULD NOT* be coded like *$.ajax()*.
  - DOM events binding belongs to the View and *SHOULD NOT* be coded like *$(el).on('click', ...)*.
- This is a common scenario in code migrations to Backbone, but simple to fix. Just have Model and View do their work.
- Follow [Step by step from jQuery to Backbone](https://github.com/kjbekkelund/writings/blob/master/published/understanding-backbone.md) to better understand this process.

---

## Views and Memory leaks

- Backbone leaves much of the code structure for to the developer to define and implement.
- Bad designs easily lead to memory leaks.
```javascript
  var MyView = Backbone.View.extend({
    initialize: function() {
      this.model.on('change', this.render, this);
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
      // unbind the events that this view is listening to
      this.stopListening();
    }
```
- However, we must remember to manually call this method whenever we destroy a View.
- A good practice is a Manager to keep the current View:
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

## Marionette.ItemView

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

## Marionette.Region

- *Marionette.Region* is a Views container and manager.
- It manages their lifecycles and proper display on a DOM element and closing (no more Zombie Views).
```javascript
    var myRegion = new Marionette.Region({
      el: '#content'
    });

    var view1 = new MyView({ /* ... */ });
    myRegion.show(view1);

    var view2 = new MyView({ /* ... */ });
    myRegion.show(view2);
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
          this.$el.append(view.render().el);
        }, this);
        return this;
      }
    });
```
- If the Collection has N items, this code makes N operations on the DOM, which is *expensive*. Imagine N = 1000.


----

## Manual approach

- A better approach is to append to a *document fragment* instead, and just add the fragment once to the DOM:
```javascript
    var CollectionView = Backbone.View.extend({
      render: function() {
        var fragment = document.createDocumentFragment();
        _.each(this.collection.models, function(item) {
          var view = new MyView({
            model: item
          });
          fragment.appendChild(view.render().el);
        }, this);

        this.$el.html(fragment);
        return this;
      }
    });
```

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

---

## Challenge

1. Make your awesome talk based on this template.
1. Push it to a gh-pages branch on your GitHub account.
1. Share the URL with the world!