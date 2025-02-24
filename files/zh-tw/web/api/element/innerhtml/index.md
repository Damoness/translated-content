---
title: Element.innerHTML
slug: Web/API/Element/innerHTML
tags:
  - 元素組件
  - 注譯
  - 裏HTML
  - 裏超文本
translation_of: Web/API/Element/innerHTML
---
{{APIRef("DOM")}}

[Element](/zh-TW/docs/Glossary/Element) 的「innerHTML」屬性獲取或設置元素中包含的 HTML 或 XML 標記。

> **備註：** 如{{HTMLElement("div")}}, {{HTMLElement("span")}}, or {{HTMLElement("noembed")}} 節點有包含字符（＆），（<）或（>），innerHTML 分別地回傳這些字符成為 HTML 的 `“&”`，`“<”` 和 `“>”`。 使用 `Node.textContent` 得到的這些文本節點的內容的原始拷貝件。

To insert the HTML into the document rather than replace the contents of an element, use the method {{domxref("Element.insertAdjacentHTML", "insertAdjacentHTML()")}}.

## Syntax

```js
const content = element.innerHTML;

element.innerHTML = htmlString;
```

### Value

A {{domxref("DOMString")}} containing the HTML serialization of the element's descendants. Setting the value of `innerHTML` removes all of the element's descendants and replaces them with nodes constructed by parsing the HTML given in the string _htmlString_.

### Exceptions

- `SyntaxError`
  - : An attempt was made to set the value of `innerHTML` using a string which is not properly-formed HTML.
- `NoModificationAllowedError`
  - : An attempt was made to insert the HTML into a node whose parent is a {{domxref("Document")}}.

## Usage notes

The `innerHTML` property can be used to examine the current HTML source of the page, including any changes that have been made since the page was initially loaded.

### Reading the HTML contents of an element

Reading `innerHTML` causes the user agent to serialize the HTML or XML fragment comprised of the elment's descendants. The resulting string is returned.

```js
let contents = myElement.innerHTML;
```

This lets you look at the HTML markup of the element's content nodes.

> **備註：** The returned HTML or XML fragment is generated based on the current contents of the element, so the markup and formatting of the returned fragment is likely not to match the original page markup.

> **備註：** 基於元素的當前內容產生返回的 HTML 或 XML 片段，所以返回的片段的標記和格式可能不匹配原始頁面標記。

### Replacing the contents of an element

Setting the value of `innerHTML` lets you easily replace the existing contents of an element with new content.

For example, you can erase the entire contents of a document by clearing the contents of the document's {{domxref("Document.body", "body")}} attribute:

```js
document.body.innerHTML = "";
```

This example fetches the document's current HTML markup and replaces the `"<"` characters with the HTML entity `"&lt;"`, thereby essentially converting the HTML into raw text. This is then wrapped in a {{HTMLElement("pre")}} element. Then the value of `innerHTML` is changed to this new string. As a result, the document contents are replaced with a display of the page's entire source code.

```js
document.documentElement.innerHTML = "<pre>" +
         document.documentElement.innerHTML.replace(/</g,"&lt;") +
            "</pre>";
```

#### Operational details

What exactly happens when you set value of `innerHTML`? Doing so causes the user agent to follow these steps:

1. The specified value is parsed as HTML or XML (based on the document type), resulting in a {{domxref("DocumentFragment")}} object representing the new set of DOM nodes for the new elements.
2. If the element whose contents are being replaced is a {{HTMLElement("template")}} element, then the `<template>` element's {{domxref("HTMLTemplateElement.content", "content")}} attribute is replaced with the new `DocumentFragment` created in step 1.
3. For all other elements, the element's contents are replaced with the nodes in the new `DocumentFragment`.

### Security considerations

It is not uncommon to see `innerHTML` used to insert text into a web page. There is potential for this to become an attack vector on a site, creating a potential security risk.

```js
const name = "John";
// assuming 'el' is an HTML DOM element
el.innerHTML = name; // harmless in this case

// ...

name = "<script>alert('I am John in an annoying alert!')</script>";
el.innerHTML = name; // harmless in this case
```

Although this may look like a [cross-site scripting](https://zh.wikipedia.org/wiki/cross-site_scripting) attack, the result is harmless. HTML5 specifies that a {{HTMLElement("script")}} tag inserted with `innerHTML` [should not execute](https://www.w3.org/TR/2008/WD-html5-20080610/dom.html#innerhtml0).

However, there are ways to execute JavaScript without using {{HTMLElement("script")}} elements, so there is still a security risk whenever you use `innerHTML` to set strings over which you have no control. For example:

```js
const name = "<img src='x' onerror='alert(1)'>";
el.innerHTML = name; // shows the alert
```

For that reason, it is recommended that you do not use `innerHTML` when inserting plain text; instead, use {{domxref("Node.textContent")}}. This doesn't parse the passed content as HTML, but instead inserts it as raw text.

> **警告：** If your project is one that will undergo any form of security review, using `innerHTML` most likely will result in your code being rejected. For example, [if you use `innerHTML`](https://wiki.mozilla.org/Add-ons/Reviewers/Guide/Reviewing#Step_2:_Automatic_validation) in a [browser extension](/zh-TW/docs/Mozilla/Add-ons/WebExtensions) and submit the extension to [addons.mozilla.org](https://addons.mozilla.org/), it will not pass the automated review process.

## Example

This example uses `innerHTML` to create a mechanism for logging messages into a box on a web page.

### JavaScript

```js
function log(msg) {
  var logElem = document.querySelector(".log");

  var time = new Date();
  var timeStr = time.toLocaleTimeString();
  logElem.innerHTML += timeStr + ": " + msg + "<br/>";
}

log("Logging mouse events inside this container...");
```

The `log()` function creates the log output by getting the current time from a {{jsxref("Date")}} object using {{jsxref("Date.toLocaleTimeString", "toLocaleTimeString()")}}, and building a string with the timestamp and the message text. Then the message is appended to the box with the class `"log"`.

We add a second method that logs information about {{domxref("MouseEvent")}} based events (such as {{event("mousedown")}}, {{event("click")}}, and {{event("mouseenter")}}):

```js
function logEvent(event) {
  var msg = "Event <strong>" + event.type + "</strong> at <em>" +
            event.clientX + ", " + event.clientY + "</em>";
  log(msg);
}
```

Then we use this as the event handler for a number of mouse events on the box that contains our log:

```js
var boxElem = document.querySelector(".box");

boxElem.addEventListener("mousedown", logEvent);
boxElem.addEventListener("mouseup", logEvent);
boxElem.addEventListener("click", logEvent);
boxElem.addEventListener("mouseenter", logEvent);
boxElem.addEventListener("mouseleave", logEvent);
```

### HTML

The HTML is quite simple for our example.

```html
<div class="box">
  <div><strong>Log:</strong></div>
  <div class="log"></div>
</div>
```

The {{HTMLElement("div")}} with the class `"box"` is just a container for layout purposes, presenting the contents with a box around it. The `<div>` whose class is `"log"` is the container for the log text itself.

### CSS

The following CSS styles our example content.

```css
.box {
  width: 600px;
  height: 300px;
  border: 1px solid black;
  padding: 2px 4px;
  overflow-y: scroll;
  overflow-x: auto;
}

.log {
  margin-top: 8px;
  font-family: monospace;
}
```

### Result

The resulting content looks like this. You can see output into the log by moving the mouse in and out of the box, clicking in it, and so forth.

{{EmbedLiveSample("Example", 640, 350)}}

## Specification

{{Specifications}}

## Browser compatibility

{{Compat("api.Element.innerHTML")}}

## See also

- {{domxref("Node.textContent")}} and {{domxref("Node.innerText")}}
- {{domxref("Element.insertAdjacentHTML()")}}
- Parsing HTML into a DOM tree: {{domxref("DOMParser")}}
- Serializing XML or HTML into a DOM tree: {{domxref("XMLSerializer")}}
