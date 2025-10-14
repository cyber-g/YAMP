# YAMP
YAMP - Yet Another Markdown Previewer : A lightweight, markdown preview generator (simple and quick)

YAMP is a lightweight markdown previewer inspired by [Just a simple markdown previewer](https://blog.meain.io/2021/offline-markdown-preview/) (2021). 
The JSMP was tailored for macOS. This project adapts and extends it for Linux users, with a few personal tweaks.

## Porting to Linux

Adapting the original JSMP to Linux is easy to make it a simple command-line tool:

```bash
pandoc --no-highlight --template $HOME/bin/markdown-preview-dir/pandoc-github-template.html --output "$filename" -f gfm -t html5 --metadata title="$1 - Preview" "$1"
```

While this worked for basic markdown, it failed to properly render LaTeX equations. 
In addition, please note that admonitions are not properly supported with this approach.

### The fix: KaTex

Digging through my previous personal Makefiles, I found a simpler and more effective approach using KaTeX:

```bash
pandoc input.md -s \
  --katex=https://cdn.jsdelivr.net/npm/katex/dist/ \
  -o output.html
```

This solution renders equations beautifully with minimal setup.

### Customization: Adjusting the Page Width

One small annoyance remained: the default HTML body width in pandoc is set to `36em` (docs [reference](https://pandoc.org/demo/example33/6.2-variables.html#variables-for-html)). As shown in the doc, it can be changed by setting the `maxwidth` variable (which sets in return the `max-width` CSS property). 

Pandoc allows you to pass metadata via a separate YAML file or in-the-document, at the top of the document. Instead of modifying files manually, I chose to inject the metadata dynamically and pipe it directly into pandoc. This way, the previewer works out-of-the-box with a more comfortable body width.

```bash
filename=$(basename "$1" .md)
(
    printf -- "---\nmaxwidth: 80%%\n---\n\n"
    cat "$1"
) | pandoc -s \
        --katex=https://cdn.jsdelivr.net/npm/katex/dist/ \
        --output "$filename.html"
```

The final version uses some processing to use default width or a user-defined width (passed as a second argument).
