
# WAML

![](https://i.imgur.com/KIUsDDn.png?1)

Web Automation Markup Language.

> Here you will find the full WAML Specification, simple examples of what it looks like, and some general information regarding the project.

## Overview

The Web Automation Markup Language (WAML) defines a standard, programming language-agnostic specification for a sequence of steps which can be performed on web pages within the context of a web browser.

```yaml
waml: 0.1.0

steps:
  - visit: https://google.com
  - fill:
      selector: input#search
      value: Ugandan knuckles
  - click: a#search
  - wait
  - screenshot
```

WAML documents are represented in either YAML or JSON formats. These documents may be written manually or generated dynamically from an application (such as a browser extension.) WAML is both human and machine-readable.

> The WAML schema is based on [JSON Schema](http://json-schema.org/) defined [here](https://github.com/waml-lang/waml/blob/master/schemas/schema.json).

## Use Cases

The WAML specification can be used to define flows for:

* Web scraping / data extraction
  * Track changes on a web page (price drops, new releases)
  * Turning websites into APIs
  * Content aggregation
  * Online review monitoring
  * Lead generation
* Web automation
  * Integration end-to-end testing
  * Uptime monitoring
  * Evaluating web best practices


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
* [Step Library](#step-library)
  * [Visit](#visit)
  * [Click](#click)
  * [Hover](#hover)
  * [Focus](#focus)
  * [Select](#select)
  * [Fill](#fill)
  * [Type](#type)
  * [Press](#press)
  * [Wait](#wait)
  * [Refresh](#refresh)
  * [Back](#back)
  * [Forward](#forward)
  * [Scrape](#scrape)
  * [Screenshot](#screenshot)

## Terminology

A WAML document represents a single *Flow*: a sequence of *Steps* in a specified browser *Context*. Each Step corresponds to a single user action such as clicking a button, typing into a form, and so on.

A WAML document consists of many *Fields*.

## Full Example

Here is an example WAML document which exercises all possible fields and options.

```yaml
waml: 0.1.0

info:
  name: An example WAML document.
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
  device: 'iPhone4'
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
  - wait: div#results
  - wait: 3000
  - wait
  - refresh
  - back
  - forward
  - scrape:
      selector: input#search
      attr: value
      as: myKey
  - screenshot: test.png
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
  name: An example WAML document.
  description: High-level summary of the user flow.
```

##### Params

Field Name | Type | Description
---|:---:|---
name | `string` | The name of the flow.
description | `string` | A short summary of the flow.

### 3. Variables

The Variables field stores variables used in the `steps` field.
You can store any key-value pairs. Values must be a literal value (`string`, `number`, or `boolean`) and must not be nested.

The use of variables and the `variables` field is optional.

```yaml
variables: # A mapping of variables to be defined in the flow context
  searchTerm: Hello world
  isMobile: true
```

You can reference any pre-defined variables with the `${variable}` syntax.

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
  - screenshot # Take screenshot of the page
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
[scrape](#scrape)  | Scrape the HTML document for specified elements and/or attributes.
[screenshot](#screenshot)  | Take a screenshot of the page.

> Properties marked with a `*` suffix is **required**.

### Visit

The `visit` step navigates to a specified URL.

```yaml
- visit: https://google.com
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`url`*    | The URL to navigate. Must be prefixed with `http://` or `https://` protocols. | `string` | `undefined`

### Click

The `click` step clicks to a specified HTML element by [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).
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
`selector`*    | The CSS selector of an HTML element. | `string` | `undefined`
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
`selector`*    | The CSS selector of an HTML element. | `string` | `undefined`

### Focus

The `focus` step focuses over a specified HTML element.
If there are multiple elements satisfying the selector, the first will be focused.

```yaml
- focus: input#search
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`selector`*    | The CSS selector of an HTML element. | `string` | `undefined`

### Select

The `select` step focuses over a specified dropdown element and selects one or more values.

If the `<select>` has the `multiple` attribute, all values are considered, otherwise only the first one is taken into account.

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
`selector`*    | The CSS selector of an HTML element. | `string` | `undefined`
`value`*    | One or more values. | `array` | `undefined`

### Fill

The `fill` step types a string of text into a focused HTML element.

```yaml
- fill:
    selector: input#search
    value: Ugandan knuckles
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`selector`*    | The CSS selector of an HTML element. | `string` | `undefined`
`value`*    | String of text to type in. | `string` | `undefined`

### Type

The `types` step types a string of text.

```yaml
- type: Hello world 123 嗨
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`value`*    | String of text to type in. | `string` | `undefined`

### Press

The `press` step types special characters.

```yaml
- press: ArrowLeft
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`key`*    | Name of key to press, such as `ArrowLeft`. See [USKeyboardLayout](https://github.com/GoogleChrome/puppeteer/blob/v1.0.0/lib/USKeyboardLayout.js) for a list of all key names. | `string` | `undefined`

### Wait

The `wait` step lets you wait by a specified interval, for an HMTL element to load, or for a page navigation to finish.

```yaml
- wait: div#results # Selector of an element to wait for
- wait: 3000 # Time to wait
- wait # Wait for a page load to finish
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`selector`    | The CSS selector of an HTML element. | `string` | `undefined`
`timeout`    | Number of milliseconds to wait for. | `integer` | 0

### Refresh

The `refresh` step refreshes the current page.

```yaml
- refresh
```

### Back

The `back` step moves the current page back in the browser navigation history.

```yaml
- back
```

### Forward

The `forward` step moves the current page forward in the browser navigation history.

```yaml
- forward
```

### Scrape

The `scrape` step lets extract and evaluate attributes and HTML elements of the current page.

```yaml
- scrape: # Scrape data from the HTML document
    selector: input#search # can be single or multiple
    attr: value # Element attribute to scrape
    as: abc # Key to write to in ephemeral data store
```

##### Params

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`selector`*    | The CSS selector of an HTML element. | `string` | `undefined`
`attr`    | Element attribute to extractm e.g. `value`, `href`. Defaults to `outerHTML`| `string` | `outerHTML`
`as`    | Key in an ephemeral storage to save contents to. | `string` | `undefined`

### Screenshot

The `screenshot` step takes a screenshot of the current page and saves it into the local filesystem, relative to current working directory.

Property | Description | Type | Default
---------|:-----------:|:----:|--------------
`name`*    | The filename to save the screenshot as. | `string` | `undefined`

```yaml
- screenshot: test.png
```

## Get Involved

Documentation, bug reports, pull requests, and all other contributions are welcome!

## Thanks

**waml** © 2018+, Yos Riady. Released under the [MIT] License.<br>
Authored and maintained by Yos Riady with help from contributors ([list][contributors]).

> [yos.io](http://yos.io) &nbsp;&middot;&nbsp;
> GitHub [@yosriady](https://github.com/yosriady)

[MIT]: http://mit-license.org/
[contributors]: http://github.com/waml-lang/waml/contributors
