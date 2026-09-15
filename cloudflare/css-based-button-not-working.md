# CSS Based Button Not Working

## Our Fix

Disable Rocket Loader in Cloudflare (Dashboard Fix)
Go to your Cloudflare Dashboard for gosmart.health.
Navigate to Speed → Optimization → Content Optimization.


## Our Root Cause

What Is Actually Happening
The failure is caused by a conflict between Cloudflare Rocket Loader and your strict Content Security Policy (CSP) in production.

1. Direct Backend Output (Server-side)
When querying Gunicorn directly on the production VPS (127.0.0.1:8000), Django renders the inline <script> normally:

html


<script nonce="fFYgKoi0XUmimynPJ81Dgw==">
    var menuToggle = document.getElementById('menu-toggle');
    ...
2. Cloudflare Modification in Transit
Before the HTML reaches the client browser, Cloudflare's Rocket Loader intercepts the response and rewrites the tag to:

html


<script nonce="..." type="233a946e068b6e7cebc8a58e-text/javascript">
3. Execution Block in the Browser
MIME type bypass: Because the browser sees type="...-text/javascript" instead of standard JavaScript, it treats the script as a data block and refuses to execute it natively.
CSP blocks Rocket Loader: Rocket Loader's runner script (rocket-loader.min.js) is supposed to execute the deferred script dynamically. However, your production Content Security Policy strictly enforces:
http


script-src 'self' 'nonce-...' https://challenges.cloudflare.com https://cdnjs.cloudflare.com
Because the policy does not allow 'unsafe-eval' or 'strict-dynamic', the browser blocks Rocket Loader from dynamically evaluating or injecting the script.
Result: The script never executes at all. The event listeners for #menu-toggle, submenu toggles, and the cookie banner are never registered.
Locally, there is no Cloudflare proxy or Rocket Loader, so the script executes natively without interference.

How to Fix It
You can resolve this using either (or both) of the following methods:

Solution 1: Tell Rocket Loader to Ignore the Script (Code Fix)
Add data-cfasync="false" directly to the <script> tag in 
mysite/templates/base.html
:

html


<script data-cfasync="false" nonce="{{ request.csp_nonce }}">
Cloudflare explicitly respects data-cfasync="false" and will leave the script untouched as native JavaScript.

TIP

Moving the navigation script from inline base.html into the external static bundle 
mysite/static/js/mysite.js
 (loaded with <script data-cfasync="false" src="{% static 'js/mysite.js' %}"></script>) is even cleaner because 'self' in your CSP allows static scripts without relying on per-request nonces.

Solution 2: Disable Rocket Loader in Cloudflare (Dashboard Fix)
Go to your Cloudflare Dashboard for gosmart.health.
Navigate to Speed → Optimization → Content Optimization.
Toggle Rocket Loader to Off (or configure a Rule to bypass it for gosmart.health/*).
(Rocket Loader was designed in the HTTP/1.1 era to bundle script downloads; with modern HTTP/2 and HTTP/3 multiplexing already enabled on your domain, it provides negligible benefit and frequently breaks CSP nonces and modern JS).