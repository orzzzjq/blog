---
title: How does this blog work?
date: 2023-11-04 15:55:39
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
