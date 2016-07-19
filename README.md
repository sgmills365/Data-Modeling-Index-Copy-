.l-main-section
:marked
  <a id="binding-syntax"></a>
  ## Binding Syntax: An Overview
  Data binding is a mechanism for coordinating what users see with application data values.
  While we could push values to and pull values from HTML,
  the application is easier to write, read, and maintain if we turn these chores over to a binding framework.
  We simply declare bindings between binding sources and target HTML elements and let the framework do the work.

  Angular provides many kinds of data binding, and we’ll discuss each of them in this chapter.
  But first we'll take a high-level view of Angular data binding and its syntax.

  We can group all bindings into three categories by the direction in which data flows.
  Each category has its distinctive syntax:
table
  tr
    th Data Direction
    th Syntax
    th Binding Type
  tr
    td One-way<br>from data source<br>to view target
    td
      code-example().
        {{expression}}
        [target] = "expression"
        bind-target = "expression"
    td.
      Interpolation<br>
      Property<br>
      Attribute<br>
      Class<br>
      Style
    tr
      td One-way<br>from view target<br>to data source
      td
        code-example().
          (target) = "statement"
          on-target = "statement"
      td Event
    tr
      td Two-way
      td
        code-example().
          [(target)] = "expression"
          bindon-target = "expression"
      td Two-way

:marked
  Binding types other than interpolation have a **target name** to the left of the equal sign,
  either surrounded by punctuation (`[]`, `()`) or preceded by a prefix (`bind-`, `on-`, `bindon-`).

  What is that target? Before we can answer that question, we must challenge ourselves to look at template HTML in a new way.

  ### A New Mental Model

  With all the power of data binding and our ability to extend the HTML vocabulary
  with custom markup, it is tempting to think of template HTML as *HTML Plus*.

  Well, it *is* HTML Plus.
  But at the same time, it’s also significantly different than the HTML we’re used to.
  We really need a new mental model.

  In the normal course of HTML development, we create a visual structure with HTML elements, and
  we modify those elements by setting element attributes with string constants.

+makeExample('template-syntax/ts/app/app.component.html', 'img+button')(format=".")
:marked
  We still create a structure and initialize attribute values this way in Angular templates.

  Then we learn to create new elements with components that encapsulate HTML
  and drop them into our templates as if they were native HTML elements.
+makeExample('template-syntax/ts/app/app.component.html', 'hero-detail-1')(format=".")
:marked
  That’s HTML Plus.

  Now we start to learn about data binding. The first binding we meet might look like this:

+makeExample('template-syntax/ts/app/app.component.html', 'disabled-button-1')(format=".")
:marked
  We’ll get to that peculiar bracket notation in a moment. Looking beyond it,
  our intuition tells us that we’re binding to the button's `disabled` attribute and setting
  it to the current value of the component’s `isUnchanged` property.

  Our intuition is wrong! Our everyday HTML mental model is misleading us.
  In fact, once we start data binding, we are no longer working with HTML *attributes*. We aren't setting attributes.
  We are setting the *properties* of DOM elements, components, and directives.

.l-sub-section
  :marked
    ### HTML Attribute vs. DOM Property

    The distinction between an HTML attribute and a DOM property is crucial to understanding how Angular binding works.

    **Attributes are defined by HTML. Properties are defined by the DOM (Document Object Model).**

    * A few HTML attributes have 1:1 mapping to properties. `id` is one example.

    * Some HTML attributes don't have corresponding properties. `colspan` is one example.

    * Some DOM properties don't have corresponding attributes. `textContent` is one example.

    * Many HTML attributes appear to map to properties ... but not in the way we might think!

    That last category can be especially confusing ... until we understand this general rule:

    **Attributes *initialize* DOM properties and then they are done.
    Property values can change; attribute values can't.**

    For example, when the browser renders `<input type="text" value="Bob">`, it creates a
    corresponding DOM node with a `value` property *initialized* to "Bob".

    When the user enters "Sally" into the input box, the DOM element `value` *property* becomes "Sally".
    But the HTML `value` *attribute* remains unchanged as we discover if we ask the input element
    about that attribute: `input.getAttribute('value') // returns "Bob"`

    The HTML attribute `value` specifies the *initial* value; the DOM `value` property is the *current* value.

    The `disabled` attribute is another peculiar example. A button's `disabled` *property* is
    `false` by default so the button is enabled.
    When we add the `disabled` *attribute*, its presence alone initializes the  button's `disabled` *property* to `true`
    so the button is disabled.

    Adding and removing the `disabled` *attribute* disables and enables the button. The value of the *attribute* is irrelevant,
    which is why we cannot enable a button by writing `<button disabled="false">Still Disabled</button>`.

    Setting the button's `disabled` *property*  (say, with an Angular binding) disables or enables the button.
    The value of the *property* matters.

    **The HTML attribute and the DOM property are not the same thing, even when they have the same name.**

:marked
  This is so important, we’ll say it again.
  ### Binding Targets
  The **target of a data binding** is something in the DOM.
  Depending on the binding type, the target can be an
  (element | component | directive) property, an
  (element | component | directive) event, or (rarely) an attribute name.
  The following table summarizes:

// If you update this table, UPDATE it in Dart & JS, too.
<div width="90%">
table
  tr
    th Binding Type
    th Target
    th Examples
  tr
    td Property
    td.
      Element&nbsp;property<br>
      Component&nbsp;property<br>
      Directive&nbsp;property
    td
      +makeExample('template-syntax/ts/app/app.component.html', 'property-binding-syntax-1')(format=".")
  tr
    td Event
    td.
      Element&nbsp;event<br>
      Component&nbsp;event<br>
      Directive&nbsp;event
    td
      +makeExample('template-syntax/ts/app/app.component.html', 'event-binding-syntax-1')(format=".")
  tr
    td Two-way
    td.
      Event and property
    td
      +makeExample('template-syntax/ts/app/app.component.html', '2-way-binding-syntax-1')(format=".")
  tr
    td Attribute
    td.
      Attribute
      (the&nbsp;exception)
    td
      +makeExample('template-syntax/ts/app/app.component.html', 'attribute-binding-syntax-1')(format=".")
  tr
    td Class
    td.
      <code>class</code> property
    td
      +makeExample('template-syntax/ts/app/app.component.html', 'class-binding-syntax-1')(format=".")
  tr
    td Style
    td.
      <code>style</code> property
    td
      +makeExample('template-syntax/ts/app/app.component.html', 'style-binding-syntax-1')(format=".")
</div>

:marked
  Let’s descend from the architectural clouds and look at each of these binding types in concrete detail.


  **Template binding works with *properties* and *events*, not *attributes*.**

.callout.is-helpful
  header A world without attributes
  :marked
    In the world of Angular 2, the only role of attributes is to initialize element and directive state.
    When we data bind, we're dealing exclusively with element and directive properties and events.
    Attributes effectively disappear.
:marked
  With this model firmly in mind, let's learn about binding targets.
