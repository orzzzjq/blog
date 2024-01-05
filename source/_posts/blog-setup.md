---
title: How does this blog work?
date: 2024-01-05 18:32:00
tags: misc
mathjax: true
---

The pipeline for some of the articles in this blog is: LaTeX $\rightarrow$ HTML $\rightarrow$ Markdown.

<!-- more -->

This blog is based on the [Hexo framework](https://hexo.io/index.html) and uses the [Sky theme](https://github.com/iJinxin/hexo-theme-sky). Some of the articles is written in pure LaTeX since its easiler for me to write these notes using LaTeX. However, it took me a while to figure out how to convert LaTeX articles into HTML/Markdown so that it can be rendered by Hexo. I decide to write a tutorial for those who also want to use LaTeX as their main language for blogging.

See [Hard-margin SVM and polytope distance](https://orzzzjq.github.io/blog/2023/hard_margin_svm_pd/) for an example of the final result.

# LaTeX $\rightarrow$ HTML

The first and also the most difficult step is to convert LaTeX articles into HTML. Before this, I knew that nowadays many academic publishers actually support reading articles in HTML, instead of a PDF file. Also, I was impressed by [ar5iv](https://ar5iv.labs.arxiv.org/) which can convert arXiv articles into HTML5 pages. After some researching and trying, I found that [LaTeXML](https://math.nist.gov/~BMiller/LaTeXML/) is the tool that really works for me.

## Installation of LaTeXML

I recommend to use LaTeXML on Linux or WSL. I tried to install it on Windows following the official guides, but it fails and very few useful info can be found online. Thus I turned to WSL and it works fine.

Install LaTeXML via:

```
$ sudo apt-get install latexml
```

## Usage of LaTeXML

Compile the \*.tex file to html using:

```
$ latexmlc --dest=somewhere/mydoc.html mydoc
```

Then the html file will be genereated in the destination directory. If there are special image formats in the tex file (e.g. pdf), you may need ImageMagick for converting. However something went wrong on my computer and it does not work appropriately. Currently I use jpeg/png for images in the documents to avoid such issues.

# HTML $\rightarrow$ Markdown

Now we can embed the html file into markdown and thus it could be rendered by Hexo.

## Customize CSS styles

We can remove unneccessary sources in the html file and change the CSS files for better formatting. Currently I am using an [customized version](https://github.com/orzzzjq/blog/blob/source/source/mypage/css/ar5iv.min.css) of the ar5iv CSS file. The CSS file should be put at somewhere accessible by the html file (e.g. `source/mypage/css`).

## Embed HTML into Markdown

Directly copy the html source into markdown does not work appropriately. The reason is yet clear to me, probably the html is too massive. Instead, I use iframe to embed it into markdown.

Put the html file (together with the auxiliary folder if any) in `source/mypage`. Create a new post via Hexo under `source/_posts`, and embed the html into the markdown file via iframe:

```html
<iframe
  src="/blog/mypage/mydoc.html"
  scrolling="no"
  onload='javascript:(function(o){o.style.height=o.contentWindow.document.body.scrollHeight+100+"px";}(this));'
  style="height:200px;width:100%;border:none;overflow:hidden;"
></iframe>
```

The onload JS script is for adjusting the height of the iframe so that the content is well fitted into the region.
