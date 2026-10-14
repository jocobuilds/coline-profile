---
title: "📖 The Lazy Polyglot's LaTeX Workflow"
date: 2026-10-14
draft: false
author: "Joachim Coline"
tags:
  - technology
  - LaTeX
  - tips
toc: false
---

Maintaining a multilingual LaTeX project can be a painful task. In this post, I present a simple trick to ease it.

<!--more-->

# The problem
Have you ever opened a LaTeX document and been asked to translate it into one or more languages? And then stared at the `.tex` file with the sinking feeling that you're about to maintain two or three copies of the same document in parallel?

If so, you know the pain. Fix a formula in the original? Now fix it in the translation. Restructure a section? Do it again. Add a figure? Duplicate the environment, translate the caption, and remember which copy you edited last. Multiply all of that by every language, and a single document quietly becomes a small maintenance project of its own.

The deeper issue is a violation of the *Don't Repeat Yourself* (DRY) principle. A LaTeX document should be a single source of truth. The moment you duplicate the file to translate it, you've created two sources of truth that will drift apart the first time you edit them. The mere existence of more than one source of truth is the root cause of all the struggles.

# Existing solutions
Translating a LaTeX project is not a new problem. A few options exist:
- The `translator` and `glossaries` packages help define a set of terms or glossary entries that render differently depending on the active `babel` language. However, they are not meant to translate a whole project--- only specific terms.
- The [po4a](https://www.po4a.org/) project is a tool that extracts translatable strings from your `.tex` file into a `.po` catalog (a simple "source text to translated text" list). A translator fills in the translations in that catalog, and running `po4a` again generates a translated `.tex` file from it. Although this approach seems promising, it still requires maintaining a separate catalog alongside the source, plus the `po4a` toolchain itself. That's more machinery than the task calls for.
- You can prompt a Large Language Model (LLM) to do the translation for you. In the short term this saves the initial effort, and the output is a decent first draft. In the long run however, maintenance will not be simple:
  - the output is still a duplicate file, so every future edit has to be made twice and kept in sync,
  - LLM outputs are not deterministic, so you cannot re-run the translation, diff two runs, or trust that a minor edit to the original text won't silently rewrite its translated counterpart.
- Maybe another option I missed?

I'm not happy with these solutions because they either
- introduce new files,
- require extra tooling beyond the LaTeX project itself, or
- only partially solve the problem.

What if we could directly translate the project inside the existing `.tex` file? 

# The method: one file, any language, zero dependencies
The idea behind the suggested method is that original and translated texts exist together (in the same file), and you get to decide which one to compile.

To reach this goal, the trick is to define a new command for each language at the very beginning of your project:
```latex
\newcommand\en[1]{} % command for English language
\newcommand\fr[1]{} % command for French language
\newcommand\nl[1]{} % command for Dutch language
% etc.
```
and then to use them wherever translation is needed, e.g.
```latex
% ...
\section{\en{Tutorial}\fr{Tutoriel}\nl{Handleiding}}
\en{Welcome!}\fr{Bienvenue!}\nl{Welkom!}
% ...
```
To compile the project in a specific language, insert `#1` inside the command of your choice, and leave the others empty, e.g.
```latex
\newcommand\en[1]{}
\newcommand\fr[1]{#1} % document will be compiled in French 
\newcommand\nl[1]{} 
```
In this example, all code inside the `\en{}` and `\nl{}` commands will be ignored by the compiler. Only the code outside these commands and inside the `\fr{}` commands will be considered. The result is a French-only output.

The same rule will apply if another language is selected.

# Tips
- Translating text requires particular attention when it comes to modifications. Although the suggested method allows to easily compare translated against original text, it does not track modifications. Hence, it can become hard to edit already translated text. For this reason, I recommend using a Version Control System (VCS) such as [git](https://git-scm.com/). Although not necessary, it will save you a lot of time, especially for large projects.
- Translate your project only when you have finished editing the original version. Otherwise, you will waste time translating text that will be modified or even removed later.
- Some LaTeX packages depend on language. You can use the same trick to import them in one language or another, e.g.
```latex
% ...
\en{
\usepackage[english]{babel}
}
\fr{
\usepackage[french]{babel}
}
\nl{
\usepackage[dutch]{babel}
}
% ...
```
Same applies for material used in figures, e.g.
```latex
% ...
\begin{figure}[h]
\centering
\includegraphics{\en{EN.png}\fr{FR.png}\nl{NL.png}}
\caption{\en{Example}\fr{Exemple}\nl{Voorbeeld}}
\label{fig:example}
\end{figure}
% ...
```
 - Code that is common to all languages does not need to be wrapped inside a language command, e.g. the `figure` environment above.

 # Future work
 To further ease the translation task, for example when adding a new language, it would be nice to have a tool to automatically generate the new language commands inside an already-translated project. Again, an LLM could probably do that, but where's the fun if we delegate it all to them? 😜

 Thanks for reading. I hope this post was helpful in one way or another.