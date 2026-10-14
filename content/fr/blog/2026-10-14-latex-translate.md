---
title: "📖 Le Workflow LaTeX du Polyglotte Paresseux"
date: 2026-10-14
draft: false
author: "Joachim Coline"
tags:
  - technologie
  - LaTeX
  - conseils
toc: false
---

Maintenir un projet LaTeX dans plusieurs langues peut être une corvée. Dans cet article, je présente une simple astuce pour vous faciliter la tâche.

<!--more-->

# Le problème
Vous est-il déjà arrivé d'ouvrir un document LaTeX et de devoir le traduire dans une ou plusieurs langues? Puis de fixer le fichier `.tex` avec le sentiment désagréable que vous allez devoir gérer deux ou trois versions du même document en parallèle?

Si c'est le cas, vous connaissez la douleur. Une formule est modifiée dans le texte source? Il faut maintenant la corriger dans la traduction. Une section est restructurée? Il faut recommencer. Une illustration est ajoutée? Il faut dupliquer l'environnement, traduire la légende et ne pas oublier quelle version était modifiée en dernier. Multipliez tout cela par chaque langue, et un simple document se transforme peu à peu en un véritable projet de maintenance.

Le problème sous-jacent réside dans la violation du principe "Don't Repeat Yourself" (DRY). Un document LaTeX doit constituer une source unique de vérité. Dès que le fichier est dupliqué pour le traduire, deux sources de vérité sont créées et finiront par diverger dès la première modification. L'existence même de plusieurs sources de vérité est à l'origine de toutes les difficultés.

# Solutions existantes
La traduction d’un projet LaTeX n’est pas un problème nouveau. Il existe plusieurs options:
- Les packages `translator` et `glossaries` permettent de définir un ensemble de termes ou d’entrées de glossaire dont l’affichage varie en fonction de la langue `babel` active. Cependant, ils ne sont pas destinés à traduire un projet dans son intégralité, mais uniquement des termes spécifiques.
- Le projet [po4a](https://www.po4a.org/) est un outil qui extrait les chaînes traduisibles de votre fichier `.tex` pour les placer dans un catalogue `.po` (une simple liste « texte source - texte traduit »). Un traducteur remplit ce catalogue avec les traductions, puis une nouvelle exécution de `po4a` génère un fichier `.tex` traduit. Bien que cette approche semble prometteuse, elle nécessite tout de même de gérer un catalogue séparé en plus du fichier source, ainsi que la chaîne d'outils `po4a` elle-même. C'est une mise en oeuvre fort complexe par rapport à la tâche réelle.
- Vous pouvez demander à un Large Language Model (LLM) d'effectuer la traduction à votre place. À court terme, cela permet d’économiser l’effort initial, et le résultat obtenu constitue un premier jet tout à fait correct. À long terme cependant, la gestion ne sera pas simple :
  - le résultat reste un fichier dupliqué, ce qui signifie que chaque modification future devra être effectuée deux fois et synchronisée,
  - les résultats des LLM ne sont pas déterministes ; vous ne pouvez donc pas relancer la traduction, comparer deux exécutions, ni vous assurer qu'une modification mineure en anglais ne réécrira pas silencieusement sa contrepartie traduite.
- Peut-être y a-t-il une solution existante que j'aurais ratée?


Je ne suis pas satisfait de ces différentes options car soit
- elles introduisent de nouveaux fichiers,
- elles nécessitent des outils supplémentaires (en plus du projet LaTeX lui-même), ou alors
- elles ne résolvent que partiellement le problème.

Pourquoi ne pas traduire directement le projet à l'intérieur du fichier `.tex` existant? 

# La méthode: un seul fichier, n'importe quelle langue, aucune dépendance
Le principe de la méthode proposée est que les textes originaux et traduits coexistent (dans le même fichier), et que c'est vous qui décidez lequel compiler.

Pour y parvenir, l'astuce consiste à définir une nouvelle commande pour chaque langue dès le tout début de votre projet:
```latex
\newcommand\en[1]{} % commande pour l'anglais
\newcommand\fr[1]{} % commande pour le français
\newcommand\nl[1]{} % commande pour le néerlandais
% etc.
```
puis de les utiliser partout où une traduction est nécessaire, par exemple :
```latex
% ...
\section{\en{Tutorial}\fr{Tutoriel}\nl{Handleiding}}
\en{Welcome!}\fr{Bienvenue!}\nl{Welkom!}
% ...
```
Pour compiler le projet dans une langue spécifique, insérez `#1` dans la commande de votre choix et laissez les autres champs vides, par exemple:
```latex
\newcommand\en[1]{}
\newcommand\fr[1]{#1} % le projet sera compilé en français
\newcommand\nl[1]{} 
```
Dans cet exemple, tout le code situé dans les commandes `\en{}` et `\nl{}` sera ignoré par le compilateur. Seul le code situé en dehors de ces commandes et dans les commandes `\fr{}` sera pris en compte. Le résultat sera un document entièrement en français.

La même règle s'appliquera si une autre langue est sélectionnée.

# Conseils
- La traduction de textes nécessite une attention particulière en matière de modifications. Bien que la méthode proposée permette de comparer facilement le texte traduit et le texte original, elle ne permet pas de suivre les modifications. Il peut donc s'avérer difficile de modifier un texte déjà traduit. C'est pourquoi je recommande d'utiliser un système de contrôle de version (VCS) tel que [git](https://git-scm.com/). Bien que cela ne soit pas indispensable, cela vous fera gagner beaucoup de temps, en particulier pour les gros projets.
- Ne traduisez votre projet qu'une fois que vous avez terminé de modifier la version originale. Sinon, vous perdrez du temps à traduire du texte qui sera modifié, voire supprimé par la suite.
- Certains packages LaTeX dépendent de la langue. Vous pouvez utiliser la même astuce pour les importer dans une langue ou une autre, par exemple:
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
La même astuce s'applique pour les ressources utilisées dans les figures, par exemple:
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
- Du code qui est commun à toutes les langues n'a pas besoin d'être enveloppé dans une commande de langue, comme par exemple l'environnement `figure` ci-dessus.

 # Travaux futurs
 Pour faciliter encore davantage le travail de traduction, par exemple lors de l'ajout d'une nouvelle langue, ce serait bien de disposer d'un outil permettant de générer automatiquement les nouvelles commandes au sein d'un projet déjà traduit. Un LLM pourrait sans doute s'en charger, mais où est le plaisir si on leur confie tout ? 😜

Merci d'avoir lu. J'espère que cet article vous aura été utile.
