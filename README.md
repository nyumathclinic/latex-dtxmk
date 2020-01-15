# latex-dtxmk

Harness latexmk to autogenerate files within a docstrip file

## Installation

You need `latexmk` on your system.  If you have TeX-live you probably already
have this.

The simplest installation method is to copy the file `dtxmk.latexmkrc` into the
folder of the projection you're working on.  

If you don't want to duplicate files, you can copy `dtxmk.latexmkrc` into the
folder `$TEXMFHOME/scripts/latexmk/perl/`  Here `$TEXMFHOME` is your user TeX
tree.  In MacTeX, it's probably `~/Library/texmf`.  In Windows, I have no idea.
On the CIMS Linux machines, it's...?  Run

    kpsewhich -var-value TEXMFHOME

to find your local TeX tree.  

## Usage

### Docstrip source file

Instead of a regular TeX file, your main source file will be a DocStrip file.
It is a TeX file with a few additions and changes.  Usually the extension of a
DocStrip file is `.dtx`.

The head of your DocStrip file will set up the `docstrip` package and call
`\generate`.  It might look like this:

    %<*driver>
    \input docstrip.tex
    \askforoverwritefalse
    \generate{
        \file{\jobname.qns.tex}{\from{\jobname.dtx}{questions}}
        \file{\jobname.ans.tex}{\from{\jobname.dtx}{questions,answers}}
        \file{\jobname.sol.tex}{\from{\jobname.dtx}{questions,solutions}}
    }
    \endbatchfile
    %</driver>

The lines marked `%<driver>` and `%</driver>` delimit the “driver” portion of
the file.  Since they begin with percent characters, TeX will normally treat
them like comments.  

The command `\askforoverwritefalse` allows overwriting of generated files
without confirmation.

The `\generate` command is given a list of `\file` commands, one for each file
you want to generate.  The first argument to each `\file` command is the name
of the file to be generated.  The second argument is the list of source files
and options to set for that generated file.

The macro `\jobname` expands to the current file name, without extension.  The
command `\endbatchfile` ends DocStrip processing.

For instance, the excerpt above is taken from a file called `hw02.dtx`, included
in this repository.  The command `tex hw02.dtx` will process `hw02.dtx` with
DocStrip.  Three files will be generated:

* a file `hw02.qns.tex`, from the file `hw02.dtx` with the `questions` option
  turned on

* a file `hw02.ans.tex`, from the file `hw02.dtx` with the `questions` and
  `answers` options turned on

* a file `hw02.sol.tex`, from the file `hw02.dtx` with the `questions` and
  `solutions` options turned on

Then you can compile `hw02.qns.tex` into `hw02.qns.pdf` and distribute it as the
“problem sheet”.  You can distribute `hw02.ans.tex` to the students as a template
for them to edit, compile, and submit.  Once the homework is collected and
graded, you can compile `hw02.sol.tex` into `hw02.sol.pdf` and distribute solutions.

The *rest* of the DocStrip file is like an interleaved set of TeX files.  Any
unmarked line of code is included in every generated file.  But we can insert
“guards” indicating code lines to be included depending on the options set.
Each guard begins with a percent sign (normally ignored by regular TeX) and
contains a boolean expression enclosed by angle brackets.

In our example homework file, a line such as

    %<solutions>The answer is $6$.

will not be included in `hw02.qns.tex`, because that file is generated without the
`solutions` option.  Options can be combined using the `&` (and), `|` (or), and
`!` operators.  `&` takes precedence over `|`, but parentheses are also allowed.
So a line such as

    %<!answers&!solutions> Hint: Use Theorem 17.3.

will be in `hw02.qns.tex` (by default) but *not* in `hw02.ans.tex` or `hw02.sol.tex`.

Now putting guards at the start of every line would be cumbersome to type.  It
might also make the source file hard to read, since an editor may color the
entire line as a comment.  So guard modifiers are used to delimit blocks of code
with the same guard.  Any expression preceded by `*` will apply the indicated
guard to every line that follows, until the identical expression is encountered
with the `/` modifier.  This gives an almost HTML-like layer to your source
file.  Here is an example from the same homework file:

    \begin{question}
        Scheinerman, Exercise 6.6:
        Disprove the statement ``If $p$ is prime, then $2^p-1$ is also prime.''
    \end{question}
    %<*!answers&!solutions>
    \begin{hint}
        All you need is one counterexample.  Guess and check, and be persistent.
    \end{hint}
    %</!answers&!solutions>
    %<*answers>
    \begin{answer}
    %% Put your answer here!
    \end{answer}
    %</answers>
    %<*solutions>
    \begin{solution}
        Let $p=11$.  Then $p$ is prime.  But $2^p-1 = 2^{11}-1 = 2047 = 23 \times 89$.
        So the statement is false.
    \end{solution}
    A prime number that is equal to $2^n-1$ for some $n$ is called a
    \href{https://en.wikipedia.org/wiki/Mersenne_prime}{\emph{Mersenne Prime}}.
    %</solutions>

With the configuration as above, the `hw02.qns.tex` file will contain a block that
looks like this:

    \begin{question}
        Scheinerman, Exercise 6.6:
        Disprove the statement ``If $p$ is prime, then $2^p-1$ is also prime.''
    \end{question}
    \begin{hint}
        All you need is one counterexample.  Guess and check, and be persistent.
    \end{hint}

Note the lines with modified guards are stripped out.  The `hw02.ans.tex` file
will contain this block instead:

    \begin{question}
        Scheinerman, Exercise 6.6:
        Disprove the statement ``If $p$ is prime, then $2^p-1$ is also prime.''
    \end{question}
    \begin{answer}
    %% Put your answer here!
    \end{answer}

Note that a comment with a single percent character would get stripped out and
discarded by DocStrip.  But a double percent gets kept.  The `hw02.sol.tex` file
will contain this block:

    \begin{question}
        Scheinerman, Exercise 6.6:
        Disprove the statement ``If $p$ is prime, then $2^p-1$ is also prime.''
    \end{question}
    \begin{solution}
        Let $p=11$.  Then $p$ is prime.  But $2^p-1 = 2^{11}-1 = 2047 = 23 \times 89$.
        So the statement is false.
    \end{solution}
    A prime number that is equal to $2^n-1$ for some $n$ is called a 
    \href{https://en.wikipedia.org/wiki/Mersenne_prime}{\emph{Mersenne Prime}}.

The implementation of the environments `question`, `answer`, `hint`, and
`solutions` have to be set up in the preambles of the generated TeX files (or in
packages used by them).  But guards can be used in the preambles too.  In this
way, we can conditionally style the document.  For instance, I prefer that the
question text be upright in the questions file and italicized in the
answers/solutions file.  This is done like so:

    \usepackage{amsthm}
    \theoremstyle{definition}
    %<answers|solutions>\theoremstyle{plain}
    \newtheorem{question}{Question}

#### Questions you might have right now

##### Why not just maintain separate files?

You probably know the answer to that.  If your question has a typo, you need to
fix it in three files.  Or if you want to copy this question to another
assignment file, you have to copy and paste from three old files to three new
files.

##### Why not just a boolean that and produces whichever variant is needed?

Some workarounds for multiple variants of a document involve one or more LaTeX
booleans.  For instance:

    \newboolean{solutions}
    \setboolean{solutions}{false}
    % \setboolean{solutions}{true}% uncomment this line to get solutions

    % ...
    
    \ifsolutions
    \begin{solution}
        Let $p=11$.  Then $p$ is prime.  But $2^p-1 = 2^{11}-1 = 2047 = 23 \times 89$.
        So the statement is false.
    \end{solution}
    \fi

If this file is called `hw02.tex`, then compiling it as shown will exclude the
solution.  Uncommenting the indicated line and recompiling it will include the
solution.

A drawback to this approach is that you don't know what options `hw02.pdf` was
compiled with.  Consider an instructor who carefully writes the solution to the
homework questions before assigning them to the students, but carelessly forgets
to exclude the solutions when posting the assignment PDF to the course website.
The DocStrip method allows one TeX file (and one PDF file) per document variant,
but also one common source file to minimize repetition.

##### Why use so many guards?  Why not configure the environments to keep or discard their contents depending on DocStrip options?

It seems the example file contains a lot of repetitive snippets like

    %<*solutions>
    \begin{solution}
        The solution
    \end{solution}
    %</solutions>

It would require less typing to conditionally define the `solutions` environment
to discard its body except in the solutions.  In the preamble, you could have

    \NewEnviron{solution}{}
    %<*solutions>
    \renewenvironment{solution}[1][Solution]
        {\begin{proof}[#1]\renewcommand\qedsymbol{$\blacktriangle$}}
        {\end{proof}}
    %</solutions>

This does not totally eliminate the need for guards in the document body.
Anything you wanted to put in the solutions that you did not want in a
`solution` environment (e.g., a post-solution note with a reference and more
information) would require it.

Also, it means the source file includes the solution in all document variants.
This is problem if you are distributing LaTeX code for students to use as a
template.  If the solution is in the source, students will be able to see it.

##### Why is there a `questions` option if it's not used in any of the example snippets?

Have a look at the full example file.  The entire LaTeX document from
`\documentclass{article}` to `\end{document}` is delimited by `%<*questions>`
and `%</questions>`.  In this simple example, it would not be necessary to use
the questions option at all.  However, if you had another file to generate which
was not one of these document variants, it could go after `%</questions>` inside
its own guard.  An example might be a `.bib` file with bibliography data.

##### Doesn't this mean I have to process two files instead of one each time I'm editing and testing?

Well, someone (or something) has to process two files.  But if you configure it
right, you can have this done automatically with one command.  For that, read on.

### Compiling via the command line

[`latexmk`](http://personal.psu.edu/~jcc8/software/latexmk/) is a script for
“making” LaTeX files.  It compiles the document, runs auxiliary programs (e.g.,
`bibtex`, `makeindex`) if necessary, and ensures that the document is “finished”
and all references are defined.  It is a perl script that runs the necessary
commands as subprocesses.

`latexmk` is extensively configurable, both with command line arguments and
configuration files.  Configuration files are simply perl code.

The `dtxmk.latexmkrc` is one of those configuration files.  It builds the set of
generated files (by looking at the log files) and makes sure each of them are
built.  All you need to do is

    latexmk -r dtxmk.latexmkrc

by default, this will use `latex` to compile the generated TeX files.  If you
would rather use `pdflatex`, add the `-pdf` option.  If you would rather use
`xelatex` or `lualatex`, use the `-pdfxe` or `-pdflua` options instead.  

If you would rather install `dtxmk.latexmkrc` in a user directory, you can.
Just replace the file name above with a (relative or absolute) path to wherever
you installed it.

For our purposes, it's important that `latexmk` be called *without* any file
arguments.  The `dtxmk.latexmkrc` config file supplies the list of documents to
be compiled by a separate process.

### Compiling in TeXShop

Coming soon.

### Compiling in [Visual Studio Code](https://code.visualstudio.com/)

Install the [LaTeX Workshop](https://github.com/James-Yu/LaTeX-Workshop)
extension.  Configure the source file to use `latexmk` with the
`dtxmk.latexmkrc` configuration file with these lines near the top of the file:

    % !TEX program = latexmk
    % !TEX options = -r dtxmk.latexmkrc -synctex=1 -interaction=nonstopmode -file-line-error -pdf

As you can probably guess, this is just a front end to the command line.  
The first option calls our config file.  If `dtxmk.latexmkrc` is not in the
current working directory, preface it with an absolute or relative path.

The last option creates PDF files with `pdflatex`.  You can change that if you
want, as above.

The other options are to help `latexmk` play nice with Visual Studio Code.  
