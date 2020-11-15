Notepad++ is a light-weight editor for viewing files and quick editing when using other heavy-weight editors would be too painful. But its dark themes don't work with Markdown files (`.md`). Here are some brief notes on simple tinkering to get dark themes to work with Markdown.

<!-- TEASER_END -->

## Quick Fix For Markdown With Notepad++
I tried [Markdown-Plus-Plus](https://github.com/Edditoria/markdown-plus-plus) but the themes provided still seemed to render weird highlighting on all the text *sometimes*. I'm using the latest Notepad++ 7.9.1 at the time of writing this.

The trick I found was to delete the pre-installed XML file `markdown._preinstalled.udl.xml` from `%APPDATA%\Notepad++\userDefineLangs`. In my setup, this folder is empty right now.

## Material Theme (Optional)
You can add a theme defined using an XML file to Notepad++ by placing it at `%APPDATA%\Notepad++`. I like this [Material Theme](https://github.com/Codextor/npp-material-theme) created for Notepad++. Follow the simple [installation instructions](https://github.com/Codextor/npp-material-theme#installation) from the site.

