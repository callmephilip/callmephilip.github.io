---
title: "Adding rich text editor to your HTMX project"
date: 2024-11-11T09:58:49Z
draft: false
---

How much extra javascript do you need to wire a reasonably ambitious and feature full rich text editor in your HTMX based app? Let's find out.   

> Some context: The very first build of [tinychat](https://github.com/callmephilip/tinychat) shipped with a plain text editor. tinychat uses HTMX to get the SPA feel going. Last week I added integration with [CK Editor 5](https://ckeditor.com/). Here is what it looks like:

![tinychat with rich text editor](/tinychat-editor.png)

Let me preface this by saying, I am not married to CK Editor as the choice for rich text editor and am happy to explore other alternatives. The approach to integrating it into HTMX app will remain the same as long as we have access to a js module build. CK Editor has a very helpful [builder](https://ckeditor.com/ckeditor-5/builder/) that allows you to customize your editor and get going very fast. 

After running the above mentioned builder, I ended up with a few things in my hands:  

- a [hosted](https://cdn.ckeditor.com/ckeditor5/43.3.0/ckeditor5.js) CKEditor5 [js module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) with a module import map
- a [hosted](https://cdn.ckeditor.com/ckeditor5/43.3.1/ckeditor5.css) stylesheet for the editor
- some js to glue things together and initialize the editor 

The next step was to figure out how to integrate this component into the app. Until now all I had in the app was a bunch of vanilla HTML elements laying around. I wanted to turn message composer area into a component that would be able to turn itself into a shiny rich text editor using CKEditor and that extra glue JS from previous step. Here's how I did it with [AlpineJS](https://alpinejs.dev/).

First things first, let's make some changes to the initialization js for the [editor.js](https://github.com/callmephilip/tinychat/blob/main/editor.js).

```js
import { ... } from "ckeditor5";

// this module exports a default function that takes care of config and initialization 
export default function ($el, { placeholder, formId }) {
  // a quick home made submit-on-enter plugin
  function SubmitOnEnter(editor) {
    editor.editing.view.document.on("enter", (evt, data) => {
      if (data.isSoft) { return; }
      data.preventDefault();
      evt.stop();
      document.querySelector(`#${formId} > input[type=hidden]`).value =
        editor.getData();
      htmx.trigger(`#${formId}`, "submit");
      editor.setData("");
      document.querySelector(`#${formId} > input[type=hidden]`).value = null;
    });
  }

  ClassicEditor.create($el, { /* editor config */ }).then(
    (editor) => editor.editing.view.focus()
  );
}
```

Back in the app, we can now import the module and initialize it. Here is what integration looks like:

```html
<div
    x-data="{ placeholder: 'Message #general', formId: 'f-1' }"
    x-init="import('editor.js').then((editor) => editor.default($el, $data))">
</div>
```

The underlying form responsible for sending the message looks as something like this:

```html
<form enctype="multipart/form-data" hx-post="/messages/send/1"
    hx-target="#channel-1" id="f-1" ... >
    <!-- ... -->
</form>
```

`x-init` takes care of the setup by first loading `editor.js` module containing initialization code and then running initialization code. Notice `$el` and `$data` grabbing attached element and data respectively.
