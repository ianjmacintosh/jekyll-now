---
published: false
---
Moving blocking JS calls from the top of your page to the bottom is a very well-established performance recommendation -- you shouldn't block your visitors from _seeing_ your content before it becomes interactive. I followed this advice to update our entire site's behavior, even on pages that used Handlebars to render big important parts of the page. On those pages, I put a `<script>` tag right in the middle of the page to render the form then and there. But now that the dependencies weren't there, I had to make that script call at the bottom of the page.

This meant those pages appeared to load slower with this performance technique. But I imagined it would speed up the visual load time for all the others.

So the question I needed to answer was: if this performance recommendation is going to be mostly good for our site, how bad is it for those pages? And a sharp follow-up question (which I'm not sure how to answer yet) would be _how_ good is it for the rest of the site?

My solution was to use the User Timing API to record a mark at the top of the `<head>` tag, another mark after the call to render the form, and measure the difference. I determined the difference varied depending on network conditions, but was fairly consistently 300-400ms slower.

### Measurement 1: Old Approach to Loading Handlebars Content
```html
<head>
  <script>
    // Do Handlebars stuff here
    ...
  
    // Make a mark
    performance.mark("mark_start_loading_head");
  </script>
  ...
</head>
<body>
  <script src="Handlebars.min.js"></script>
  <script src="templates.js"></script>
  <script src="all-the-things.js"></script>
</body>
```

```js
(function doAllTheThings() {
  // Do a bunch of stuff
  ...

  // Then mark this point in time
  performance.mark("mark_render_form");
  
  // And assuming you've got your first mark around, you can complete
  performance.measure("measure_time_from_head_until_form_render", "mark_start_loading_head", "mark_render_form");
  console.log(performance.getEntriesByName("measure_time_from_head_until_form_render")[0].duration + "ms until loaded form");
}());
```

### Measurement 2: New Approach to Loading Handlebars Content
```html
<head>
  <script>
    performance.mark("mark_start_loading_head");
  </script>
  ...
</head>
<body>
  ...
  <script src="all-the-things.js"></script>
</body>
```

```js:all-the-things.js
(function doAllTheThings() {
  // Do a bunch of stuff
  ...

  // Then mark this point in time
  performance.mark("mark_render_form");
  
  // And assuming you've got your first mark around, you can complete
  performance.measure("measure_time_from_head_until_form_render", "mark_start_loading_head", "mark_render_form");
  console.log(performance.getEntriesByName("measure_time_from_head_until_form_render")[0].duration + "ms until loaded form");
}());
```