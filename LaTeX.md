---
title: LaTeX
lastmod: 2023-08-14T19:16:14-05:00
---
# LaTeX
## Installing
Use Tex Workshop extension with VS Code. You'll need to install the full TexLive to get the Perl script that makes things easy. It is recommended to NOT use apt to install TeX Live as it is not up to date. But if you're lazy or going simple, it will work fine.
```bash
sudo apt install texlive-full
```
If you want to install it correctly, you need to use `tlmgr`. More reading can be done [here](https://tug.org/texlive/quickinstall.html).
## Tricks
You can set the font to Times New Roman with
```tex
\renewcommand{\familydefault}{ptm}
```
I wouldn't recommend it; Times was designed for tight spaces. The default, called Computer Modern, is better. Garamond is another nice choice. For sans-serif, Helvetica is popular.
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

```