#  Themes

Typemill provides different themes and customization options in the theme section of the system area. Here you can add, activate, and configure the themes.

![Screenshot of the theme section in the system settings of Typemill](media/live/typemill-themes-admin.webp){.center loading="lazy" width="821" height="572"}

## Theme Directory

Typemill ships with a standard theme called "Cyanine." Cyanine is a multi-purpose theme that you can use for documentation as well as for any other kind of informational website. You can select and download more themes from the [Typemill Theme Directory](https://themes.typemill.net).

![Screenshot of the theme directory of Typemill](media/live/themes-redesigned.webp){.center loading="lazy" width="820" height="592"}

Many themes are free to use. Themes that require a license include a button indicating the specific license: MAKER or BUSINESS.

## Install a Theme

You can add as many themes to your Typemill website as you want. Just download a new theme from the theme directory, unzip it, and then upload it (e.g., with FTP) to the theme folder of your Typemill website (e.g., `/themes/pilot`).

After uploading the theme, the folder structure should look like this:

```YAML
/typemill
  / themes
    /pilot
     - pilot.yaml
     - pilot.php
     - [more theme files and theme folders]
```

After uploading the theme to your Typemill installation, please refresh the page with the listed themes, and the new theme should be part of the list. To switch a theme, just click on the activate checkbox at the top.

## Configure a Theme

Each theme can have very individual configuration options. The only configuration option that is common for all themes is a custom CSS field. Here you can add your own CSS to make small changes to the frontend design if needed. To open the configuration options, click on the configuration button.

Some themes, like the Cyanine theme, the Guide theme, and the Pilot theme, provide **readymades** at the top of the configuration settings. Readymades are predefined settings that help you switch the configurations of a theme quickly. Just load a readymade, save the settings with the button at the bottom, and check the frontend page. You can also save your own configuration as a readymade. This is very helpful and recommended if you want to experiment with some configurations and have the option to restore your original configurations later.

![Screenshot of the readymades provided by the Guide theme of Typemill](media/live/theme-readymades.webp){.center loading="lazy" width="820" height="591"}

## Update a Theme

If there is a new version of the theme available, then you will see an update banner in the theme area of the system panel. To update a theme, open your FTP software, go to the theme folder of your Typemill installation, delete the old folder of your theme (e.g., `/themes/typemill`), and upload the new folder.

## Use Bundles

The Typemill website also provide bundles for specific use cases that ship with a selection of helpful plugins and themes for different use cases. Check out the [different solutions](https://typemill.net/solutions) on the Typemill website.

## Develop Themes

If you are a developer or web designer, you can easily create your own theme with the Twig templating language. Please read the [theme documentation](/theme-developers) for more details.

