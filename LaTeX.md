---
title: LaTeX
lastmod: 2024-01-08T19:08:50-06:00
---
# LaTeX
## Resources
The [texfaq](https://texfaq.org/)!
## Installing
Use Tex Workshop extension with VS Code. You'll need to install the full TexLive to get the Perl script that makes things easy. It is recommended to NOT use apt to install TeX Live as it is not up to date. But if you're lazy or going simple, it will work fine.
```bash
sudo apt install texlive-full
```
If you want to install it correctly, you need to use `tlmgr`. More reading can be done [here](https://tug.org/texlive/quickinstall.html).
## Tricks
### Font
You can set the font to Times New Roman with
```tex
\renewcommand{\familydefault}{ptm}
```
I wouldn't recommend it; Times was designed for tight spaces. The default, called Computer Modern, is better. Garamond is another nice choice. For sans-serif, Helvetica is popular.
### Package Installing
[Here's](https://tex.stackexchange.com/questions/73016/how-do-i-install-an-individual-package-on-a-linux-system) a good guide. If installing manually (a `tds.zip` file called a TDS or "Tex Directory Structure" file), you need to find your install location:
```bash
# system-wide
kpsewhich -var-value TEXMFLOCAL
# your own user access only
kpsewhich -var-value TEXMFHOME
```

Then, you unzip the file to it's proper location.

```bash
sudo unzip -o texdef.tds.zip -d `kpsexpand '$TEXMFLOCAL'`
```

The resulting file location will look something like `/usr/local/share/texmf/tex/generic/<package>/<package>.tex`

Finally, you must update the indices for all available packages using the following command

```bash
sudo texhash
```

If the package contains a font, the map must be updated

```
sudo updmap --sys --enable Map new-font.map
```

## Templates
### Formal Letter
```tex
\documentclass[11pt]{letter}
\usepackage[left=1in, right=1in, top=1in, bottom=1in]{geometry}
\longindentation=25em

\signature{Rick Gray \\ Member}
\address{123 MyAddress \\ Houston, TX \\ 77777}
\begin{document}

\begin{letter}{Friend \\ 123 Main \\ Houston, TX \\ 77777}
    \opening{Dear Friend,}

    How are you?

    \closing{Sincerely,}

\end{letter}
\end{document}
```

### Two Column Scientific Article with Abstract
```tex
\title{Title of Report}

\author{
  Gray, Rick\cite{Author1}
  \and
  Smith, John\cite{Author2}
}

\newcommand{\abstractText}{\noindent
  This is an abstract.
}

\documentclass[11pt, twocolumn]{article}
\usepackage{graphicx}
\usepackage{xurl}
\usepackage[super, comma, sort&compress]{natbib}
\usepackage{abstract}
\usepackage{amsmath}

\renewcommand{\abstractnamefont}{\normalfont\bfseries}
\renewcommand{\abstracttextfont}{\normalfont\small\itshape}

\begin{filecontents}{citations.bib}

@misc{Author1,
    author       = "Gray, Rick",
    howpublished = "\url{mailto:rickdgray@utexas.edu}"
}

@misc{Author2,
    author       = "Smith, John",
    howpublished = "\url{mailto:rickdgray@utexas.edu}"
}

\end{filecontents}

\usepackage{hyperref}
\hypersetup{colorlinks=true, urlcolor=blue, linkcolor=blue, citecolor=blue}

\begin{document}

\twocolumn[
  \begin{@twocolumnfalse}
    \maketitle
    \begin{abstract}
      \abstractText
      \newline
      \newline
    \end{abstract}
  \end{@twocolumnfalse}
]

\section{Introduction}

\begin{table*}
    \centering
    \begin{tabular}{lll}
        1 & 2 & 5 \\
        3 & 4 & 6
    \end{tabular}
    \caption{Floating table} \label{tab:hresult}
\end{table*}

\subsection{Section I}

This is an intro.

\subsubsection{Inline Table}

This is an inline table.

\begin{center}
\begin{tabular}{|l|l|}
  \hline
  1 & 2 \\ \hline
  3 & 4 \\ \hline
  5 & 6 \\ \hline
\end{tabular}
\end{center}

\section{Conclusion}

This is a conclusion.

\nocite{*}
\bibliographystyle{plain}
\bibliography{citations}

\end{document}
```