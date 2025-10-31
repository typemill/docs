# Developer Guide: Upgrade Themes

The process to update themes should be straight forward, since most logic for themes stayed the same. However, you should individually check for errors that might result from new ... Some known issues resulting from changes are:

## External Resources and CSP

If your theme uses any external resources (like externally hosted javascript libraries or css libraries), then those resources will be blocked by the content security policy (csp) in Typemill version 2.1.0. To load external libraries, you have to add the domains into your yaml-definitions under the new item "csp" (preferably at the end of your definitions).

## Internal Resources from Typemill

If you have integrated any internal resources from Typemill, then you have to change the url to the new location:

* Old location: `/system/author/js/jsfile.js`
* New location: `/system/typemill/author/js/jsfile.js`

In most themes you probably have integrated the `typemillutils.js` that loads the youtube-videos after a click. So you have to add the `/typemill/` segment like this:

```javascript
<script src="{{base_url}}/system/typemill/author/js/typemillutils.js"></script>
```



## Tachyons CSS not Available

Version 2 of Typemill switched from Tachyons CSS to Tailwind CSS. If you use Tachyons CSS in your Theme (which is still a recommendation), you should include the library in your theme now.

