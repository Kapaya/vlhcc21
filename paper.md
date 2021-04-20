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

Many websites on the internet do not meet the exact needs of all of their users. End-user web customization systems like Thresher [@hogue2005], Sifter [@huynh2006] and Vegemite [@lin2009] help users to tweak and adapt websites to fit their unique requirements, ranging from reorganizing or annotating content on the website to automating common tasks. Millions of people also use tools like Greasemonkey [@zotero-224] and Tampermonkey [@zotero-191] to install browser userscripts, snippets of Javascript code which customize the behavior of websites.

We previously developed an alternative approach known as data-driven web customization [@litt2020; @litt2020b]. It enables web customization without traditional programming through direct manipulation of a spreadsheet-like table, right within the context of a browser. In this paradigm, a table is added to a website that contains its underlying structured data and is bidirectionally synchronized with it. Changes to the table, including sorting, filtering, adding columns and running computations in a spreadsheet formula language are propagated to the website thereby customizing it.

While end-user friendly, data-driven web customization suffers from a divide between generating the table and using it to perform customizations: the table is generated through web scraping code, referred to as *website adapters*, written by programmers but used via direct manipulation by end-users. This divide limits the agency of end-users because they rely on programmers to write the required web scraping code before being able to customize a website.

To bridge this divide, we start by harnessing programming-by-demonstration to achieve end-user web scraping within the context of the table used for customizations. This is achieved via a technique called *wrapper induction* which we discuss in [@sec:implementation]. As in prior approaches [@chasins2018; @le2014], users demonstrate examples of column values and the system uses the demonstration to synthesize a program that generalizes to all the available column values. However, this is not effective as it reduces the expressiveness of web scraping for programmers for when demonstrating is not sufficient and hides the underlying program synthesized to perform the web scraping, thus preventing modifications to be made to it. This friction between designing for both end-users and programmers is a general problem with software interfaces as discussed by Chugh in his position paper [@chugh2016a].

To fully bridge the divide, we build on several key design goals we discuss in [@sec:design-principles] to evolve end-user web scraping: users scrape by demonstrating values on the website and the system presents a formula, in the aforementioned spreadsheet formula language, corresponding to the synthesized web scraping program as a Cascading Style Sheet (CSS) selector. This is important because it enables programmers to view the synthesized CSS selector, modify it and even write one from scratch, thereby achieving the seemingly opposing goals of empowering end-users without disempowering programmers. Because the web scraping process generates formulas in the same format as formulas used for customization, we can fuse the two into a unified model for web scraping and customization in which users can interleave scraping and customizing. This results in a more incremental approach to building web customizations as we show in [@sec:examples].

To test the viability of our unified model for web scraping and customization, we implement it as an extension of Wildcard, a browser extension that implements data-driven web customization. Our contributions are:

- A **unified model for web scraping and customization** that enables end-users to perform the task of web scraping via demonstrations and programmers to view and modify the web scraping program synthesized from demonstrations, both within the context of the table used for customization
- A novel combination of design principles (**Unified User Model**, **Functional Reactive Programming** and **Mixed-Initiative Interaction**) which make our unified model for web scraping and customization seamless for both end-users and programmers
- An example gallery of websites that can be customized via our unified model for web scraping and customization and a preliminary user study providing some qualitative results of using it, both in [@sec:evaluation]


# Motivating Example {#sec:examples}


# System Implementation {#sec:implementation}

We implemented our unified model for web scraping and customization as an extension to Wildcard [@litt2020; @litt2020b]. Prior to this work, website adapters (web scraping code) were manually written in Javascript by programmers. Now, adapters can be created via demonstration and are represented in a spreadsheet formula language which can be viewed and modified. We start by describing our implementation of *wrapper induction* [@kushmerick2000] which is what enables our system to generalize from the demonstration of a single column value to the entire set of column values. Then, we describe how our system synthesizes CSS selectors which select the elements to be scrapped. Finally, we briefly discuss how the unified user model is achieved and close with notable limitations of our current approach.

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

The algorithm stops traversing upwards once it reaches the `BODY` element. It chooses the element with the largest positive value of $el_{siblings}$ as the row element, preferring nodes lower in the tree as a tiebreaker. It then generates a _row selector_ which returns the row element and all its direct siblings. The final value of `selector` is the column selector since traverses from the row element to the demonstrated data value. These row and column selectors are then used to generate an adapter as a combination of formulas which returns the DOM elements corresponding to a data row in the table and sets up the bidirectional synchronization.

## CSS Selector Synthesis Algorithm

As part of the the wrapper induction process described in the previous section, our system synthesizes two types of CSS selectors: a single row selector that selects a set of DOM elements corresponding to individual rows of the table and a column selector for each column which selects the element containing the column value within a given row.

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

One aspect of future work is saving the list of all valid selectors, instead of just picking one, and making them available to users to see and pick from. This would be similar to Mayer et el's user interaction model called *Program Navigation* [@mayer2015] that gives users the opportunity to navigate all valid, synthesized programs and pick the best one.

## Web Scraping Formulas

The wrapper induction algorithm described above results in the generation of a website adapter which consists of a combination of web scraping formulas in Wildcard's spreadsheet formula language. The language is similar to visual database query systems like SIEUFERD [@bakke2016] and Airtable [@zotero-228]. Formulas automatically apply across an entire column of data and reference other column names instead of values in specific rows. This is more efficient than users having to copy a formula across a column as in traditional spreadsheets like Microsoft Excel and Google Sheets. It of course comes at the cost of not being able to specify an operation for only a subset of columns but this hasn't yet come up in our use cases.

Web scraping formulas for rows have the following form:

`=QuerySelector(<selector>)`

The parameter `<selector>` refers to a CSS selector. Our CSS selector synthesis algorithm generate class-based selectors, falling back to index-based selectors (which utilize the `nth-child` notation) if the given elements do not have a `class` attribute. The name of the formula matches the `querySelector` method avaliable on DOM elements and should thus be familiar to programmers that write web scraping code in Javascript. The web scraping formula for rows is used in a hidden column of the table named `rowElement` for which each column cell corresponds to the row element representing the table row. The column is hidden as it serves as a reference for web scraping formulas for columns that contain the actual table data. We are exploring whether there would be value in making it visible which starts from figuring out how to reprsent a DOM element as a text value in a table cell.

Web scraping formulas for columns have the following  form:

`=QuerySelector(rowElement, <selector>)`

The parameter `rowElement` is a reference to the hidden column containing row elements and `<selector>` is a CSS selector as previously described. This form is consistent with Wildcard's existing customization formulas which reference column names to perform operations on each of the cells in the column. The Javascript equivalent is:

`rowElement.querySelector(<selector>)`

Again, should be familiar to programmers that write web scraping code once they understand how the formula language works.

Representing web scraping code as web scraping formulas in the table allows programmers to not only view them to understand the outcome of the wrapper induction algorithm but also to modify them. Modifying a web scraping formula is as simply as editing the synthesized selector and executing the formula. Furthermore, programmers can manually author web scraping formulas in empty columns in a simpler fashion than the equivalent Javascript: all they have to do is determine the selectors of the desired data and the system will take care of iterating through rows and extracting values from the selected elements.

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

*Generalization Limitation 1* shows a case where the data is displayed in a grouped structure. Without the constraint that row elements have to be direct siblings, the row generalization algorithm could determine the row selector to be *.avenger* (elements with blue border) because it matches the largest number of parallel structures (has the largest $el_{siblings}$). While this may be the correct result for the task of extraction, it is not necessarily suitable for the task of customization. When the user sorts and filters, this could result in rows moving between the two tables, disrupting the nested layout and producing a confusing result. Because of this, our system currently does not support such layouts. In the future, we may explore the possibility of extracting multiple tables from a website and joining them together.

*Generalization Limitation 2*, also in @fig:limitations, shows a case where the website contains one table of data in which rows are made up of alternating `H1` and `SPAN` tags (elements within blue border). This poses a challenge because each row does not correspond directly to a single DOM element; instead, each row consists of multiple consecutive DOM elements without any grouped structure. Moving the rows when customizing the website would require treating multiple consecutive elements as a single row. This is supported in the underlying Wildcard system, but not yet by our demonstration-based approach.

### CSS Selector Synthesis Algorithm

Our CSS selector synthesis algorithm guarantee the synthesis of a valid class-based selector if it is present because of its exhaustive nature. However, it does not guarantee that the synthesized selector is robust. Take column elements corresponding to a column of movie titles whose `class` attribute has a  value of “column-1  movie-title”. Our algorithm would synthesize “column-1” as the column selector because it would be the first class in the generated class combination list and it selects all the column elements. An inexperienced programmer writing code to scrape the column elements could also use “column-1” for similar reasons. However, an experienced programmer would likely know to use “movie-title” because it describes the meaning of the element as opposed to its ordering which could change. In general, generating robust selectors is a non trivial task [@furche2016] which is made even harder by the ambigous nature of demonstrations as specifications for program synthesis [@peleg2018].

Another shortcoming of our CSS synthesis algorithm is their reliance on the `class` attribute. CSS selectors can comprise various combinations of an element’s full set of attributes. For example, a desired set of link elements could not have a `class` attribute but could be selected based on the value of their `href` attribute using a selector like `a[href*=”/id”]`. This would select a link element whose `href` attribute ends with “/id” for example. Again, this is a non trivial task. Rousillon [@chasins2018], an end-user web scraping system that also utilizes programming-by-demonstration, uses Ringer [@barman2016] to select elements by saving all of their attributes during scraping and comparing them to candidate elements during selection. While this is more robust because it considers all of an element's attributes, this does not fit our approach of representing selectors as formulas that can be viewed and modified by programmers.

To handle the cases in which elements do not have a `class` attribute, our algorithm generate index-based selectors using the `nth-child` notation. This accurately select the given element but is not robust as the addition or removal of an element in the DOM could invalidate the selector.

# Design Principles {#sec:design-principles}

## Unified User Model

Prior to this work, web scraping and customization in Wildcard [@litt2020; @litt2020b] were divided: web scraping was done by programmers in an Integrated Development Environment (IDE) while customizing (via direct manipulation and formulas) was done in the browser. This type of divide between tasks can be seen in other domains:

- In data science, workflows revolve between cleaning and using data which often happen in different environments (e.g. data wrangling tools and live notebooks). Wrex [@drosos2020], an end-user programming-by-example system for data wrangling, reported that “although data scientists were aware of and appreciated the productivity benefits of existing data wrangling tools, having to leave their native notebook environment to perform wrangling limited the usefulness of these tools.” This was a major reason Wrex was developed as an add-on to Jupyter notebooks, the environment in which data scientists use their data.
- In web scraping, users have to switch from the environment in which they are using the scraped data (database, spreadsheet etc) to the environment in which the data is scraped if they need more of it or need to fix omissions. This can be seen in tools like Rousillon [@chasins2018], FlashExtract [@le2014] and import.io [@import.io].

Based on this, we have provided support for end-user web scraping in Wildcard in a uniform environment via the table used for customization. This relates to the idea of “in-place toolchains” [@zotero-165] for end-user programming systems: users should be able to program using familiar tools (spreadsheet table) in the context where they already use their software (browsers).

In spite of this, web scraping and customization are still divided: web scraping has to be performed prior to customization in a separate phase. Early user tests revealed that this discontinuity between the two phases was a source of confusion. Vegemite [@lin2009], a system for end-user programming of mashups, reported similar findings from its user study in which participants thought that “it was confusing to use one technique to create the initial table, and another technique to add information to a new column”.

Armed with this, we go beyond providing a uniform environment for web scraping and customization to providing a *unified user model* for web scraping and customization. Both web scraping and customization are performed in the same, single phase, with users being able to seamlessly interleave the two as desired. We can see this in the Ebay example in [@sec:examples]: the user starts out by demonstrating to scrape values on the website into a column in the table, proceeds to populate the next column with the results of a formula, sorts the table to view the resulting customization and then continues on to the next task.

## Functional Reactive Programming

In general terms, functional reactive programming (FRP) is the combination of functional and reactive programming: it is functional in that it uses functions to define programs that manipulate data and reactive in that it defines data flows through which changes in data are propagated.

FRP has seen wide adoption in end-user programming through implementations such as spreadsheet formula languages (Microsoft Excel & Google Sheets) and formula languages for low-code programming environments (Microsoft Power Fx [@zotero-150], Google AppSheet [@zotero-218], Airtable [@zotero-228], Glide [@zotero-148], Coda [@zotero-155] & Gneiss [@chang2014]).

Because of this, Wildcard [@litt2020; @litt2020b] already provides functional reactive programming via a spreadsheet formula language aimed at increasing the expressiveness of customizations. The language provides formulas to encapsulate logic, perform operations on strings, call browser APIs and even invoke web APIs. As per the FRP paradigm, users only have to think in terms of operations on the data in the table without having to worry about traditional programming concepts such as variables, state and data flow. This makes it easier for them to program customizations in a declarative manner without having to worry about all the steps that have to take place to make this possible.

Our unified model for web scraping and customizations extends this formula language to mitigate the limitations of programming-by-demonstration to specify complex web scraping tasks. Demonstrations are represented as formulas containing the corresponding, synthesized web scraping code as a CSS selector. As with other formulas in the language, the synthesized web scraping formulas can be modified (or authored from scratch) and run to achieve more expressive web scraping. We can see this in the Ebay example in [@sec:examples] when the user manually authors a formula to scrape the value of the column of ratings. Because of FRP, all the user has to do is provide is a CSS selector that will select the desired elements: the row iteration and extraction of values from the elements is done automatically for them.

## Mixed-Initiative Interaction

In a position paper [@chugh2016a], Chugh discusses how programmatic and direct manipulation systems each have distinct strengths but users are often forced to choose one over another. As a solution, he makes a proposal for “novel software systems that tightly couple programmatic and direct manipulation” which led to the emergence of systems like Sketch-N-Sketch [@chugh2016]. More generally, this idea relates to work on mixed-initiative interaction by Horvitz [@horvitz1999] in which he advocates for “designs that take advantage of the power of direct manipulation and potentially valuable automated reasoning.”

Mixed-initiative interaction can be seen in the following programming-by-example systems:

- Sketch-N-Sketch [@chugh2016] integrates direct manipulation and programming for the creation of Scalable Vector Graphics (SVG). Users can start out by creating a shape via programming and then switch to modifying its size or shape via direct manipulation which updates the underlying program to reflect the changes. Their central theme is that users do not have to choose between direct manipulation and programmatic systems
- Wrex [@drosos2020] takes examples of data transforms and generates readable and editable Python code. This was motivated by their formative study in which participants emphasized the need for programming-by-example systems to “produce code as an inspectable and modifiable artifact”
- Small-Step Live Programming By Example [@ferdowsifard2020] presents a paradigm in which programming-by-example is used to synthesize small parts of a user authored program instead of delegating construction of entire program
- Peleg et al [@peleg2018] outline how programming-by-example is not enough to differentiate all the possible programs and present an interaction model in which users not only provide feedback about the expected output of the program but also the program itself
- Mayer et al [@mayer2015] report that “a key impedance in adoption of PBE systems is the lack of user confidence in the correctness of the program that was synthesized by the system.” To this end, they developed a user interaction model called *Program Navigation* which allows users to navigate between all synthesized programs instead of only displaying the top-ranked one

Our unified model for web scraping and customization offers mixed-initiative interaction by presenting the result of web scraping by demonstration as a formula. This is advantageous as it not only allows users to delegate automation (via synthesis of web scraping programs) to the system via demonstrations but also keeps the interaction loop open by allowing users to view and modify the output of the demonstration. In the Ebay example in [@sec:examples], the user starts out by demonstrating to scrape, switches to manually authoring a web scraping formula when demonstrating is insufficient and then switches back to demonstrating, all in a seamless and fluid manner.

# Evaluation {#sec:evaluation}

## Example Gallery

## User Study

## Cognitive Dimensions Analysis

*GL note: I keep saying "our system" or "our tool" below, is there a better way to refer to it? Should we name it the Wildcard Scraper UI or something?*

In this section, we evaluate our system using the Cognitive Dimensions of Notation (cite), a heuristic evaluation framework that has been used to evaluate programming languages and visual programming systems (cite). When contrasting our tool with traditional scraping and other visual tools, we find particularly meaningful differences along several of the dimensions:

*Progressive evaluation.* In our tool, a user can see the intermediate results of their scraping and customization work at any point, and adjust their future actions accordingly. The table UI makes it easy to inspect the intermediate results and notice surprises like missing values.

In contrast, traditional scraping typically requires editing the code, manually re-running it, and inspecting the results in an unstructured textual format, making it harder to progressively evaluate the results. Also, many end-user scraping tools (Helena and Vegemite, cite these) require the user to demonstrate all the data extractions they want to perform before showing any information about how those demonstrations will generalize across multiple examples.^[This is an instance where Wildcard's limitation of only scraping a single page at a time proves beneficial. All the relevant data is already available on the page without further network requests, making it possible to support progressive evaluation with low latency. Other tools that support scraping across multiple pages necessarily require a slower feedback loop.]

*Premature commitment.* Many scraping tools require making a _premature commitment_ to a data schema: first, the user extracts a dataset, and then they perform downstream analysis or customization using that data. Our previous versions of Wildcard suffered from this problem: when writing code for a scraping adapter, a user would need to try to anticipate all future customizations and extract the necessary data.

Our current system instead supports extracting data _on demand_. The user can decide on their desired customizations and data transformations, and extract data as needed to fulfill those tasks. There is never a need to eagerly guess what data will be needed in advance.

We have also borrowed a technique from spreadsheets for avoiding premature commitment: default naming. New data columns are automatically assigned a single-letter name, so that the user does not need to prematurely think of a name before extracting the data. (We have not yet implemented the capability to optionally rename demonstrated columns, but it would be straightforward to do so, and would provide a way to offer the benefits of names without requiring a premature commitment.)

*Provisionality.* Our tool makes it easy to try out a scraping action without fully committing to it. When the user hovers over any element in the page, they see a preview of how that data would be entered into the table, and then they can click if they'd like to proceed. This makes it feel very fast and lightweight to try scraping different elements on the page.

*Viscosity.* Some scraping tools have high viscosity: they make it difficult to change a single part of a scraping specification without modifying the global structure. For example, in Helena (cite), changing the desired demonstration for a single column of a table requires re-demonstrating all of the columns (todo: make 100% sure this is true). In contrast, our system allows a user to change the specification for a single column of a table without modifying the others, resulting in a lower viscosity in response to changes.

*Role-expressiveness.* One dimension we are still exploring in our tool is role-expressiveness: having different elements of a program clearly indicate their role to the user. In particular, in our current design, the visual display of references to DOM elements in the table is similar to the display of primitive values. In our experience, this can sometimes make it difficult to understand which parts of the table are directly interacting with the page, vs. processing the downstream results. In the future we could consider adding more visual differentiation to help users understand the role of different parts of their customization.

# Related Work {#sec:related-work}

Our unified model for web scraping and customization relates to existing work in end-user web scraping, end-user web customization and program synthesis by a number of systems and tools.

## End-user Web Scraping

FlashExtract [@le2014] is a programming-by-example tool for data extraction. Like that of our unified model, FlashExtract's interface provides immediate visual feedback about demonstrations and scrapes the selected values into a table. It supports scraping substrings of column values which our model only supports through formulas. While it has desirable features we aim to emulate, it does not align with our goal to provide a unified model for web scraping and customization which goes beyond data extraction to setup a bidirectional connection between the scraped data and a website for the purpose of customization.

Rousillon [@chasins2018] is a tool that enables end-users to scrape distributed, hierarchical web data. Because demonstrations can span across several websites and involve complex data access automation tasks, its interface does not provide *full* immediate feedback about its generalizations until all the demonstrations have been made and the synthesized program has been run. Our system doesn't have this limitation because it only supprts scraping on a single website. Rousillon presents the web scraping code generated by demonstration as an editable, high-level, block-based language called Helena [@zotero-179]. While Helena can be used to perform more complex editing tasks like adding control flow, it does not display the synthesized web scraping program which means that it can only scrape what can be demonstrated. Our system on the other hand displays the synthesized program as a formula which can be modified.

## End-user Web Customization

Vegemite [@lin2009] is a tool for end-user programming of mashups that harnesses web scraping for web automation. Unlike our model, its table interface is only populated and can only be interacted with after all the demonstrations have been provided which does not support interleaving of scraping and table operaions to achieve an incremental workflow for users. Its web automation utilizes CoScripter [@leshed2008] which is used to record operations on the scraped values in the table for automation tasks. CoScripter provides the generated automation program as text-based commands, such as “paste address into ‘Walk Score’ input”, which can be edited via “sloppy programming” [@lin2009] techniques. However, this editing does not extend to the synthesized web scraping program which is not displayed and therfore cannot be modified.

Sifter [@huynh2006] is a tool that augments websites with advanced sorting and filtering functionality. Similar to Wildcard, it uses web scraping to extract data from websites in order to enable customizations. However, Wildcard supports a broader range of customizations beyond sorting and filtering, including adding annotations to websites and running computations with a spreadsheet formula language. Sifter attempts to automatically detect items and fields on the page with a variety of clever heuristics, including automatically detecting link tags and considering the size of elements on the page. If this falls, it gives the user the option of demonstrating to correct the result if the heuristics. In contrast, our model is simpler and makes fewer assumptions about the structure of websites by giving control to the user from the beginning of the process and display the synthesized program which can be modified. We hypothesize that focusing on a tight feedback loop rather than automation may support a scraping process that is just as fast as an automated one, offers more expressive scraping and extends to a greater variety of websites. However, further user testing is required to actually validate this hypothesis.

## Program Synthesis

FlashProg [@mayer2015] is a framework that provides program navigation and disambuguation for programming-by-example tools like FlashExtract [@le2014]. The program viewer provides a high level description of synthesized programs as well as a way to navigate the list of alternative programs that satisfy the demonstrations. This is important because demonstrations are an ambigous specification for program synthesis [@peleg2018] and thefore the set of synthesized programs can be large. To further ensure that the best synthesized program is arrived at, FlashProg has a disambiguation viewer that asks the user questions in order to resolve ambiguities in the user's demonstrations. In constrast, our model only presents the top-ranked synthesized program which may not be the best one. Furthermore, the program is presented in is low-level form as a formula which may not be end-user friendly but allows for more expressiveness because it can be directly modified by programmers.


# Conclusion And Future Work {#sec:conclusion}





