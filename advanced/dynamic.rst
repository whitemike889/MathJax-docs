.. _ajax-mathjax:

***************************
Loading MathJax Dynamically
***************************

MathJax is designed to be included via a ``<script>`` tag in the
``<head>`` section of your HTML document, and it does rely on being
part of the original document in that it uses an ``onload`` or
``DOMContentLoaded`` event handler to synchronize its actions with the
loading of the page.  If you wish to insert MathJax into a document
after it has been loaded, that will normally occur *after* the page's
``onload`` handler has fired, and prior to version 2.0, MathJax had to
be told not to wait for the page ``onload`` event by calling
:meth:`MathJax.Hub.Startup.onload()` by hand.  That is no longer
necessary, as MathJax v2.0 detects whether the page is already
available and when it is, it processes it immediately rather than
waiting for an event that has already happened.

Here is an example of how to load and configure MathJax dynamically:

.. code-block:: javascript

    (function () {
      var script = document.createElement("script");
      script.type = "text/javascript";
      script.src  = "https://example.com/MathJax.js?config=TeX-AMS-MML_CHTML";
      document.getElementsByTagName("head")[0].appendChild(script);
    })();

If you need to provide in-line configuration, you can do that using a
MathJax's configuration script:

.. code-block:: javascript

    (function () {
      var head = document.getElementsByTagName("head")[0], script;
      script = document.createElement("script");
      script.type = "text/x-mathjax-config";
      script[(window.opera ? "innerHTML" : "text")] = 
        "MathJax.Hub.Config({\n" +
        "  tex2jax: { inlineMath: [['$','$'], ['\\\\(','\\\\)']] }\n" +
        "});";
      head.appendChild(script);
      script = document.createElement("script");
      script.type = "text/javascript";
      script.src  = "https://example.com/MathJax.js?config=TeX-AMS-MML_CHTML";
      head.appendChild(script);
    })();

You can adjust the configuration to your needs, but be careful to get
the commas right, as Internet Explorer 6 and 7 will not tolerate an
extra comma before a closing brace.  The ``window.opera`` test is
because some versions of Opera don't handle setting ``script.text``
properly, while some versions of Internet Explorer don't handle
setting ``script.innerHTML``.

Note that the **only** reliable way to configure MathJax is to use an
in-line configuration block of the type discussed above.  You should
**not** call :meth:`MathJax.Hub.Config()` directly in your code, as it will
not run at the correct time --- it will either run too soon, in which case
``MathJax`` may not be defined and the function will throw an error, or it
will run too late, after MathJax has already finished its configuration
process, so your changes will not have the desired effect.


MathJax and GreaseMonkey
========================

You can use techniques like the ones discussed above to good effect in
GreaseMonkey scripts.  There are GreaseMonkey work-alikes for all the
major browsers:

- Firefox: `GreaseMonkey <http://addons.mozilla.org/firefox/addon/748>`_
- Safari: `TamperMonkey <https://tampermonkey.net>`_
- Microsoft Edge:  `TamperMonkey <https://tampermonkey.net>`_
- Internet Explorer: `IEPro7 <http://ie7pro.blogspot.co.uk/>`_
- Chrome:  `TamperMonkey <https://tampermonkey.net>`_
- Opera Next:  `TamperMonkey <https://tampermonkey.net>`_

Note, however, that most browsers don't allow you to insert a script
that loads a ``file://`` URL into a page that comes from the web (for
security reasons).  That means that you can't have your GreaseMonkey
script load a local copy of MathJax, so you have to refer to a
server-based copy.  One of the CDNs that serve MathJax works nicely for this.

----

Here is a script that runs MathJax in any document that contains
MathML (whether it includes MathJax or not).  That allows 
browsers that don't have native MathML support to view any web pages
with MathML, even if they say it only works in Firefox and
IE+MathPlayer.

.. code-block:: javascript

    // ==UserScript==
    // @name           MathJax MathML
    // @namespace      http://www.mathjax.org/
    // @description    Insert MathJax into pages containing MathML
    // @include        *
    // ==/UserScript==

    if ((window.unsafeWindow == null ? window : unsafeWindow).MathJax == null) {
      if ((document.getElementsByTagName("math").length > 0) ||
          (document.getElementsByTagNameNS == null ? false : 
          (document.getElementsByTagNameNS("http://www.w3.org/1998/Math/MathML","math").length > 0))) {
        var script = document.createElement("script");
        script.type = "text/javascript";
        script.src = "https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_CHTML-full";
        document.getElementsByTagName("head")[0].appendChild(script);
      }
    }

**Source**: `mathjax_mathml.user.js <_static/mathjax_mathml.user.js>`_

----

Here is a script that runs MathJax in Wikipedia pages after first
converting the math images to their original TeX code.  

.. code-block:: javascript

    // ==UserScript==
    // @name           MathJax in Wikipedia
    // @namespace      http://www.mathjax.org/
    // @description    Insert MathJax into Wikipedia pages
    // @include        http://en.wikipedia.org/wiki/*
    // ==/UserScript==

    if ((window.unsafeWindow == null ? window : unsafeWindow).MathJax == null) {
      //
      //  Replace the images with MathJax scripts of type math/tex
      //
      var images = document.getElementsByTagName('img'), count = 0;
      for (var i = images.length - 1; i >= 0; i--) {
        var img = images[i];
        if (img.className === "tex") {
          var script = document.createElement("script"); script.type = "math/tex";
          if (window.opera) {script.innerHTML = img.alt} else {script.text = img.alt}
          img.parentNode.replaceChild(script,img); count++;
        }
      }
      if (count) {
        //
        //  Load MathJax and have it process the page
        //
        var script = document.createElement("script");
        script.type = "text/javascript";
        script.src = "https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_CHTML-full";
        document.getElementsByTagName("head")[0].appendChild(script);
      }
    }

**Source**: `mathjax_wikipedia.user.js <_static/mathjax_wikipedia.user.js>`_
