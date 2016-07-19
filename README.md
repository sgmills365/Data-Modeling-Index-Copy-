## Attribute, Class, and Style Bindings
  The template syntax provides specialized one-way bindings for scenarios less well suited to property binding.

  ### Attribute Binding
  We can set the value of an attribute directly with an **attribute binding**.
.l-sub-section
  :marked
    This is the only exception to the rule that a binding sets a target property. This is the only binding that creates and sets an attribute.

:marked
  We have stressed throughout this chapter that setting an element property with a property binding is always preferred to setting the attribute with a string. So then why does Angular offer attribute binding?

  **We must use attribute binding when there is no element property to bind.**

  Consider the [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA),
  [SVG](https://developer.mozilla.org/en-US/docs/Web/SVG), and
  table span attributes. They are pure attributes.
  They do not correspond to element properties or set element properties since there are no property targets to bind to.

  We become painfully aware of this fact when we try to write something like this:
code-example(language="html").
  &lt;tr>&lt;td colspan="{{1 + 1}}">Three-Four&lt;/td>&lt;/tr>
:marked
  And then we get this error:
code-example(format="nocode").
  Template parse errors:
  Can't bind to 'colspan' since it isn't a known native property
:marked
  As the message says, the `<td>` element does not have a `colspan` property.
  It has the "colspan" *attribute*, but
  interpolation and property binding can set only *properties*, not attributes.

  We need attribute bindings to create and bind to such attributes.

  Attribute binding syntax resembles property binding.
  Instead of an element property between brackets, we start with the prefix **`attr`**,
  followed by a dot (`.`) and the name of the attribute. We then set the attribute
  value, using an expression that resolves to a string.

  Here we bind `[attr.colspan]` to a calculated value:
+makeExample('template-syntax/ts/app/app.component.html', 'attrib-binding-colspan')(format=".")
:marked
  Here's how the table renders:
  <table border="1px">
    <tr><td colspan="2">One-Two</td></tr>
    <tr><td>Five</td><td>Six</td></tr>
   </table>

  One of the primary use cases for attribute binding
  is to set ARIA attributes, as in this example:
+makeExample('template-syntax/ts/app/app.component.html', 'attrib-binding-aria')(format=".")
:marked
  ### Class Binding

  We can add and remove CSS class names from an element’s `class` attribute with
  a **class binding**.

  Class binding syntax resembles property binding.
  Instead of an element property between brackets, we start with the prefix `class`,
  optionally followed by a dot (`.`) and the name of a CSS class: `[class.class-name]`.

  The following examples show how to add and remove the application's "special" class
  with class bindings.  Here's how we set the attribute without binding:
+makeExample('template-syntax/ts/app/app.component.html', 'class-binding-1')(format=".")
:marked
  We can replace that by binding to a string of the desired class names; this is an all-or-nothing, replacement binding.
+makeExample('template-syntax/ts/app/app.component.html', 'class-binding-2')(format=".")

block dart-class-binding-bug
  //- N/A
:marked
  Finally, we can bind to a specific class name.
  Angular adds the class when the template expression evaluates to #{_truthy}.
  It removes the class when the expression is #{_falsey}.
+makeExample('template-syntax/ts/app/app.component.html', 'class-binding-3')(format=".")

.l-sub-section
  :marked
    While this is a fine way to toggle a single class name,
    we generally prefer the [NgClass directive](#ngClass) for managing multiple class names at the same time.

:marked
  ### Style Binding

  We can set inline styles with a **style binding**.

  Style binding syntax resembles property binding.
  Instead of an element property between brackets, we start with the prefix `style`,
  followed by a dot (`.`) and the name of a CSS style property: `[style.style-property]`.

+makeExample('template-syntax/ts/app/app.component.html', 'style-binding-1')(format=".")
:marked
  Some style binding styles have unit extension. Here we conditionally set the font size in  “em” and “%” units .
+makeExample('template-syntax/ts/app/app.component.html', 'style-binding-2')(format=".")

.l-sub-section
  :marked
    This way works best when setting a single style,
    but for several inline styles at the same time, we generally prefer the [NgStyle directive](#ngStyle).

.l-sub-section
  :marked
    Note that a _style property_ name can be written in either
    [dash-case](glossary.html#dash-case), as shown above, or
    [camelCase](glossary.html#camelcase), such as `fontSize`.


















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
