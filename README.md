# mdBook translate template

This is a minimal mdBook that sets up the infrastructure for translation to other languages. It contains one chapter with the text "Hello world!" and can be translated to my native language Marathi using the Globe toggle button on the top right. You can see it in action [here](https://joshichaitanya3.github.io/mdbook-translate-template/index.html). This is something I have set up for my personal reference, but might be useful for folks who want to setup translation for mdbooks in a clean environment. It assumes you are familiar with the [`mdbook`](https://github.com/rust-lang/mdBook) tool. It uses the [mdbook-i18n-helpers](https://github.com/google/mdbook-i18n-helpers) repository and draws a lot from it and other repos that are referenced within it.

> Note: This only sets up the infrastructure. The actual addition of the translations needs to be done manually by editing the .po files using Poedit or other dedicated editors.

## Instructions to construct this book

### Initialize the mdbook

This can be done by running 

```
mdbook init
```

which will generate `book.toml`, `chapter_1.md`, `SUMMARY.md` and other necessary files. I added "Hello world!" as the text for the chapter.

### Install mdbook-i18n-helpers

[mdbook-i18n-helpers](https://github.com/google/mdbook-i18n-helpers) is a repository by Google that provides translation infrastructure for mdBooks. You can install it using instructions from their website. This will provide the tools for setting up the translations.

### Get the Globe button 

Why not start with the fun part --- getting the Globe button on your mdBook to toggle between languages! This is added by customizing the index.hbs file. As mentioned in the [mdBook manual](https://rust-lang.github.io/mdBook/format/theme/index.html), you can customize any of the default theme files by simply including your modified file in an appropriate directory in your mdBook. Here, we add our own index.hbs file under theme/index.hbs, and only modify it to add the button and the option for Marathi. See lines [173-213](https://github.com/joshichaitanya3/mdbook-translate-template/blob/821126213591c615d3a807a974361aaa3abdb76e/theme/index.hbs#L173-L213). These are picked from the several examples of internationalized mdBooks available on [mdbook-i18n-helpers](https://github.com/google/mdbook-i18n-helpers) GitHub page.

### Customizing the behavior of the language picker button

Once you add this index.hbs file in the correct directory and serve your book, you should see the Globe button in your book! However, if you click it, the dropdown menu doesn't appear right under it. This can be changed by specifying the CSS for this. We will do this by including a file theme/css/language-picker.css. This specifies the behavior for `language-list`, which we refer to in index.hbs. To use the language-picker, we need to add 
```
[output.html]
additional-css = ["theme/css/language-picker.css"]
```
to our book.toml file.

### Generating the PO files

PO (Portable Object) is the file format used by the GNU gettext library for internationalization and localization purposes. 

We first extract the original English text and generate a messages.pot (PO template) file. We will use the special renderer `mdbook-xgettext` installed via the i18n-helpers crate:<a href="#fn1" id="ft1"><sup>1</sup></a>
```
MDBOOK_OUTPUT='{"xgettext": {"pot-file": "messages.pot"}}' \
  mdbook build -d po
```

Running this command should generate a directory called `po` in your top directory and a file `messages.pot` within it.

> Note: Since this file is an autogenerated template file and does not need version control, it is excluded from this repository via gitignore.

Once we have the template file, we generate the `.po` file for our particular language. The po files are named after the [ISO 639](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) language codes: Spanish would go into po/es.po, Marathi would go into po/mr.po, etc.

For a hypothetical language with code `xx`, we type
```
msginit -i po/messages.pot -l xx -o po/xx.po
```

I am using my native Marathi language in this repository!

### Add in the translations

The .po file is where the actual translation would go. It is not recommended to add the translation directly to the .po file, but to use a dedicated editor like [Poedit](https://poedit.net/).

> Note: Poedit might by default also compile a binary `.mo` file, which we don't need to generate the transation here. We gitignore it just in case.

### Enable the gettext preprocessor

Before serving, we need to enable the gettext preprocessor by adding the following to our book.toml:

```
[preprocessor.gettext]
```

### Serve the translation

It's time to serve the translation!

```
MDBOOK_BOOK__LANGUAGE=xx mdbook serve -d book/xx --open
```

### (Optional) Add Git URL button

This can be done simply by adding the line

```
git-repository-url = "https://github.com/joshichaitanya3/mdbook-translate-template"
```

under `[output.html]`.
### Deploying

You can build the book with translations by typing

```
MDBOOK_BOOK__LANGUAGE=xx mdbook build -d book/mr 
```

Deployment can be done in the same way as a normal mdbook, by copying the `book` directory contents to the server. 

## Caveat

For some reason, the language toggle doesn't work in the serve mode. If served normally, the other language doesn't build, and if served for the specific language, the toggle to English doesn't work. However, if you build with the new language, and open book/index.html directly in your browser, you can toggle back and forth successfully!

<p><sup>1</sup> I struggled with this step in the documentation for mdbook-i18n-helpers, but found useful info in TRANSLATING.md in the <a href="https://github.com/gbdev/gb-asm-tutorial">gb-asm-tutorial</a> repository which uses it. Need to figure out what went wrong with the official guide under USAGE.md in <a href="https://github.com/google/mdbook-i18n-helpers">mdbook-i18n-helpers</a>. <a href="#ft1" id="fn1">↩</a></p>
