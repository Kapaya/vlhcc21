---
title: "A Unified Model For Web Scraping & Customization"
author: "Kapaya Katongo, Geoffrey Litt, Kathryn Jin and Daniel Jackson"
link-citations: true
csl: templates/acm.csl
reference-section-title: References
figPrefix:
  - "Figure"
  - "Figures"
secPrefix:
    - "Section"
    - "Sections"
abstract: |
 Websites are malleable: users can run code in the browser to customize them. However, this malleability is typically only accessible to programmers with knowledge of HTML and Javascript.
---

# Introduction {#sec:introduction}

Many websites on the internet do not meet the exact needs of all of their users. End-user web customization systems like Thresher [], Sifter [] and Vegemite [] help users to tweak and adapt websites to fit their unique requirements, ranging from reorganizing or annotating content on the website to automating common tasks. Millions of people also use tools like Greasemonkey [] and Tampermonkey [] to install browser userscripts, snippets of Javascript code which customize the behavior of websites.

We previously developed an alternative approach known as data-driven web customization []. It enables web customization without traditional programming through direct manipulation of a spreadsheet-like table, right within the context of a browser. In this paradigm, a table is added to a website that contains its underlying structured data and is bidirectionally synchronized with it. Changes to the table, including sorting, filtering, adding columns and running computations in a spreadsheet-like formula language are propagated to the website thereby customizing it.

While end-user friendly, data-driven customization suffers from a divide between generating the table and using it to perform customizations: the table is generated through web scraping code, referred to as website adapters, written by programmers but used via direct manipulation by end-users. This divide limits the agency of end-users because they rely on programmers to write the required web scraping code before being able to customize. 

To bridge this divide, we start by harnessing programming-by-demonstration to achieve end-user web scraping within the context of the table used for customizations. As in prior approaches [], users demonstrate examples of column values and the system uses the demonstration to synthesize a program that generalizes to all the available column values. However, this is not sufficient as it reduces the expressiveness of web scraping for programmers for when demonstrating is not sufficient and hides the underlying program synthesized to perform the web scraping, thus preventing modifications to be made to it. This friction between designing for both end-users and programmers is a general problem with software interfaces as discussed by Chugh [].

To fully bridge the divide, we build on several key design goals we discuss in Section X to evolve end-user web scraping: users scrape by demonstrating values on the website and the system presents a formula, in the aforementioned spreadsheet-like formula language, corresponding to the synthesized web scraping program as a Cascading Style Sheet (CSS) selector. This is important because it enables programmers to view the synthesized CSS selector, modify it and even write one from scratch, thereby achieving the seemingly opposing goals of empowering end-users without disempowering programmers. Because the web scraping process generates formulas in the same format as formulas used for customization, we can fuse the two into a unified model for web scraping and customization in which users can interleave scraping and customizing which results in a more incremental approach to building web customizations. 

To test the viability of our unified model for web scraping and customization, we implement it as an extension of Wildcard, a browser extension that implements data-driven web customization. Our contributions are:

- A unified model for web scraping and customization that enablers end-users to perform the task of web scraping via demonstrations and programmers to view and modify the web scraping program synthesized from demonstrations, both within the context of the table used for customization
- A novel combination of design goals (Unified User Model, Functional Reactive Programming and Mixed-Initiative Interaction) which make our unified model for web scraping and customization seamless for both end-users and programmers
- An example gallery of websites that can be customized via our unified model for web scraping and customization and a preliminary user study providing some qualitative results of using it



# Motivating Example {#sec:demos}


# System Implementation {#sec:implementation}

We implemented our unified model for web scraping and customization as an extension to Wildcard []. Prior to this work, web scraping code (website adapters) were manually coded in Javascript by programmers. Now, adapters can be created via demonstration and are represented in a spreadsheet-like formula language which can be viewed and modified. We start by describing our implementation of *wrapper induction*  [@kushmerick2000] which is what enables our system to generalize from the demonstration of a single column value to the entire set of column values. Then, we describe how our system synthesizes CSS selectors which select the elements to be scrapped. Finally, we briefly discuss how the unified user model is achieved and close with notable limitations of our current approach.

## Wrapper Induction Algorithm

In order to generate reusable scrapers from user demonstrations, our system solves the wrapper induction task: generalizing from a small set of user-provided examples to a scraping specification that will work on other parts of the website, and on future versions of the website.

We take an approach similar to that used in other tools like Vegemite [@lin2009] and Sifter [@huynh2006]:

- We synthesize a single *row selector* for the website: a CSS selector that returns a set of Document Object Model (DOM) elements corresponding to individual rows of the table.
- For each column in the table, we synthesize a *column selector*, a CSS selector that returns the element containing the column value within that row.

One important difference is that our algorithm only accepts row elements that have direct siblings with a similar structure. We refer to this as the *row-sibling* constraint. Later, we describe how the constraint provides a useful simplification of the wrapper induction task and discuss the resulting limitations this puts on our system.

When a user first demonstrates a column value, the generalization algorithm is responsible for turning the demonstration into a row selector that will correctly identify all the row elements in the website and a column selector that will correctly identify the element that contains the column value within a row element. During subsequent demonstrations, the generalization algorithm uses the generated row selector to find the row element that contains the column value and generates a column selector which identifies the corresponding column element.

At a high level, the wrapper induction algorithm’s challenge is to traverse far enough up in the DOM tree from the demonstrated element to find the element which corresponds to the row. We solve this using a heuristic; the basic intuition is to find a large set of elements with similar parallel structure. Consider the sample HTML layout in @fig:algorithm, which displays a truncated table of superheroes, with each row containing some nested structure:

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/algorithm.png}
  \caption{\label{fig:algorithm}Our system applies a heuristic to identify DOM elements that correspond to rows in the data table.}
\end{figure*}
</div>

The user performs a demonstration by clicking on element $a$ in @fig:algorithm containing “Tony Stark”. Our algorithm traverses upwards from the demonstrated element, considering each successive parent element ($b1$, $c1$ and $d$ in @fig:algorithm) as a potential candidate for the row element. For each parent element `el`, the process is as follows:

1. Compute a column selector `selector` that, when executed on `el`, only returns the demonstrated element
2. For each sibling `el'` of `el`, execute `selector` on `el'` and record whether the selector returns an element. If it does, this suggests that `el'` has some parallel structure to `el`.
3. Compute $el_{siblings}$, the number of sibling elements of `el` which have parallel structure.

Notice how the *row-sibling* constraint simplifies the problem. Row candidates without siblings with parallel structure ($b1$ in @fig:algorithm) have $el_{siblings}$ = 0, thus disqualifying them.

The algorithm stops traversing upwards once it reaches the `BODY` element. It chooses the element with the largest positive value of $el_{siblings}$ as the row element, preferring nodes lower in the tree as a tiebreaker. It then generates a _row selector_ which returns the row element and all its direct siblings. The final value of `selector` is the column selector since traverses from the row element to the demonstrated data value. These row and column selectors are then used to generate a scraping adapter as a combination of formulas which returns the DOM elements corresponding to a data row in the table and sets up the bidirectional synchronization.


## CSS Selector Synthesis Algorithms

As part of the the wrapper induction process described in the previous section, our system synthesizes two types of CSS selectors: a single row selector that selects a set of DOM elements corresponding to individual rows of the table and a column selector for each column which only selects the element containing the column value within a given row.

For a given row element, it’s row selector is synthesized as follows:

1. Generate a list of all possible combinations of the classes on the element’s `class` attribute and initialize their scores to 0. For example, an element with a `class` attribute value of “a b c” would generate “a”, “b”, “c”, “a b”, “b c” and  “a b c”
2. Retrieve a list of all of the sibling elements of the element. For each sibling element, check whether it’s `class` attribute contains each of the generated class combinations. If a sibling’s `class` attirbute contains a given class combination, the combination’s score is incremented by 1
3. Pick the class combinations with the highest score across the element’s siblings and then select the combination with the fewest number of classes. For example, if the combinations with the highest score are “a” and “b c”, “a” will be picked. This is done to ensure that only the minimal required classes are used for selection
4. Combine the tag name of the element with the final selector to further ensure that it only selects the desired row elements. For example, if the row element is has a tag name of `DIV` and the final selector is “a” the synthesized selector will be `div.a`

For a given column element, it’s column selector is synthesized as follows:

1. Generate a list of all possible combinations of the classes on the element’s `class` attribute, as previously described, and initialize their scores to 0
2. For each class combination, check that it only selects a single element within the row element and that that element corresponds to the given column element. If a class combination satisfies the check, its score is incremented by 1
3. Pick the class combinations with the highest score and then select the combination with the fewest number of classes as previously described
4. Combine the tag name of the element with the final selector to further ensure that it only selects the desired column element as previously described


## Unified User Model


## Limitations

Since our system is still under development, it has a variety of limitations. In this section we describe the most notable ones.

### Wrapper Induction Algorithm

The row-sibling constraint we mentioned earlier is important for the end goal of customization because row elements that are not direct siblings may not represent data on the website that should be related as part of the same table by customizations such as sorting and filtering. In @fig:limitations we demonstrate two examples where this limitation becomes relevant.

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/limitations.png}
  \caption{\label{fig:limitations} Two example pages where our generalization algorithm does not currently work. The elements with the blue border correspond to rows of the data and the elements with green borders correspond to tables of data in each layout respectively. For the layout on the left, sorting could lead to rows from one table ending up in the other. For the layout on the right, sorting would lead to a distortion of the table since the column elements cannot be moved as a unit.}
\end{figure*}
</div>

<div class="html-only">
![Two example pages where our generalization algorithm does not currently work. The elements with the blue border correspond to rows of the data and the elements with green borders correspond to tables of data in each layout respectively. For the layout on the left, sorting could lead to rows from one table ending up in the other. For the layout on the right, sorting would lead to a distortion of the table since the column elements cannot be moved as a unit.](media/limitations.png){#fig:limitations}
</div>

*Generalization Limitation 1* shows a case where the data is displayed in a grouped structure. Without the constraint that row elements have to be direct siblings, the row generalization algorithm could determine the row selector to be *.avenger* (elements with blue border) because it matches the largest number of parallel structures (has the largest $el_{siblings}$). While this may be the correct result for the task of extraction, it is not necessarily suitable for the task of customization. When the user sorts and filters, this could result in rows moving between the two tables, disrupting the nested layout and producing a confusing result. Because of this, our system currently does not support such layouts. In the future, we may explore the possibility of extracting multiple tables from a website and joining them together.

*Generalization Limitation 2*, also in @fig:limitations, shows a case where the website contains one table of data in which rows are made up of alternating `H1` and `SPAN` tags (elements within blue border). This poses a challenge because each row does not correspond directly to a single DOM element; instead, each row consists of multiple consecutive DOM elements without any grouped structure. Moving the rows when customizing the website would require treating multiple consecutive elements as a single row. This is supported in the underlying Wildcard system, but not yet by our demonstration-based approach.

### CSS Selector Synthesis Algorithms

Our CSS selector synthesis algorithms guarantee the synthesis of a valid selector if it is present because of its exhaustive nature. However, it does not guarantee that the synthesized selector is robust. Take column elements corresponding to a column of movie titles whose `class` attribute has a  value of “column-1  movie-title”. Our algorithm would synthesize “column-1” as the column selector because it would be first class in the generated class combination list and it selects all the column elements. An inexperienced programmer writing code to scrape the column elements could also use “column-1” for the similar reasons. However, an experienced programmer would likely know to use “movie-title” because it describes the meaning of the element as opposed to its ordering which could change. In general, generating robust CSS selectors is a non trivial task even for humans.   

Another shortcoming of our CSS synthesis algorithms is their reliance on the `class` attribute. CSS selectors can comprise various combinations of an element’s full set of attributes. For example, a desired set of link elements could not have a `class` attribute but could be selected based on the value of their `href` attribute using a selector like `a[href*=”/stargazers”]`. This would select link elements whose `href` attribute ends with “/stargazers”. Again, this is a task that is non trivial even for humans. Rousillon [], an end-user web scraping system that also utilizes programming-by-demonstration, uses Ringer [] to select elements by saving all of their attributes during scraping and comparing them to candidate elements during selection. While this is more robust because it considers all of an element's attributes, this does not fit our approach of representing selectors as formulas that can be modified by users. To handle the cases in which elements do not have a `class` attribute, our algorithms generate an index-based selector using the `nth-child` notation which accurately select the given element but are not robust as the addition or removal of an element in the sub tree could invalidate it.


# Design Principles {#sec:design-principles}


## Unified User Model

In the first iteration of data-driven customization, web scraping and customization were divided: web scraping was done by programmers in an Integrated Development Environment (IDE) while customizing (via direct manipulation and formulas) was done in the browser. This type of divide between tasks can be seen in other domains:

- In data science, workflows revolve between cleaning and using data which often happen in different environments (e.g. data wrangling tools and live notebooks). Wrex [], an end-user programming-by-example system for data wrangling, reported that “although data scientists were aware of and appreciated the productivity benefits of existing data wrangling tools, having to leave their native notebook environment to perform wrangling limited the usefulness of these tools.” This was a major reason Wrex was developed as an add-on to Jupyter notebooks, the environment in which data scientists use their data.
- In web scraping, users have to switch from the environment in which they are using the scraped data (database, spreadsheet etc) to the environment in which the data is scraped if they need more of it or need to fix omissions. This can be seen in tools like Rousillon [], FlashExtract [], import.io [], dexi.io [], Octoparse [] and ParseHub [].

Based on this, we have provided support for end-user web scraping in data-driven web customization in a uniform environment via the table used for customization. This relates to the idea of “in-place toolchains” [] for end-user programming systems: users should be able to program using familiar tools (spreadsheet table) in the context where they already use their software (browsers).

In spite of this, web scraping and customization are still divided: web scraping has to be performed prior to customization in a separate phase. Early user tests revealed that this discontinuity between the two phases was a source of confusion. Vegemite, a system for end-user programming of mashups, reported similar findings from its user study in which participants thought that “it was confusing to use one technique to create the initial table, and another technique to add information to a new column”.

Armed with this, we go beyond providing a uniform environment for web scraping and customization to providing a unified user model for web scraping and customization. Both web scraping and customization are performed in the same, single phase, with users being able to seamlessly interleave the two as desired. We can see this in the Ebay example in Section X: the user starts out by demonstrating to scrape values on the website into a column in the table, proceeds to populate the next column with the results of a formula, sorts the table to sort view the resulting customization and then continues on to the next task.


## Functional Reactive Programming

In general terms, functional reactive programming (FRP) is the combination of functional and reactive programming: it is functional in that it uses functions to define programs that manipulate data and reactive in that it defines data flows through which changes in data are propagated.

FRP has seen wide adoption in end-user programming through implementations such as spreadsheet formula languages (Microsoft Excel & Google Sheets) and formula languages for low-code programming environments (Microsoft Power Fx, Google AppSheets, Glide, Coda & Gneiss).

Because of this, data-driven web customization already provides functional reactive programming via a spreadsheet-like formula language aimed at increasing the expressiveness of customizations. The language provides formulas to encapsulate logic, perform operations on strings, call browser APIs and even invoke web APIs. As per the FRP paradigm, users only have to think in terms of manipulating the data in the table without having to worry about traditional programming concepts such as variables and data flow. This makes it easier for them to program customizations in a declarative manner without having to think about all the steps that have to take place to make this possible. 

Our unified model for web scraping and customizations extends this formula language to mitigate the limitations of programming by demonstration. Demonstrations are represented as formulas containing the corresponding, synthesized web scraping code as a CSS selector. As with other formulas in the language, web scraping formulas can be modified (or authored from scratch) and run to achieve more expressive web scraping. We can see this in the Ebay example in Section X when the user manually authors a formula to scrape the value of XXX.

## Mixed-Initiative Interaction

In a position paper [], Chugh discusses how programmatic and direct manipulation systems each have distinct strengths but users are often forced to choose one over another. As a solution, he made a proposal for “novel software systems that tightly couple programmatic and direct manipulation” which led to the emergence of systems like Sketch-N-Sketch []. More generally, this idea relates to work on mixed-initiative interaction by Horvitz [] in which he advocates for “designs that take advantage of the power of direct manipulation and potentially valuable automated reasoning.”

Our unified model for web scraping and customization offers mixed-initiative interaction by presenting the result of web scraping by demonstration as a formula. This is advantageous as it not only allows users to delegate automation (via synthesis of web scraping programs) to the system via demonstrations but also keeps the interaction loop open by allowing users to view and modify the output of the demonstration. In the Ebay example in Section X, the user starts out by demonstrating to scrape, switches to manually authoring  a web scraping formula when demonstrating is insufficient and then switches back to demonstrating, all in a seamless and fluid manner.

This type of mixed-initiative interaction can be seen in other programming-by-example systems:

- Sketch-N-Sketch integrates direct manipulation and programming for the creation of Scalable Vector Graphics (SVG). Users can start out by creating a shape via programming and then switch to modifying its size or shape via direct manipulation which updates the underlying program to reflect the changes. Their central theme is that users do not have to choose between direct manipulation and programmatic systems
- Wrex [@drosos2020] takes examples of data transforms and generates readable and editable Python code. This was motivated by their formative study in which participants emphasized the need for programming-by-example systems to “produce code as an inspectable and modifiable artifact”
- Small-Step Live Programming By Example presents a paradigm in which programming-by-example is used to synthesize small parts of a user authored program instead of delegating construction of entire program
- Pileg et al outline how programming-by-example is not enough to differentiate all the possible programs and present an interaction model in which users not only provide feedback about the expected output of the program but also the program itself
- Mayer et al report that “a key impedance in adoption of PBE systems is the lack of user confidence in the correctness of the program that was synthesized by the system.” They describe how even though FlashFill, a programming-by-example tool for string manipulation in Excel, received many positive reviews from popular media sources, a prominent Excel user expressed caution because of the lack of insight into what the synthesized program is actually doing. To this end, they developed a under interaction model called Program Navigation which allows users to navigate between all synthesized programs instead of only displaying the top-ranked one


# Evaluation {#sec:evaluation}

## Example Gallery

## User Study

## Cognitive Dimensions Of Notation

# Related Work {#sec:related-work}

# Conclusion And Future Work {#sec:conclusion}





