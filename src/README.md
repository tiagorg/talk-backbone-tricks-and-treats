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
 - Memory leaks
   - Manual approach
   - Marionette.js
 - View management
   - Nesting views
 - Bloated code
   - Application and Modules
   - Router vs Controller
 - Tight coupling
   - Pub/sub

----

## Agenda

 - Overwhelming the DOM
   - Fragment
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

## Memory leaks

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
- If you instantiate this view twice, the 1st will never be Garbage Collected, as the model still has a reference for it.
- This can cause *side-effects*: alert box will appear twice.

----

## Manual approach

- To fix it, we just need a method to *unbind* the view:
```javascript
    close: function() {
      // unbind the events that this view is listening to
      this.stopListening();
    }
```
- However, we must remember to manually call this method whenever we destroy a view.
- A good practice is to create a Manager for the current view:
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

## Deploying

1. Make sure your build is not breaking.
  - You should see *Done, without errors.* in the terminal.
1. *git add*, *git commit* and *git push* to *gh-pages* branch.
1. You should see your talk in an address like:
  - http://*your-github-username*.github.io/*your-repo-name*
  - Ex: <http://acbr.github.io/talk-template>

---

## Your talk

 - Should last no more than 50 minutes, leave up to 10 minutes for questions.
 - Should not be too deep neither too superficial.
 - Give at least 3 reference links to be followed for further studies.
 - Give a challenge that would be solved using ideas that were covered on the talk.
 - It is ok to go a little bit far (forcing the attendee to do some research), but that should be optional.

----

## Organization

 - 1st slide: the cover, featuring your talk name, your name, the lecture date and AC logo.
 - 2nd slide: the agenda, in topics.
 - 3nd slide: the prerequisites of your talk.
 - From 4th slide on: your content
  - When content from the same topic doesn't fit on a slide -> grow it *VERTICALLY* by adding a slide below (----).
  - When you finish a topic and will start a different one -> grow it *HORIZONTALLY* by adding a slide to the right (---).
 - The 3 last slides: Conclusion, Learn more (with the reference links) and Challenge.

----

## Content requirements

1. *BE CONSISTENT*. Master the subject and do not contradict yourself.
1. *CATCH THE ATTENTION*. Let the audience know WHY they cannot live one more day without this technology.
1. *BALANCE THEORY AND PRACTICE*. Your target is keeping the subject interesting for everybody.
1. *BE CONCISE*. Don't overexplain in such way you could cause confusion to your attendees.
1. *KEEP THE FOCUS*. Off-topic discussions are ok, but only if it doesn't disturb the natural flow of your content.
1. *BE PREPARED*. If you are going to use examples or live coding, make sure you have them all prepared beforehand.

----

## Communication requirements

1. *COMMUNICATE WELL*. Be sharp on English, no bad words or slangs and use the best words for the audience.
1. *BE A PRO*. Please watch some good screencasts in order to learn how to use your voice and conduct the talk.
1. *BE POLITE*. Be respectful and avoid heavy criticism.
1. *BE PROFESSIONAL*. Use jokes and humor with parsimony.
1. *TRAIN* your full talk at least once before your talk.

----

## The DONT's

1. *DO NEVER SHOW PRIVATE CODE FROM THE CLIENT*. This is CRITICAL and can cause serious problems.
1. *DON'T BE ARROGANT*. Be humble and don't focus the talk on yourself.
1. *DON'T GENERALIZE*, specially stuff that you are not sure about.
1. *DON'T MAKE UP DATA*. Base yourself on trustable references.
1. *DON'T TALK LIKE A ROBOT*. Just be yourself, natural. Relax :)

----

## Tips

 - *ENJOY* your experience by creating the talk, because you will surely learn MUCH MORE than your attendees.
 - *BRING WATER* to drink while you present. You will certainly need it!
 - *BE OPEN* to receive questions and even criticism. You will learn a lot from them.
 - *ALWAYS* be polite when talking to your audience. This will always open doors for you.
 - People might come to you with questions and more complex cases after your talk. Consider it as a gift, it means you represent something good for them!

----

## If you are recording

- Make sure you use a professional microphone when available.
- Don't do *drastic transitions* on your screen, as the recorded amount of frames per second is low.
- Ask atendees to only make questions on the end - so future watchers will just get the real content without interruption.
- Introduce yourself: "Hello everybody, my name is xxxx, I work for Avenue Code and today's talk will be about yyyyy". Finish it like: "That's it, thanks for watching.".
- Problems with recording/connection? Always restart from the beginning of the slide. Don't try to restart from where it fails, its impossible to do a clean cut on the video after that.

---

## Contributing

Should you wish to contribute, please be welcome to!

1. Fork the repository <https://github.com/acbr/talk-template>
1. Create a feature branch for your contribution
```sh
git checkout -b my-new-feature
```
1. Commit your changes
```sh
git commit -am 'Add some feature'
```
1. Push to the branch
```sh
git push origin my-new-feature
```
1. Create a Pull Request

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

---

## Challenge

1. Make your awesome talk based on this template.
1. Push it to a gh-pages branch on your GitHub account.
1. Share the URL with the world!