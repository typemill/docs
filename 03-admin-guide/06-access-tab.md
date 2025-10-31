#  Access Tab

UPDATE: In the access tab, you can activate and configure access restrictions for the website or individual pages. You can also determine the redirect targets for different user groups after they log in. Please also read the article about [user roles](/admin-guide/users) and [page restrictions](/author-guide/page-restrictions). Please note that session cookies are used for all frontend visitors to handle page restrictions.

![Screenshot of the access tab of Typemill](media/live/typemill-restrict-pages.webp){.center loading="lazy" width="794" height="498"}

| Option | Description | 
|:---|:---|
| Website Restriction | Show the website only to authenticated users and redirect all other users to the login page. This option is useful if you want to prepare your website before launch or if you want to keep the entire website internal. | 
| Page Restriction | Activate individual restrictions for specific pages. If you activate this option, a new fieldset for access restrictions will be visible in the meta-tab of each page. | 
| Hide Restricted Pages | Activate this option to hide restricted pages from the navigation in the frontend and only show the navigation items to users with enough rights. Be aware that this option might slow down websites with many pages. | 
| Content Break | By default, the content of a restricted page will be cut off after the title. If you activate the content break, you can truncate the content for unauthorized users with an hr-element and display a restriction notice below. | 
| Restriction Notice | Display a notice to inform users about access restrictions. You can use Markdown for the notice. | 
| Wrap Restriction Notice | Enclose the restriction notice in a notice-4 element, which can be styled as a special box. | 
| Redirect after login for users with admin rights | Redirect users with admin rights to a specified page after login. The default is the content editor. Options include system page, editor page, account page, and homepage. | 
| Redirect after login for users with edit rights | Redirect users with edit rights to a specified page after login. The default is the content editor. Options include editor page, account page, and homepage. | 
| Redirect after login for users without edit rights | Redirect users without edit rights to a specified page after login. The default is the account settings. Options include account page and homepage. |

