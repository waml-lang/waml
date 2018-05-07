
# WAML

![](https://i.imgur.com/ItWr0HO.png)

Web Automation Markup Language.

**Give WAML a try with the online [WAML editor](https://waml.io/editor).**

> Here you will find the full WAML Specification, examples of what it looks like, and general information regarding the project.

## Overview

The Web Automation Markup Language (WAML) defines a standard, programming language-agnostic specification for a sequence of steps which can be performed on web pages within the context of a web browser.

> **The WAML schema is based on [JSON Schema draft-07](http://json-schema.org/) defined [here](https://github.com/waml-lang/waml/blob/master/schemas/schema.json).**

```yaml
waml: 0.1.0

steps:
  - visit: https://google.com
  - fill:
      selector: input#search
      value: Ugandan knuckles
  - click: a#search
  - wait: 10
  - snap:
      filename: test
      format: png
```

WAML documents are represented in either YAML or JSON formats. These documents may be written manually or generated dynamically from an application (such as a browser extension.) WAML is both human and machine-readable.

## Use Cases

The WAML specification can be used to define flows for:

* Web scraping / data extraction
  * Content monitoring (Online review monitoring, competitor monitoring, price drops, new releases)
  * Content aggregation (Business profile data, leads, jobs, news, events)
  * Turning websites into APIs
* Web automation
  * Integration end-to-end testing
  * Uptime monitoring
  * Evaluating web best practices (SEO, HTML best practices)


# WAML Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Table of contents

* [Terminology](#terminology)
* [Full Example](#full-example)
* [Fields](#fields)
  * [waml](#1-waml)
  * [Info](#2-info)
  * [Variables](#3-variables)
  * [Context](#4-context)
  * [Steps](#5-steps)
* [Variable Store](#variable-store)  
* [Step Library](#step-library)
  * [Visit](#visit)
  * [Click](#click)
  * [Hover](#hover)
  * [Focus](#focus)
  * [Select](#select)
  * [Fill](#fill)
  * [Type](#type)
  * [Scroll](#scroll)
  * [Press](#press)
  * [Wait](#wait)
  * [Refresh](#refresh)
  * [Back](#back)
  * [Forward](#forward)
  * [Scrape](#scrape)
  * [SetContext](#setcontext)
  * [Snap](#snap)
* [Flow Control](#flow-control)

## Terminology

A WAML document represents a single *Flow*: a sequence of *Steps* in a specified browser *Context*. Each Step corresponds to a single user action such as clicking a button, typing into a form, and so on.

A WAML document consists of many *Fields*.

## Full Kitchen Sink Example

Here is an example WAML document which exercises all possible fields and options.

```yaml
waml: 0.1.0

info:
  title: An example WAML document.
  description: High-level summary of the user flow.

variables:
  someValue: Hello world
  isMobile: true

context:
  viewport:
    width: 800
    height: 600
  headers:
    X-API-Key: 123
    Authorization: Bearer abc
  cookies:
    - cookieName: cookieValue
  mediaType: screen
  userAgent: myUserAgent
  device: iPhone4
  timeout: 5

steps:
  - visit: https://google.com
  - click: a.link
  - hover: a#about
  - focus: input#search
  - select:
      selector: select#colors
      value: 'blue'
  - fill:
      selector: input#search
      value: Ugandan knuckles
  - type: ${someValue}
  - press: Shift
  - scroll: 100
  - scroll: a#load-more
  - scroll:
  - wait: div#results
  - wait: 3000
  - wait:
  - refresh:
  - back:
  - forward:
  - scrape:
      articles:
        - select article {0,}
        - body: select .body | get html
          imageUrl: select img | get attr src
          summary: select .body p:first-child | get text
          title: select .title | get text
      checked: select input[type="checkbox"] | get prop checked
  - setContext:
      viewport:
        height: 400
        width: 300
  - snap:
      filename: test
      format: png
  - pdf: test
```

## Fields

The table below lists the 5 root-level fields of a WAML document.

Field Name | Type | Description
---|:---:|---
[waml](#1-waml) | `string` | **REQUIRED**. The string must be the semantic version number of the WAML Specification that the WAML document uses.
[info](#2-info) | `InfoObject` | Provides metadata about the flow.
[variables](#3-variables) | `object` | Stores variables used in the `steps` field.
[context](#4-context) | `ContextObject` | Defines the browser's context and emulation configuration.
[steps](#5-steps) | `[]StepObject` | **REQUIRED**. A sequence of steps for the flow.

All field names in the specification are **case sensitive**.

### 1. waml

The WAML Specification is versioned using [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html) (semver) and follows the semver specification.

A WAML document must begin with a version declaration within the top-level `waml` field. The `waml` field must contain the document's WAML Specification's semantic version number.

The `waml` field is **required**.

```yaml
waml: 0.1.0
```

### 2. Info

The Info field provides general information about the Flow.

The `info` field is optional.

```yaml
info: # Metadata about the WAML document
  title: An example WAML document.
  description: High-level summary of the user flow.
```

##### Params

Field Name | Type | Description
---|:---:|---
title | `string` | The title of the flow.
description | `string` | A short summary of the flow.

### 3. Variables

The Variables field stores variables usable elsewhere in the Flow, such as in the `steps` field.
You can store any key-value pairs. Values must be a literal value (`string`, `number`, or `boolean`).

The use of variables and the `variables` field is optional.

```yaml
variables: # A mapping of variables to be defined in the flow context
  searchTerm: Hello world
  isMobile: true
```

> You can reference any pre-defined variables with the `${variable}` syntax.

```yaml
steps:
  - fill:
      selector: input#search
      value: ${searchTerm}
```

> Referencing a variable in `steps` that is not defined beforehand in the `variables` field SHOULD cause a WAML implementation to raise a validation error.

### 4. Context

The Context field defines the browser's context and emulation configuration.

The `context` field is optional.

```yaml
context: # Browser emulation settings (optional)
  viewport: # Browser window dimensions
    width: 800
    height: 600
  headers: # Additional HTTP headers to be sent with every request
    X-API-Key: 123
    Authorization: Bearer abc
  cookies: # Browser cookies
    - cookieName: cookieValue
  mediaType: screen # screen / print / none
  userAgent: myUserAgent # Browser user agent
  timeout: 5 # Maximum navigation time
  device: 'iPhone4'
```

##### Params

Field Name | Type | Description
---|:---:|---
viewport | `ViewportObject`  | The `width` and `height` of the viewport.
headers | `object` | A set of key-value pairs containing additional HTTP headers to be sent with every request.
cookies | `[]CookieObject` | A set of key-value pairs containing cookies of the browser context.
mediaType | `string` | The browser's media type. Values can be "screen" or "print".
userAgent | `string` | The browser's user agent string.
timeout | `integer` | The maximum navigation timeout in milliseconds.
device | `string` | Device descriptor to emulate. List [here](https://github.com/GoogleChrome/puppeteer/blob/master/DeviceDescriptors.js).

### 5. Steps

The Steps field contains a sequence of one or more steps.
Each step is a StepObject that represents a particular user action, such as clicking an HTML element.

The `steps` field is **required**.

```yaml
steps: # A sequence of user interactions
  - visit: https://google.com # Navigating to a URL
  - fill: # Focus to an element and type things in
      selector: input#search
      value: Ugandan knuckles
  - click: a#search # Click an HTML element
  - wait # Wait for a page load to finish
  - snap: # Take screenshot of the page
      filename: foo
      format: png
```

#### Note: Compact and Expanded versions

Steps can have a compact version which omits default and optional parameters, and an expanded version with additional parameters & features. For example, the `click` step normally just needs a CSS `selector`:

> Compact version

```yaml
- click: a.link # Click an HTML element
```

But you can also use the expanded version of `click` to specify a target from a list of elements:

> Expanded version

```yaml
- click: # Expanded version of click
    selector: a.link
    index: 2 # click 3rd element from a list
```

## Variable Store

The variable store is an ephemeral, mutable key-value store for reading and storing values to be used elsewhere in the Flow. At the start of a Flow, the variable store can be initialized through the `variables` field:

```yaml
variables:
  someValue: Hello world
  isMobile: true
```

> Variables can be read using the `${}` syntax:

```yaml
steps:
  - fill:
      selector: input#search
      value: ${someValue}
```

> Steps such as `scrape` saves its results into the variable store.

```yaml
- scrape:
    articles:
      - select article {0,}
      - body: select .body | get html
        imageUrl: select img | get attr src
        summary: select .body p:first-child | get text
        title: select .title | get text
    checked: select input[type="checkbox"] | get prop checked
```

At the end of a WAML flow, the contents of the variable store should be returned and the variable store cleared.

## Step Library

The table below lists the step types available in WAML:

Step Name | Description
----------|-------------
[visit](#visit)  | Navigate to a specified URL.
[click](#click)  | Click an element.
[hover](#hover)  | Hover over an element.
[focus](#focus)  | Focus to an element.
[select](#select)  | Focus and select one or more values from a dropdown element.
[fill](#fill)  | Focus an type a string into an input element.
[type](#type)  | Type a string of characters.
[press](#press)  | Type a special key, such as `Shift` and `Enter`.
[wait](#wait)  | Wait for a specified interval, for an element to load, or for a page load to finish.
[refresh](#refresh)  | Refresh the current page.
[back](#back)  | Move backward in the browser history.
[forward](#forward)  | Move forward in the browser history.
[scrape](#scrape)  | Scrape the HTML document for specified elements and attributes.
[assert](#assert)  | Evaluates a runtime assertion. Aborts the flow if `false`.
[setContext](#setcontext) | Modify browser context / emulation settings.
[snap](#snap)  | Take a screenshot of the page.

> **`Bolded`** properties are **required** properties.

### Visit

The `visit` step navigates to a specified URL.

```yaml
- visit: https://google.com
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`url`**    | The URL to navigate. Must be prefixed with either `http://` or `https://`. | `string` | `undefined`

### Click

The `click` step clicks a specified HTML element by [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).
If there are multiple elements satisfying the selector, the first will be clicked.

```yaml
- click: a.link # Clicks first matching element
```

```yaml
- click:
    selector: ytd-thumbnail.ytd-video-renderer
    index: 2 # Clicks 3rd video from a list of elements
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`selector`**    | The CSS selector of an HTML element. | `string` | `undefined`
`index`    | Index of element, if there are multiple matching elements. | `integer` | `undefined`

### Hover

The `hover` step hovers the mouse pointer over a specified HTML element.
If there are multiple elements satisfying the selector, the first will be hovered.

```yaml
- hover: a#sign-in
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`selector`**    | The CSS selector of an HTML element. | `string` | `undefined`

### Focus

The `focus` step focuses over a specified HTML element.
If there are multiple elements satisfying the selector, the first will be focused.

```yaml
- focus: input#search
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`selector`**    | The CSS selector of an HTML element. | `string` | `undefined`

### Select

The `select` step focuses over a specified dropdown element and selects one or more values.

If the `<select>` HTML element has the `multiple` attribute, all values are considered, otherwise only the first one is taken into account.

```yaml
- select:
    selector: select#colors
    value: 'blue'
```

```yaml
- select: # Focusing and selecting one or more values from a dropdown
    selector: select#colors
    value:
      - 'blue'
      - 'red'
      - 'green'
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`selector`**    | The CSS selector of an HTML element. | `string` | `undefined`
**`value`**    | One or more values. | `array` | `undefined`

### Fill

The `fill` step types a string of text into a focused HTML element, usually form inputs.

```yaml
- fill:
    selector: input#search
    value: Ugandan knuckles
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`selector`**    | The CSS selector of an HTML element. | `string` | `undefined`
**`value`**    | String of text to type in. | `string` | `undefined`

### Type

The `types` step types a string of text.

```yaml
- type: Hello world 123 嗨
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`value`**    | String of text to type in. | `string` | `undefined`

### Press

The `press` step types special keys such as `Shift` and `Enter`.

```yaml
- press: ArrowLeft
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`key`**    | Name of key to press, such as `ArrowLeft`. See [USKeyboardLayout](https://github.com/GoogleChrome/puppeteer/blob/v1.0.0/lib/USKeyboardLayout.js) for a list of all key names. | `string` | `undefined`

### Scroll

The `scroll` step scrolls the browser's viewport by a specified number of pixels, a specified HTML element, or to the bottom of the page (for infinite scrolls.)

```yaml
- scroll: 100 # Scroll 100 pixels
- scroll: a#load-more # Scroll to HTML element
- scroll: # Infinite scroll
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`value`**    | Integer, CSS selector, or 'infinite'-ly. | `string` or `integer` | `undefined`

### Wait

The `wait` step lets you wait by a specified interval, for an HMTL element to load, or for a page navigation to finish.

```yaml
- wait: div#results # Selector of an element to wait for
- wait: 3000 # Time to wait
- wait: # Wait for a page load to finish
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`selector`    | The CSS selector of an HTML element. | `string` | `undefined`
`timeout`    | Number of milliseconds to wait for. | `integer` | 0

### Refresh

The `refresh` step refreshes the current page.

```yaml
- refresh:
```

### Back

The `back` step moves the current page backward in browser navigation history.

```yaml
- back:
```

### Forward

The `forward` step moves the current page forward in browser navigation history.

```yaml
- forward:
```

### Scrape

The `scrape` step lets extract and evaluate attributes and HTML elements of the current page.

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`DOM extraction expression`**    | An object containing a [declarative DOM extraction expression](https://github.com/gajus/surgeon). | `object` | `undefined`

> The `scrape` step accepts as input a declarative DOM extraction expression based on [`surgeon`](https://github.com/gajus/surgeon).

You describe the content you wish to scrape using `select` and `get` keywords.

#### `select`

```
select '.post-title a' {0,}
```

Property | Description | Type | Default | Example
---------|:-----------:|:----:|-------------- :|--------------
**`CSS selector`**    | A CSS selector. | `string` | `undefined` | `div.my-class a`
`quantifier`    | Slicing & indexing quantifier to pick amongst matches. | `string` | `[0]` | `{0,4}[2]`

The `select` keyword accepts as input a CSS selector (e.g. `.body p:first-child`).

By default, the first matching element is selected. To select multiple or the N-th element, supply a third argument. This 'quantifier' argument is used to select single or multiple CSS selector matches, and to pick specific indexes from the list. For example, `select div {0:2}[1]` selects the first two matches (`{0:2}`), and picks the second (`[1]`) - returning a single element. Read [this](https://github.com/gajus/surgeon#quantifier-expression) for more details on the quantifier expression.

#### `get`

```
get attr src
get prop innerHTML
get html
get text
```

Property | Description | Type | Default | Example
---------|:-----------:|:----:|-------------- :|--------------
**`content type`**    | Content to extract. | `string` | `undefined` | `text`, `html`, `prop`, `attr`
`content name`    | Name of property or attribute to extract. | `string` | `undefined` | `src`

The `get` keyword accepts as input one of `text`, `html`, `prop`, and `attr` for extracting text, raw HTML content, DOM properties, and HTML attributes respectively.

The `get` keyword accepts a name argument in the case of `prop` and `attr`, which specifies the name of the property (e.g. `checked`) or attribute (e.g. `src`) to get.

> Below is an example of both `select` and `get` keywords used together.

```yaml
- scrape:
    articles:
      - select article {0,}
      - body: select .body | get html
        imageUrl: select img | get attr src
        summary: select .body p:first-child | get text
        title: select .title | get text
    checked: select input[type="checkbox"] | get prop checked
```

> The above WAML will produce the following structured data:

```js
{
  articles: [
    {
      body: ...
      imageUrl: ...
      summary: ...
      title: ...
    },
    {
      body: ...
      imageUrl: ...
      summary: ...
      title: ...
    }
  ],
  checked: ...
}
```

#### Nested Data

Subroutines can be nested: HTML elements from a previous level is passed as arguments to the next level. Take a look at the following example:

```yaml
- select article
- body: select .body | get html
  imageUrl: select img | get attr src
  summary: select .body p:first-child | get text
  title: select .title | get text
```

Let's walk through these steps:

- The `select` line returns a list of one or more HTML elements with the selector `article`. This is equivalent to `document.querySelectAll('article')`.
- In the next line, we name and extract `body`, `imageUrl`, `summary`, and `title` with the `select`ion in the first line as input.

> In this example, `body` is equivalent to:

```js
document.querySelector('article').querySelector('.body').innerHTML
```

> `imageUrl` is equivalent to:

```js
document.querySelector('article').querySelector('img').getAttribute('src')
```


#### The Pipe Operator

You can use the `|` pipe operator to simplify a list of expressions:

```yaml
# Before
- body:
  - select .body
  - get html
```

> The two examples here are equivalent.

Into the following one-liner:

```yaml
# After
- body: select .body | get html
```

### Assert

The `Assert` Step allows you to perform runtime assertions with Javascript expressions. The results of the assertions are saved into the variable store.

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`expression`**    | Javascript expression. | `string` | `undefined`
**`message`**    | Description of the assertion. | `string` | `undefined`

```yaml
- assert:
    expression: '${pageTitle}' === 'My Page Title'
    message: Valid page title
```

### SetContext

The `SetContext` step updates the browser context / emulation settings.

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`context`**    | Context object containing headers, cookies, etc. | `Context Object` | `undefined`

```yaml
- setContext:
    viewport:
      width: 800
      height: 600
    headers:
      X-API-Key: 123
      Authorization: Bearer abc
    cookies:
      - cookieName: cookieValue
    mediaType: screen
    userAgent: myUserAgent
    device: 'iPhone4'
    timeout: 5
```

### Snap

The `snap` step takes a screenshot of the current page and saves it into the local filesystem or a cloud storage.

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`filename`**    | The filename to save the image as. | `string` | `undefined`
`format`    | The format to save the image as. PNG and JPG formats supported. | `string` | `png`
`fullPage`    | If set to true, takes a screenshot of the full scrollable page. | `boolean` | `false`

```yaml
- snap: # Take screenshot of the page
    filename: my-image
    format: png
    fullPage: true
```

### PDF

The `pdf` step saves the current page as a PDF into the local filesystem, relative to current working directory.

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
**`filename`**    | The filename to save the PDF as. | `string` | `undefined`

```yaml
- pdf: my-pdf
```

## Flow Control

WAML provides the following flow control abstractions:

### if & unless

The `if` and `unless` optional attributes can be specified in a Step to perform conditional execution.

> Pass in a Javascript expression that returns a `boolean`.

```yaml
- click:
    selector: a#search
  if: ${someValue} === "submit"
```

> The `click` Step above will be executed only `if` the expression `${someValue} === "submit"` evaluates to `true`.

You can reference variables from the Variable Store within your Expression.

#### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`if`    | If set, the step is only executed if the value evaluates to `true`. | `Expression` | `undefined`
`unless`    | If set, the step is only executed if the value evaluates to `false`. | `Expression` | `undefined`

## Get Involved

WAML is an evolving spec. [Documentation](https://github.com/waml-lang/waml/wiki), [bug reports](https://github.com/waml-lang/waml/issues), [pull requests](https://github.com/waml-lang/waml/pulls), and all other contributions are welcome!

## Thanks

**waml** © 2018+, Yos Riady. Released under the [MIT] License.<br>
Authored and maintained by Yos Riady with help from contributors ([list][contributors]).

> [yos.io](http://yos.io) &nbsp;&middot;&nbsp;
> GitHub [@yosriady](https://github.com/yosriady)

[MIT]: http://mit-license.org/
[contributors]: http://github.com/waml-lang/waml/contributors
