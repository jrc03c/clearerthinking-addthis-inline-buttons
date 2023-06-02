# 2023-06-02

AddThis was shut down a few days ago on May 31, 2023. We decided to switch to [ShareThis](https://sharethis.com/) since it mimics the AddThis functionality almost exactly. This repo has now been updated with the new ShareThis setup. Here are the steps to update from AddThis to ShareThis:

**1) In the GuidedTrack program**, replace the old `*html` and `*trigger` with this:

```
*html
	<div class="sharethis-inline-share-buttons"></div>

*trigger: load-sharethis-inline-buttons
```

**2) In the HTML**, remove _all_ AddThis elements and scripts, and then add these two scripts:

```html
<!-- The script provided by ShareThis -->
<script
  type="text/javascript"
  src="https://platform-api.sharethis.com/js/sharethis.js#property=646c9eb040353a0019caeeca&product=inline-share-buttons"
  async="async"></script>

<!-- Our script -->
<script>
  $(window).on("load-sharethis-inline-buttons", () => {
    const interval = setInterval(() => {
      const container = document.querySelector(
        ".sharethis-inline-share-buttons"
      )

      if (!container) {
        return
      }

      clearInterval(interval)

      const stickies = Array.from(
        document.querySelectorAll(".st-sticky-share-buttons")
      )

      stickies.forEach(s => {
        s.parentElement.removeChild(s)
      })

      setTimeout(() => {
        window.__sharethis__.initialize()
      }, 10)
    }, 10)
  })
</script>
```

Note that there's no need to create new elements in the HTML for ShareThis; it'll create them automatically when it first loads into the page. The only element that's needed is the `.sharethis-inline-share-buttons` element defined in the `*html` block.

Note also that there's a little "flash" during which the ShareThis sticky buttons on the left side of the page disappear and reappear when the "load-sharethis-inline-buttons" event is triggered. That's because the JS listener invokes `window.__sharethis__.initialize()`, which must be called so that ShareThis can find the element created by the `*html` block and fill it with content. Unfortunately, in addition to initializing the inline buttons, that function also creates a new set of sticky buttons _without_ removing the existing sticky buttons, causing the page to end up containing duplicate sets of sticky buttons. To keep things tidy, our script deletes the existing sticky buttons first and _then_ calls `window.__sharethis__.initialize()`. So the "flash" is the removal and recreation of the sticky buttons caused by our script. I wasn't able to find a fix for this, but I also doubt that many people will notice or care.

— Josh ([joshrcastle@gmail.com](mailto:joshrcastle@gmail.com))

# 2023-02-14

The [`index.html`](index.html) file in this folder contains a `<script>` tag at the bottom that makes it possible to inject an [AddThis](https://addthis.com) script _even after_ it has previously been loaded into the page.

The problem in general seems to be that the AddThis script only runs once; and if the target element isn't present, or if the script has already been loaded into the page, then it fails silently. This created a problem for the Clearer Thinking team because they wanted to construct a GuidedTrack program that had _both_ a floating AddThis container on the left side of the page _and_ an inline AddThis container at the very end of the program. The existence of the floating container and its corresponding scripts were preventing the inline container from loading. So, my solution — which is SUPER hacky and not at all robust for a variety of use cases 😅 — is to do something like this:

1. Temporarily remove the floating container.
2. Remove the original AddThis scripts.
3. Delete all of the JS variables created by the AddThis scripts.
4. Inject the new script into the page.
5. After the new script loads, wait for a brief moment, and then add the floating container back into the page in its original location.

All of the above operates on a few assumptions.

**Assumption #1:** There's a `*html` block in the GT program that includes a target element into which the inline container will be loaded. This target element should have a class name `addthis_inline_share_toolbox`. Like this:

```
*html
	<div class="addthis_inline_share_toolbox"></div>
```

**Assumption #2:** There's a `*trigger` after the `*html` block that tells a JavaScript listener to inject the AddThis script to do its work (i.e., the steps above). Like this:

```
*trigger: inject-addthis-script
```

**Assumption #3:** There's a `<script>` element in the embedded program's HTML that matches the one in this folder's `index.html` file. Like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- head stuff -->
  </head>
  <body>
    <!-- body stuff -->

    <!-- script from index.html -->
    <script>
      // This script injects the inline AddThis.com buttons at the end of the
      // program. Feel free to remove this entire <script> element if you
      // decide that the inline buttons no longer make sense!
      // — Josh @ February 14, 2023

      // Listen for inject-addthis-script trigger
      $(window).on("inject-addthis-script", () => {
        // run script injection steps
      })
    </script>
  </body>
</html>
```

— Josh ([joshrcastle@gmail.com](mailto:joshrcastle@gmail.com))
