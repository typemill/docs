#  Multilingual Websites

You can use the [projects feature](/author-guide/multi-project) to build multilingual websites with cross-links between languages. You can even use an AI service in Kixote to automate the whole translation process.

## Activate Projects for Multilingual Websites

To set up a multilingual website, activate projects in the system settings and select the multilingual option. Then, create new projects for each language, using the language code as the key on the left and the language name as the value on the right.

![Screenshot of a Multilingual Website Setup with Projects](media/live/multilanguage-settings.webp){.center loading="lazy" width="820" height="742"}

Please note that you can use either a website with projects or one with multilingual versions, but not both simultaneously.

## Creating Language Variants for Pages

Once your multilingual setup is configured, a dropdown menu will appear in the author area, allowing you to switch between language versions. Additionally, a new language tab will appear at the top of each page, where you can create translations.

Typemill treats your main website as the default language, with all projects serving as language variants. You always start from the base language when creating translations. Language versions also have a language tab, but it only links back to the base language and does not allow creating further translations.

To create a language version, enter a new URL segment for the translation in the input fields for each language and click "Create." Typemill will copy the original page into the project and link both pages for easy switching.

![Screenshot of a base language site with translations](media/live/multilanguage-basepage.webp){.center loading="lazy" width="820" height="504"}

Here are some key details about how the copy and link functions work:

* Once a page is created, the visit button links to the new page. If no page exists, the visit button directs to the homepage of that language version.
* If you create a URL where the parent page is missing, Typemill will return an error with a hint to create the parent element first.
* If you change an existing link and click "Create," the page will be copied again to the new location. Be careful to keep your language projects organized.
* If you add a link and the page already exists, the existing page will be linked.
* You can unlink a translation by clicking the "X" button.

## Check the Translation Status

On each page, you can check the translation status of the whole website with a matrix view. You can reach the matrix view with a link on each language page. The matrix view provides you with a detailed overview of which pages have not yet been translated into a language.

![Matrix view of all translation pages in Typemill](media/live/translation-matrix.webp){.center loading="lazy" width="820" height="469"}

## Activate AI Translations

To reduce manual translation work, you can use an AI service to automate the translation process. First, make sure that an [AI service](/author-guide/kixote-getting-started) is activated in the system settings. Then enable AI auto-translation in the Projects tab.

Once this option is enabled, every time you create a new translation page in the Language tab, Typemill automatically translates both the page content and its metadata into the target language. You can also refresh a translation with a button if you have updated the original page.

![AI translations with Typemill](media/live/ai-translations.webp){.center loading="lazy" width="820" height="444"}

