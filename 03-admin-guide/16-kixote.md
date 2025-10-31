# Kixote for Admins

Kixote is a separate interface for Typemill designed for admins and authors. It offers a growing number of administrative `commands`, as well as `prompts` with AI integration.

## The Kixote Interface

Faced with the challenge of integrating more features into software without compromising simplicity and cleanliness, Typemill has created its own solution: Kixote. This new conversational interface for administrative tasks provides an alternative to button overload.

Activating Kixote is straightforward: a small button in the author interface header reveals Kixote as an overlay. It's akin to a command line interface or a basic chatbot, offering a user-friendly way to navigate administrative functions.

![Screenshot of the Kixote interface of Typemill.](media/live/kixote-admin.webp){.center loading="lazy" width="821" height="420"}

## List of Commands

To discover available commands, simply input `help` and hit return. Kixote supports provides the following commands:

* `help`: Displays a list of all commands with brief descriptions.
* `exit`: Closes the Kixote interface.
* `clear navigation`: Clears the cached navigation data.
* `clear cache`: Removes cached files, including twig-cache, from the cache folder.
* `show security log`: Reveals the security log, which can be enabled in the security tab of the system settings.
* `clear security log`: Deletes the entries in the security log.

Kixote's responses can include action buttons, allowing you to use buttons instead of manually typing commands.

## Prompts with Kixote

Intriguingly, Kixote is set to expand its functionality by handling prompts. For AI features and prompting, please read the [introduction to Kixote](/author-guide/ai-with-kixote) in the author guide.

