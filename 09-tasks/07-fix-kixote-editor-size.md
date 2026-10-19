# Fix Kixote Editor Size

The editor in Kixote sometimes cuts long text. With the following fix in the custom.css (author, css) the issues is resovled and long text are completely visibile:

```css
 /* Fix für Autosize Editor - vollständige Textanzeige */
[data-el="editor"][data-initialized="true"] {
  max-height: none !important;
  overflow: visible !important;
}

#kieditor {
  max-height: none !important;
  overflow: visible !important;
}

.codearea {
  max-height: none !important;
  overflow: visible !important;
} 
```

