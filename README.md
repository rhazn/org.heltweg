# heltweg.org
"hugo server" for local devserver

git commit and deploy from github

## Theme
- currently Papermod v7
- https://github.com/adityatelange/hugo-PaperMod/wiki/Installation

## Generate bibliography
- install bibtex2html `brew install bibtex2html`
- generate html from bib `bibtex2html --footer "<a href="/">< Back to home</a>" --title "Bibliography" --style acm --sort-by-date --reverse-sort -o bibliography own.bib && mv bibliography*.html static/`