# ember-islands

[ ![Codeship Status for mitchlloyd/ember-islands](https://codeship.com/projects/0de87f00-c59f-0132-5306-3a52b81c571d/status?branch=master)](https://codeship.com/projects/74441)

Render Ember components UJS-style to achieve "Islands of Richness". You can
arbitrarily render Ember components in the body of the page and they will all be
connected to the same Ember app.

## Installation

This addon should be installed inside of an ember-cli project.

```
ember install:addon ember-islands
```

## Usage

```html
<div data-component='my-component' data-attrs='{"title": "Component Title"}'>
  <p>Some optional innerContent</p>
</div>
```

```handlebars
{{! inside of app/templates/components/my-component}}

{{title}}

{{{innerContent}}}
```

Ember Islands parses the JSON data that you pass in the `data-attrs` attribute
and sends it to the component as attributes. Any content within your tag
(`innerHTML`) is passed as the `innerContent` attribute.

## Example Usage in Rails

If you're using this addon within the context of a Rails app you'll probably
want to get familiar with the
[ember-cli-rails](https://github.com/rwz/ember-cli-rails) gem. This will help
you integrate ember-cli with your Rails application.

Make sure that you've included your ember-cli application on a given page either
using the helper from ember-cli-rails or by including it yourself.

Then inside of your Rails view files you can render a component using HTML data
attributes.

```html+erb
<div data-component="my-component" data-attrs="<%= {
  title: @post.title,
  body: @post.body
}.to_json %>"></div>
```

Another option is to create a Rails helper.

```ruby
module EmberComponentHelper
  def ember_component(name, attrs = {})
    content_tag(:div, '', data: { component: name, attrs: attrs.to_json })
  end
end
```

And then use that inside of your view files instead.

```html+erb
<%= ember_component 'my-component', title: @post.title, body: @post.body %>
```

## Bypassing the Addon

If you only want to use ember-islands in certain environments, you can suppress
its behavior using `APP` configuration:

```javascript
// Example app/config/environment.js file from ember-cli.

module.exports = function(environment) {
  var ENV = {
    // ...

    APP: {
      // This is a flag to supress ember-islands behavior
      EMBER_ISLANDS: { bypass: true }
    },

    if (environment === 'legacy') {
      ENV.APP.EMBER_ISLANDS.bypass = false;
    }
  };
```

## Why?

Ember Islands is a reference to the "Islands of Richness" pattern where the
majority of a web page is rendered by the server but certain portions of the
page (e.g. "widgets") are dynamic and need to use JavaScript.

Often when you start building an app you'll say to yourself "This is just a
simple Blog/Auction Site/E-Commerce app. I would never want to use a JavaScript
framework for this." One week later you're trying to figure out which
incantation of micro-libraries, design patterns, and build tooling might help you
refactor your jQuery soup.

This addon is for those times when you can't convince yourself or someone else
to create an Ember app from the start or when you're trying to introduce Ember
into an existing server-rendered application. You can start with components and
later back into features like the Router if you'd like.

## How?

This addon uses an Ember.Initializer to:

1. Locate elements on the page with `data-component="some-component-name"`.
2. Create components with the name given by `data-component` and with the
   attributes given by the json in `data-attrs`.
3. Append each component to the DOM using `appendTo`.
4. Cancel the default routing behavior which would normally load the Application
   route and render the application template when the app boots.

These components are created in the context of your Ember application so they
all share the helpers, services, models, and other objects in your application.

When any Ember application is initialized it sets up event listeners on the
document `body` that delegate events from the `body` to your Ember components.
When you render components inside of the body tag they will be fully functioning
components.

By default Ember uses the `body` tag as its root element. It is possible to
scope your Ember application to just a portion of the page by configuring the
`rootElement` property of the application. If you do set a specific
`rootElement` keep in mind that, just as with any Ember application, your
components must be rendered inside of the root element.

