# Develop the Yaml File

Each layout needs a special configuration file. The configuration file is a simple yaml-file. It is similar to the yaml-definitions for themes and plugins. 

## Basic Information

The yaml-file starts with some basic information. The information will be displayed next to the preview image of the layout:

````
Name: Typemill
Copyright: Sebastian Schürmanns, trendschau
Licence: Creative Commons BY 3.0
Link: https://trendschau.net
Sizes: A5 (148mm 210mm)
Photo: Photo by Markus Winkler on Unsplash
````

Nothing complicated here. Only the Name is mandatory. You can add as many information as you want, but keep in mind that it is displayed in the layout selection and too extensive information might destroy the layout.

## Define Everything with Forms

In the next section of the yaml-file you can define the forms that will appear in the settings-tab of the eBook. You can use all fields from the typemill [form-builder](/forms) there. The definitions start with the keyword `customforms`. Yes, you are right, this is flexible as hell, it means that you can define everything you need for you eBook-layout. Your eBook-cover has 3 images, 10 colors and 5 subtitles? No problem, just define it here!

These are the form-definitions for the eBook layout called "typemill": 

````
customforms:
  fields:
    fieldsettitlepage:
      type: fieldset
      legend: Covertext and Fly Title
      fields:
        title:
          type: text
          label: Title of your eBook
          description: Appears in the white box
        subtitle:
          type: text
          label: Subtitle of your eBook
        author:
          type: text
          label: Author
        edition:
          type: text
          label: Edition
        flytitle:
          type: checkbox
          label: Fly title
          checkboxlabel: Add a fly title after the cover. 
    fieldsetcoverimage:
      type: fieldset
      legend: Cover Image
      fields:
        coverimage:
          type: image
          label: Upload a cover image
          description: only jpg or png accepted. 
        coverimageonly:
          type: checkbox
          checkboxlabel: Use only the coverimage without background-colors and text.
    fieldsetcovercolors:
      type: fieldset
      legend: Cover Colors
      fields:
        primarycolor:
          type: text 
          label: Primary Background Color (top area)
          placeholder: cadetblue
          description: Use color name like 'black' or hex code like '#000000'.
        primaryfontcolor:
          type: text
          label: Primary Font Color (in white box)
          placeholder: black
          description: Use color name like 'black' or hex code like '#000000'.
        secondarycolor:
          type: text
          label: Secondary Background Color (bottom area)
          placeholder: cadetblue
          description: Use color name like 'black' or hex code like '#000000'.
        secondaryfontcolor:
          type: text
          label: Secondary Font Color (bottom area)
          placeholder: white
          description: Use color name like 'black' or hex code like '#000000'.
    fieldsetimprint:
      type: fieldset
      legend: Imprint
      fields:
        imprint:
          type: textarea
          label: Text for imprint (use markdown)
          description: Keep empty to skip the imprint.
    fieldsetdedication:
      type: fieldset
      legend: Dedication
      fields:
        dedication:
          type: textarea
          label: Text for dedication (use markdown)
          description: Keep empty to skip the dedication.
    fieldsettoc:
      type: fieldset
      legend: Table of Contents
      fields:
        toc:
          type: checkbox
          label: Table of contents
          checkboxlabel: Add an automatically generated table of contents.
        toctitle:
          type: text
          label: Title for the table of contents page
        toclevel:
          type: number
          label: How many headline-levels (1-6) should be included?
        toccounter:
          type: checkbox
          label: ToC Counter
          checkboxlabel: Add a counter before the headlines.
    fieldsetendnotes:
      type: fieldset
      legend: Endnotes
      fields:
        endnotes:
          type: checkbox
          checkboxlabel: Transform footnotes into endnotes
        endnotestitle:
          type: text
          label: Title for the endnotes page
    fieldsetblurb:
      type: fieldset
      legend: Blurb
      fields:
        blurb:
          type: textarea
          label: Text for the blurb (use markdown)
          description: Keep empty to skip the blurb.
    fieldsetimagequality:
      type: fieldset
      legend: Image Quality
      fields:
        originalimages:
          type: checkbox
          checkboxlabel: Use original images and do not resize them (use this for print only).
````

All standardforms will be rendered automatically as soon as the user selects your layout.

You can add, delete or change forms like you want and create stunning ebooks with it. However, the input fields `originalimages` and `endnotes` are connected to backend functionalities. Don't change them, if you want to use these functionalities:

```
    fieldsetendnotes:
      type: fieldset
      legend: Endnotes
      fields:
        endnotes:
          type: checkbox
          checkboxlabel: Transform footnotes into endnotes
        endnotestitle:
          type: text
          label: Title for the endnotes page
    fieldsetimagequality:
      type: fieldset
      legend: Image Quality
      fields:
        originalimages:
          type: checkbox
          checkboxlabel: Use original images and do not resize them (use this for print only).
```

You can use the input of all forms in the twig template later.

