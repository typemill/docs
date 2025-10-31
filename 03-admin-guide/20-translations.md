# Translations - A Contribution Guide

You can select a language for the author interface in the system settings. Currently, a variety of languages are available, with most translations contributed by Typemill users.

Translating is an excellent way to support Typemill. It doesn't require any coding skills but significantly aids other users. You can contribute by translating missing words for an existing language or by introducing Typemill to a new language.

## Contribute Translations

To correct an existing translation, you can do so directly on GitHub. Simply navigate to the translation files on [GitHub](https://github.com/typemill/typemill/tree/v2.x/system/typemill/author/translations) and make your adjustments. A GitHub account is necessary for this.

```yaml
TRANSLATION_KEY:'the corresponding translation'  
ANOTHER_TRANSLATION_KEY:'and the corresponding translation'
```


Translation keys are in English and uppercase. Add the translation to the right side. This allows you to correct an existing translation or add a new one if it's missing:

```
I_AM_A_BUTTON:'I am a chair'    <<<<< this is wrong, you can fix! 
PLEASE_HELP:''                  <<<<< translation is missing, you can add!
ALL_FINE:'everything is fine'   <<<<< everything ok? Then do not change
```


On GitHub, log in (or create an account), click the edit button at the top left of the translation file, and, after making your changes, click "commit changes" at the top right. This notifies us to review and approve your changes.

![Screenshot of GitHub: How to edit translations](media/live/github.png){.center loading="lazy" width="820" height="521"}
*Edit translation files directly on github*

! **Very Important Rules**
! 
! * NEVER change, add, or delete a translation key on the left side, only add translations on the right. Changing the key will cause Typemill to lose the translation.
! * Do not alter the order of the translations.

## Add a new language

To add a new language, fork Typemill on GitHub and submit your translations back when ready.

Copy the en.yaml file in your fork, rename it to your language code, and begin translating the English text on the right into your language.

Remember:

* Do not change any translation keys.
* Keep the original order of translations.
* Avoid adding new lines. Only translate the English text.
* If unsure, ask before proceeding to ensure your translations can be merged without issues.

## Some Background

Translation files are processed automatically due to limited resources. The process involves two scripts:

* `TranslationScanner`: Scans the code of Typemill for strings within `translate("this_wording")` functions, converts them to uppercase keys, and sorts them alphabetically. 
* `TranslationFiller`: Reads existing translation files, matches them with keys from TranslationScanner, adds existing translations, and leaves blanks for missing ones. Removed translations are appended at the end for review.

This automated process updates translation files with new keys and appends removed ones at each file's end for the next Typemill release. Authors can then add translations for new keys and remove outdated ones.

Manual changes like key modifications or sorting alterations will be overwritten in the next update.

A final note: I understand that users may sometimes have the time to contribute and sometimes not. To ensure that existing translations remain up-to-date, I will occasionally use ChatGPT to fill in missing translations if there are no recent updates.

Thank you for contributions!

