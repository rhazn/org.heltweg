---
title: "Write an exposé for your thesis"
date: 2023-02-13
slug: "thesis-expose-project-outline"
tags: ["research"]
author: "Philip Heltweg"
description: "Why write an exposé or project outline for your thesis and what to include."
---

I ask students that want to write a thesis with me to prepare an exposé (or project outline) before we start. For the student, this ensures they have an idea of the work involved and a reasonable timeline to complete it. For me, it provides a short overview about what we agreed on and highlights important deadlines.

At a minimum, I think an exposé should be a roughly two-page PDF and include:
- A clear goal
- A structure (type, approach, table of contents)
- Work packages
- A timeline

Importantly, the exposé is not graded and it is not used for evaluation. During research, it is normal that timelines change and work packages might need to be adjusted. The contents of the exposé are just meant as estimations to have a similar vision before starting a project.

An easy way to generate this PDF is using Latex, for example with [overleaf.com](https://www.overleaf.com/). I've included some starter code and an [example exposé](/files/expose/expose_example.pdf) here. If you want to see a real life example, [my exposé](/files/expose/expose.pdf) for [my master thesis](/posts/master-thesis/) can be downloaded, too.

When I write about thesis type or structure, I am referring to them as used on our [thesis portal](https://oss.cs.fau.de/theses/).

# Example code
```tex
\documentclass{article}

\usepackage[english]{babel}
\usepackage[a4paper,top=2cm,bottom=2cm,left=2cm,right=2cm,marginparwidth=1.75cm]{geometry}

\usepackage{amsmath}
\usepackage[colorlinks=true, allcolors=blue]{hyperref}

\title{Provisional Title of Your Thesis}
\author{You \& Your Advisor}

\begin{document}
\maketitle

\section{Introduction}
The introduction describes your topic and why it is important. You can also include information about related work in case you already know about it or want to work ahead. 

The introduction section should end in a clearly formulated goal for your thesis. For a research-based thesis this will be a research question, for an engineering thesis an engineering challenge.

\section{Structure}
In a structure section, you include what type of thesis you are writing and what scientific methods you plan to use. This is also the place to propose a table of contents that makes sense to you. 

In the case of a literature review, a method reference could be Kitchenham \cite{Kitchenham2004-wb}. Or, if you are using design science, Peffers et al. \cite{Peffers2007-yv}.

A table of contents should be a simple list of headlines, e.g, for a design science thesis:

\begin{enumerate}
    \item Introduction
    \item Problem identification
    \item Objective definition
    \item Solution design
    \item Implementation
    \item Demonstration
    \item Evaluation
    \item Conclusions
\end{enumerate}

\section{Expected Work Packages}
Planning out work packages gives you a chance to break down your work into smaller tasks. Work packages do not have to follow a strict structure, but it is helpful if you assign a clear identifier to each.

Your work packages are a plan for you and are not part of an evaluation of your work, so feel free to include as much detail as you want.

\begin{itemize}
    \item \textbf{WP1:} Write an introduction
    \begin{enumerate}
        \item \textbf{WP1.1:} Search for related work
    \end{enumerate}
    \item \textbf{WP2:} Write a structure
    \item \textbf{WP3:} Write expected work packages
    \item \textbf{WP4:} Write an expected timeline
\end{itemize}

\section{Expected Timeline}
A timeline is important to be on the same page with your advisor and must include an idea of when you want to start your work, when it should be finished and mention any deadlines you have. You should plan some milestones for yourself to check your progress periodically. This section, like the work packages, is for you and does not influence your grading.

For bonus points, include a Gantt chart referencing the work packages you previously defined. Consider using a text-based tool to generate your timeline diagram, for example MermaidJS\footnote{https://mermaid.js.org/syntax/gantt.html} for Markdown or pgfgantt\footnote{https://www.ctan.org/pkg/pgfgantt} for LateX. 

\begin{filecontents}{bibliography.bib}
    @ARTICLE{Peffers2007-yv,
      title     = "A Design Science Research Methodology for Information Systems
                   Research",
      author    = "Peffers, Ken and Tuunanen, Tuure and Rothenberger, Marcus A and
                   Chatterjee, Samir",
      journal   = "Journal of Management Information Systems",
      publisher = "Routledge",
      volume    =  24,
      number    =  3,
      pages     = "45--77",
      month     =  dec,
      year      =  2007,
      issn      = "0742-1222",
      doi       = "10.2753/MIS0742-1222240302"
    }
    @ARTICLE{Kitchenham2004-wb,
      title     = "Procedures for performing systematic reviews",
      author    = "Kitchenham, Barbara",
      journal   = "Keele, UK, Keele University",
      publisher = "elizabete.com.br",
      volume    =  33,
      number    =  2004,
      pages     = "1--26",
      year      =  2004
    }
\end{filecontents}

\bibliographystyle{plain}
\bibliography{bibliography}

\end{document}
```

# Downloads
[Example Exposé](/files/expose/expose_example.pdf)

[My Exposé](/files/expose/expose.pdf)

{{< aboutme >}}