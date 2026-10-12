# 1 Line per Sentence in Non-Code Writings

Whenever you write non code text that is meant to be rendered in a different format, use 1 line per sentence.
The file types referred to, include Markdown, LaTeX, Typst, Asciidoc, HTML, reStructuredText, and possibly more.
In all of these file types, a single line break doesn't create a line break at the rendered output.
Hence either using 80 characters per line, or no line breaks at all between paragraphs, is a possible idiom.
Using 1 line per sentence is the best Git-aware idiom to use, and I would like it to be used in every such file I write, even if it is not Git-tracked.

An exception to this rule, is in markdown GitHub comments which I may sometimes ask you to phrase for me.
GitHub always renders a linebreak when the user puts it in comments and issue/PR descriptions

Sometimes you may encounter situations where I ask you to write docs and the surrounding text in the same file doesn't comply with this idiom.
In these situations, we might observe that upstream's CI is enforcing one idiom or another, so in that case we don't have a choice but to comply to it.
In case there's no CI enforcement of writing style, we should write the docs in the following way:

If we are editing a single sentence or adding just a few sentences in an existing paragraph, keep using the same idiom upstream uses there.
If we are creating a new paragraph, the new paragraph should comply to this 1 sentence per line idiom described and requested here.
