# Integrate External Content

Some publishers want to integrate external content into Typemill, for example, Google Ads. If you use any **official plugins or themes** to integrate such external content, then the theme or plugin will handle it automatically. However, if you **add a script manually** (e.g., by pasting it directly into your theme templates), the script will be **blocked** and not loaded.  

This behavior is controlled by a security feature called **Content Security Policy (CSP)**, which prevents your website from unintentionally loading dangerous scripts or external content.

You can **whitelist external domains** in the **"Developer"** tab of Typemill’s system settings. There you will find:

* a checkbox to **disable CSP completely** *(not recommended!)*, and  
* an input field where you can add a **comma-separated list of URLs** that are allowed to load external content.

![Screenshot CSP whitelist in Typemill](media/live/typemill-csp-whitelist.webp){.center loading="lazy" width="819" height="421"}

You’ll need to identify which domains a service tries to load content from. You can find this information in the service’s documentation or by checking your browser’s developer tools:

1. Open your browser settings and search for **“Web Developer Tools”** (or similar).  
2. A new panel will open at the bottom or side of the browser window.  
3. Open the **“Network”** tab and reload the page where the content gets blocked.  
4. You will see error messages showing which domains have been blocked.  
5. Copy those domains and add them to the whitelist as described above.

![Screenshot Developer Tab with CSP](media/live/typemill-csp-example.webp){.center loading="lazy" width="820" height="510"}

This might seem like inconvenient extra work, but it is an important security measure that protects you and your visitors.

