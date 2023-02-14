**On February 14, 2023, Josh ([joshrcastle@gmail.com](mailto:joshrcastle@gmail.com)) said this:**

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
