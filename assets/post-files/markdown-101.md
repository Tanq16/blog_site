# Prefix line with a hash for heading 1

## Double hash for heading 2

Normal text is just normal text. You can go upto 6 hashes for headings 1-6.

---

Use the triple dash to insert a horizontal line separator.

In Markdown,
going to the next line
will not change the line in the final render.
Instead, all of it is considered a single
continuous paragraph.
To break the line, leave an empty line
between 2 paragraphs

like this. So the "like this" part appears in a new line.

---

**Bold**, *Italics*, and a ***combination of bold and italics*** can be achieved using astericks.

Strikethrough text can be achieved using the tilde symbol - ~like so~.

Inline code can be inserted with backticks `like so`.

Bullet lists can be inserted by using `-` or `*` characters -

- unordered
- bulleted
- list

* unordered
* list
* using `*` instead of `-`

Numbered list is simple too - 

1. numbered
2. list

Make sure to leave a line between any paragraph and a list to make sure the formatting is correct.

> Lines can be blockquoted using the `>` character.

You can insert hyperlinks using this format - `[hyperlink](webpage-link)`. So an example is -

[Tanishq's Markdown Guide](https://blog.tanishq.page/posts/newbified-markdown/)

Images can be inserted in the same way as links but by prepending a `!` before the link syntax, like so -

![my favicon](https://raw.githubusercontent.com/Tanq16/blog_site/main/assets/img/favicons/favicon.ico)

Inside the `()`, you can also specify the relative folder path like `../assets/image.png` for exmample, which will take in the file from your local folder.

---

Next, code blocks can be used by using triple backticks like so -

```
this is a code block
multiline code block
meant for either emphasizing or writing code
```

If your renderer supports syntax highlighting you can also mention the name of a programming language and the renderer or app will color highlight the syntax of the code like so -

```python
import json
import os

data = os.listdir()

mdcount = 0

for i in data:
    if i.endswith(".md"):
        mdcount += 1

print(mdcount)

with open("hello.json", "w") as f:
    f.write(json.dumps({"mdcount": mdcount}))
```

---

Lastly, let's look at tables -

| column heading 1 | column heading 2 |
| --- | --- |
| row 1 column 1 | row 1 column 2 |
| row 2 column 1 | row 2 column 2 |
| row 3 column 1 | row 3 column 2 |

And that's it!! Enjoy writing in Markdown. Check out the rendered version of this document on my blog too!
