#  Developer Tab

UPDATE: In the developer tab, you can configure detailed developer settings such as error reporting, proxy detection, CORS headers, and CSP headers. Note that some of these configurations require deeper technical knowledge.

![Screenshot of the developer tab of Typemill](media/live/developer-tab.webp){.center loading="lazy" width="819" height="506"}

| Setting | Description | 
|:---|:---|
| Error Reporting | Activate this to display application errors in the frontend. This is helpful for bug fixing. You should not activate this on production websites or should deactivate it immediately after bug fixing because error messages expose sensitive information to potential attackers. | 
| Twig Cache | Activate the cache for Twig templates. This can improve the performance of the frontend website. Be aware that with template caching, it can happen that changes are not immediately visible in the frontend. You can clear the cache with Kixote. | 
| Autorefresh Navigation | Activate this option to refresh the navigation automatically every 10 minutes. You can use this option if you want to add or remove pages in the content folder manually (e.g., via FTP) without using the author interface. Changes will be visible in the navigation after the next refresh. | 
| Refresh Interval | Add the interval in minutes for the autorefresh of the navigation. Defaults to 10 minutes. | 
| Proxy | Activate this feature to use the X-Forwarded header if you run Typemill behind a proxy. | 
| Proxy Base URL | If your proxy setup does not work correctly, try adding the base URL of your site here, like `https://mywebsite.com`. | 
| Trusted IPs for Proxies | If you want to add more security, you can add a comma-separated list of IPs for your proxy. | 
| Disable Custom Headers | You can disable all custom headers of Typemill except for CORS and send your own headers instead. | 
| Disable CSP Headers | Disable all CSP (Content Security Policy) headers for your website. | 
| Allowed Domains for Content Integration on Typemill (CSP) | List all domains separated by commas to allow content integration, such as iframes, on your Typemill website. Domains will be added to the CSP header. This is usually done with plugins and [themes](/theme-developers/theme-configuration), but you can also add them manually if something is blocked. | 
| Trusted IPs for API Access | List all IPs separated by commas that should have access to the Typemill API. This is the preferred security method if the requesting server has static IPs. Read the chapter about the [Typemill API](/api/introduction) for more details. | 
| Trusted Hosts for API Access | List all hosts separated by commas that should have access to the Typemill API. Read the chapter about the [Typemill API](/api/introduction) for more details. | 
| Login with Link | Allow selected guest users to log in with a login link. Read the article about the [login link feature](/admin-guide/login-link) for more details. | 
| Trusted IPs for the Login Link Referrer | A comma-separated list of domains that are accepted as referrers for login links. Your application has to send the referrer header to make this work. |

