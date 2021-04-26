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
 Web scraping, the process of extracting structured data from a website, is a common building block of web customization systems. Prior approaches have allowed users to perform web scraping by directly demonstrating examples, but this typically doesn’t allow for as much expressiveness as traditional programming.

 In this paper, we present a new interaction model for web scraping that combines the ease of use of programming-by-demonstration and the expressiveness of traditional programming. When a user demonstrates examples of data to extract, a web scraping program is synthesized and presented as a spreadsheet formula. Crucially, the user can also directly edit the formula, allowing them to specify scraping operations which can not be achieved via demonstration alone.

 To illustrate our model, we implement it as a browser extension called Joker. Through concrete examples and a formative user study with five participants, we show how Joker enables users to scrape and customize websites more flexibly than in prior systems.

---

# Introduction {#sec:introduction}

Many websites on the internet do not meet the exact needs of all of their users. Because of this, millions of people use browser extensions like Greasemonkey [@zotero-224] and Tampermoneky [@zotero-191] to install userscripts, snippets of Javascript code which customize the behavior of websites. To make the creation of web customizations more accessible to end-users without knowledge of programming, researchers have developed systems like Sifter [@huynh2006], Vegemite [@lin2009] and Wildcard [@litt2020; @litt2020b].

A common building block of these web customization systems is web scraping, the extraction of structured data from websites. They achieve web scraping in one of two ways: programming-by-demonstration [@huynh2006; @lin2009] and traditional programming [@litt2020; @litt2020b]. Web scraping by demonstration automates the web scraping process by utilizing program synthesis to synthesize a web scraping program from user demonstrations. This approach is accessible to end-users with no programming experience and enables them to fully participate in the customization lifecycle. However, web scraping by demonstration comes at the cost of expressiveness which is vital for complex web scraping tasks. On the other side, web scraping by programming involves programmers manually writing web scraping code. It is fully expressive but not accessible to end-users who can only customize websites that programmers have written web scraping code for. This friction between designing for both ease of use and expressiveness is a general problem with software interfaces as discussed by Chugh [@chugh2016a].

In this paper, we present a new model for web scraping for customization that combines the ease of use of programming-by-demonstration and the expressiveness of traditional programming in the environment of a spreadsheet-like table with a formula language. Users demonstrate to scrape and the system presents the synthesized web scraping program as a Cascading Style Sheet (CSS) selector in the table's formula language ([@sec:implementation]). More importantly, these web scraping formulas can be directly edited to specify complex scraping operations which can not be achieved by demonstrations alone.

We have implemented this new model of web scraping as an extension of Wildcard [@litt2020; @litt2020b] which only supports web scraping by programming. Wildcard enables web customization by direct manipulation of a spreadsheet-like table it adds to websites. The table contains the website's underlying structured data and is bidirectionally synchronized with it. This means that changes to the table, including sorting, adding columns and running computations in a spreadsheet formula language, are propagated to the website thereby customizing it.

By representing the output of demonstrations as formulas, our new web scraping model fit right into Wildcard's customization paradigm which utilizes formulas for web customization. This enabled us to go beyond providing a model that combined web scraping by demonstration with web scraping by programming to providing a model that combined web scraping *and* customization. We refer to this as a *unified model for web scraping and customization*. In [@sec:examples], we show how this enables users to scrape and customize websites more flexibly than in prior systems.

Our contributions are as follows:

- A unified model for web scraping and customization that combines web scraping (by demonstration and programing) with web customization through a shared spreadsheet formula language
- An implementation of the unified model for web scraping and customization that combines key design principles in a novel way ([@sec:design-principles])
- An evaluation of the model via a formative user study that provides qualitive results of using of it, an example gallery of websites that the model works on and doesn't work on and a cognitive dimensions of notation analysis of the model ([@sec:evaluation])

We end by discussing opportunities for future work ([@sec:conclusion]).

# Motivating Example {#sec:examples}
To concretely illustrate the user experience of using a unified model of demonstrations and formulas to scrape and customize a webpage, we present a scenario of customizing eBay, a popular online marketplace. The goal of the user, Jen, is to filter out search results that are sponsored. On the webpage, sponsored product listings are marked with a "Sponsored" label that Jen can try to scrape, then she can sort the scraped labels to move all sponsored results to the bottom of the page. @fig:ebay shows accompanying screenshots.

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/ebay.png}
  \caption{\label{fig:ebay}Scraping and customizing eBay by unified demonstration and formulas.}
\end{figure*}
</div>

**Scraping Names by Demonstration** (@fig:ebay Part A): Jen starts the adapter creation process by clicking a context menu item within the eBay.com page, then hovering over the listing name data value that she would like to scrape. The system provides live feedback as Jen hovers: it annotates the row of data with a border, highlights the column of data in the page with a green background, and displays how the values will appear in the data table. She commits the action by clicking.

In the table, Jen can see that her scraped data is represented by a synthesized formula:

`QuerySelector(rowElement, “h3.s-item__title”)`

Wildcard calls this formula on each listing on the page (each `rowElement`) to get the child element that corresponds to the specified CSS selector (`“h3.s-item__title”`). Wildcard exposes this formula to the user to exemplify that **functional reactive programming** is the underlying design principle and to provide a template for scraping with formulas.

After this first demonstration, Jen has a reference to each of the page's row elements (i.e. each product listing) in the Wildcard table. She can now use these references to scrape elements within the product listing with formulas, such as the Sponsored label.

**Scraping Sponsored Labels with Formulas** (@fig:ebay Part B): Next, Jen attempts to scrape the “Sponsored” label into a new column using demonstration, but she is unable to scrape more than one letter of the word at a time. By inspecting the page’s source code using her browsers developer console, Jen finds that eBay developers have inserted invisible letters between the visible letters of “Sponsored,” with each letter in its own HTML element (possibly to obfuscate against ad blockers). Scraping by demonstration is insufficient because Jen wants the element that contains the whole word, but the system is giving her the constituent elements (individual characters) instead.

Jen can overcome this insufficiency by writing her own formula to target the containing element, using our system’s support of **mixed-initiative interaction**. She copies and pastes the formula from the previous column (which was synthesized by demonstration) into a new column, then she replaces the CSS selector with the selector of the containing element, which she identifies in the developer console. This formula populates the column with the text content of the scraped element, which she can now use for string manipulation and sorting.

**Filtering Sponsored Results** (@fig:ebay Part C): In the column that she just scraped, Jen sees that all of the sponsored results appear as garbled text, whereas the non-sponsored results appear as the word “Sponsored.” Thus, in a new column, she is able to write a new formula that returns whether or not the previous column’s text includes the word “Sponsored.” Then, she is immediately able to sort the listings by whether or not they are sponsored by sorting this column, thus hiding sponsored results from view. This customization is possible because Wildcard provides a **unified user model**.

In this way, Jen is able to use our system to customize the eBay website, without needing to learn how to program in JavaScript and without even leaving the webpage. We think that our model of customization by unified demonstration and formulas is flexible enough to support a wide range of other useful modifications and web programming proficiency levels, and we present a greater variety of use cases in [@sec:evaluation].


# System Implementation {#sec:implementation}

We start by describing the formula language used to represent web scraping programs. Then, we discuss the algorithm we use to generate the CSS selectors we present in the web scraping formulas. Finally, we present the *wrapper induction* [@kushmerick2000] algorithm that synthesizes the web scraping programs.

## Web Scraping Formulas

The formula language used to present web scraping formulas is similar to visual database query systems like SIEUFERD [@bakke2016] and Airtable [@zotero-228]. Formulas automatically apply across an entire column of data and reference other column names instead of values in specific rows. This is more efficient than users having to copy a formula across a column as in traditional spreadsheets like Microsoft Excel and Google Sheets. It of course comes at the cost of not being able to specify an operation for only a subset of columns but this hasn't yet come up in our use cases.

Web scraping formulas for rows have the following form:

`=QuerySelector(<selector>)`

The parameter `<selector>` refers to a CSS selector. Our CSS selector synthesis algorithm generates class-based selectors, falling back to index-based selectors (which utilize the `nth-child` notation) if the given elements do not have a `class` attribute. The name of the formula matches the `querySelector` method available on Document Object Model (DOM) elements and should thus be familiar to programmers that write web scraping code in Javascript. The web scraping formula for rows is used in a hidden column of the table named `rowElement` for which each column cell corresponds to the row element representing the table row.

Web scraping formulas for columns have the following form:

`=QuerySelector(rowElement, <selector>)`

The parameter `rowElement` is a reference to the hidden column containing row elements and `<selector>` is a CSS selector as previously described. This form is consistent with Wildcard's existing customization formulas which reference column names to perform operations on each of the cells in the column.

Representing web scraping code as web scraping formulas in the table allows programmers to not only view them to understand the outcome of the wrapper induction algorithm but also to modify them. Modifying a web scraping formula is as simply as editing the synthesized selector and executing the formula. Furthermore, programmers can manually author web scraping formulas in empty columns in a simpler fashion than the equivalent Javascript: all they have to do is determine the selector of the desired data and the system will take care of iterating through rows and extracting values from the selected elements.

## CSS Selector Synthesis Algorithm

Our system synthesizes two types of CSS selectors: a single row selector that selects a set of DOM elements corresponding to individual rows of the table and a column selector for each column which selects the element containing the column value within a given row.

For a given row element, its row selector is synthesized as follows:

1. Generate a list of all possible combinations of the classes on the element’s `class` attribute and initialize their scores to 0. For example, an element with a `class` attribute value of “a b c” would generate “a”, “b”, “c”, “a b”, “b c” and  “a b c”
2. Retrieve a list of all of the sibling elements of the element. For each sibling element, check whether its `class` attribute contains each of the generated class combinations. If a sibling’s `class` attribute contains a given class combination, the combination’s score is incremented by 1
3. Pick the class combinations with the highest score across the element’s siblings and then select the combination with the fewest number of classes. For example, if the combinations with the highest score are “a” and “b c”, “a” will be picked. This is done to ensure that only the minimal required classes are used for selection
4. Combine the tag name of the element with the final selector to further ensure that it only selects the desired row elements. For example, if the row element is has a tag name of `DIV` and the final selector is “a” the synthesized selector will be `div.a`

For a given column element, its column selector is synthesized as follows:

1. Generate a list of all possible combinations of the classes on the element’s `class` attribute, as previously described, and initialize their scores to 0
2. For each class combination, check that it only selects a single element within the row element and that that element corresponds to the given column element. If a class combination satisfies the check, its score is incremented by 1
3. Pick the class combinations with the highest score and then select the combination with the fewest number of classes as previously described
4. Combine the tag name of the element with the final selector to further ensure that it only selects the desired column element as previously described

One aspect of future work is saving the list of all valid selectors, instead of just picking one, and making them available to users to see and pick from. This would be similar to Mayer et el's user interaction model called *program navigation* [@mayer2015] that gives users the opportunity to navigate all valid, synthesized programs and pick the best one.

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

# Design Principles {#sec:design-principles}

Below, we describe three design princples that the implementation of our model embodies. We did not invent these principles but rather combined them in a novel manner for the domain of web scraping and web customization.

## Mixed-Initiative Interaction

In a position paper [@chugh2016a], Chugh discusses how programmatic and direct manipulation systems each have distinct strengths but users are often forced to choose one over another. As a solution, he makes a proposal for “novel software systems that tightly couple programmatic and direct manipulation” which led to the emergence of systems like Sketch-N-Sketch [@chugh2016]. More generally, this idea relates to work on mixed-initiative interaction by Horvitz [@horvitz1999] in which he advocates for “designs that take advantage of the power of direct manipulation and potentially valuable automated reasoning.” In our system, mixed-initiative interaction refers to the fluid interleaving of programming-by-demonstration and traditional programming for web scraping.

This type of mixed-initiative interaction can be seen in a number of programming-by-example systems. Sketch-N-Sketch [@chugh2016] allows users to create an SVG shape via traditional programming and then switch to modifying its size or shape via direct manipulation. Wrex [@drosos2020] takes examples of data transforms and generates readable and editable Python code. Small-Step Live Programming By Example [@ferdowsifard2020] presents a paradigm in which programming-by-example is used to synthesize small parts of a user authored program instead of delegating construction of entire program. Mayer et al [@mayer2015] developed a user interaction model called *program navigation* which allows users to navigate between all synthesized programs instead of only displaying the top-ranked one.

Our unified model for web scraping and customization offers mixed-initiative interaction by presenting the result of programming-by-demonstration as a formula. This is advantageous as it not only allows users to delegate automation to the system via demonstrations but also allows users to modify the output of the demonstration. In the Ebay example in [@sec:examples], the user starts out by demonstrating to scrape, switches to manually authoring a web scraping formula when demonstrating is insufficient and then switches back to demonstrating, all in a seamless and fluid manner.

## Functional Reactive Programming

In general terms, functional reactive programming (FRP) is the combination of functional and reactive programming: it is functional in that it uses functions to define programs that manipulate data and reactive in that it defines data flows through which changes in data are propagated. FRP has seen wide adoption in end-user programming through implementations such as spreadsheet formula languages (Microsoft Excel & Google Sheets) and formula languages for low-code programming environments (Microsoft Power Fx [@zotero-150], Google AppSheet [@zotero-218], Airtable [@zotero-228], Glide [@zotero-148], Coda [@zotero-155] & Gneiss [@chang2014]).

Wildcard [@litt2020; @litt2020b] already provided functional reactive programming via a spreadsheet formula language aimed at increasing the expressiveness of customizations. The language has formulas to encapsulate logic, perform operations on strings, call browser APIs and even invoke web APIs. As per the FRP paradigm, users only have to think in terms of operations on the data in the table without having to worry about traditional programming concepts such as variables, state and data flow. This makes it easier for them to create customizations in a declarative manner without having to worry about all the steps that have to take place to make this possible.

Our unified model for web scraping and customizations adds formulas to this formula language to mitigate the lack of expressiveness of programming-by-demonstration for web scraping. Demonstrations are represented as formulas containing the synthesized web scraping code as a CSS selector. As with other formulas in the language, web scraping formulas can be modified (or authored from scratch) and executed to achieve more expressive web scraping. We can see this in the Ebay example in [@sec:examples] when the user manually authors a formula to scrape the value of the column of ratings. Because of FRP, all the user has to do is provide is a CSS selector that will select the desired elements: the row iteration and extraction of values from the elements is done automatically for them.

## Unified User Model

Prior to this work, web scraping and web customization in customization systems [@huynh2006; @lin2009] were divided: web scraping had to be performed prior to customization in a separate phase. Vegemite [@lin2009], a system for end-user programming of mashups, reported findings from its user study in which participants thought that “it was confusing to use one technique to create the initial table, and another technique to add information to a new column”.

By representing the output of demonstrations as formulas, our new web scraping model fit right into Wildcard's customization paradigm which utilizes formulas for web customization. This enabled us to go beyond providing a model that combined web scraping by demonstration with web scraping by programming to providing a model that combined web scraping *and* customization. Both web scraping and customization are performed in the same, single phase, with users being able to seamlessly interleave the two as desired. 

We can see this in the Ebay example in [@sec:examples]: the user starts out by demonstrating to scrape values on the website into a column in the table, proceeds to populate the next column with the results of a formula, sorts the table to view the resulting customization and then continues on to the next task.

# Evaluation {#sec:evaluation}

## Example Gallery

Following a method used to evaluate visualizations through a diverse gallery of examples [@ren2018], our first evaluation of Joker provides a gallery of popular websites on which Joker can be used for web customization and on which it can not be used. For the websites on which Joker can be used, we provide the sequence of interactions needed to achieve the customizations. For the websites on which Joker cannot be used, we provide an explanation.

### Websites Joker Can Be Used On

Joker's unified model of demonstrations and formulas is most effective on webpages with data that is presented as many similarly-structured HTML elements. For example, we have used Joker to modify a broad range of product listing webpages such as those of Amazon, Target, and eBay. We have used Joker's support of scraping by demonstration and formulas to scrape prices, ratings, and spnosored tags; then we use formulas to refine the scraped data and filter the results, e.g. for low prices and high ratings. 

We have also used Joker to customize directory-like websites such as Google search results, the MIT course catalog, and the MIT CSAIL people directory. In these cases, Joker's scraping and string manipulation formulas can be used to filter the directories based on the contents of the entries; for example, on the MIT course catalog, Joker can filter for courses with no prerequisites by scraping the "prerequisite" text, then applying a formula that determines if the text contains the word "None".

Another category of websites that is suited for modification by Joker is news websites such as ABC news, old Reddit and HackerNews. For news listings, we have used Joker to scrape the listed links, then we use formulas that call APIs to display the links' read times and whether they have been visited in the browser history. We can also scrape elements like number of comments and the time of posting and sort by these values.


### Websites Joker Cannot Be Used On

Joker does not currently work well on some popular social media websites like Facebook, Twitter, new Reddit, and Pinterest. Although these websites have repeated data entries, these sites have complex or inconsistent HTML structures, which inhibits Joker's Wrapper Induction algorithm from determining the correct row selectors. Many of these sites also have an "infinite scroll" feature that adds new entries to the page, which disrupts Joker's bidirectional data synchronization. On some sites, like Pinterest, most of the data (e.g. time of posting) does not appear unless a user clicks on the entry, but Joker is restricted to scraping what is visible on the page at one point in time.


## User Study

Our second evaluation of Joker reports the qualitative results of a formative user study. The focus of the study was on usability rather than learnability as we are aware that there are many areas of improvement for that in the current implementation.

### Participants

We recruited 5 participants with backgrounds ranging from limited programming experience to Software Engineers. All participants where familiar with Microsfot Excel spreadsheets but not all had used Excel spreadsheet formulas. 4 of the participants had web development experience with 3 of them having extensive experience. 3 of the participants had web scraping scraping with only 1 having extensive experience.

### Tasks

The participants completed 7 tasks across 2 websites towards the goal of web customization. The first website was MIT's course catalog and the second was a listing of iphones after searching for "iphone" on eBay. The tasks were as follows:

*MIT Course Catalog.* Thise set of tasks (A) involved web scraping by demonstration and the use of web customization formulas: 1) Scrape course titles, 2) Scrapes course prerequites, 3) Add a column that indicates whether a course has a prerequiste & 4) Add a column that indicates whether a course does not have a prerequiste and is offered in the fall.

*eBay.* This set of tasks (B) involved web scraping by demonstration, the use of web scraping formulas and the use of web customization formulas: 1) Scrape iphone listing title, 2) Scrape iphone listing price, & 3) Create a column that indicates whether an iphone listing is sponsored.

### Protocol

All participants completed the MIT course catalog and eBay tasks in the order we have described. We started each session with a description of Joker and provided a brief tutorial of its main features (web scraping by demonstration, web scraping formulas & web customization formulas) on a website not used for the tasks. Sessions where held over video conferencing with the given participant sharing their screen and talking out loud as they worked on the tasks. There was no time limit for tasks but we provided hints whenever a participant had exhausted the knowledge available to them.

### Results

All participants were able to complete all the tasks with the help of hints when they got stuck and were unable to make progress on their own. We describe the results along the following dimensions:

*Web Scraping By Demonstration.* All participants where able to complete the tasks that only involved scraping by demonstration (A1, A2 & B1) quickly and without any hints. For tasks that involved switching between demonstrations and writing formulas, there was some confusion about what the active column, i.e. column the values would be scraped into, was. The active column is indicated and controlled by a toolbar away from the table at the top of the website but participants hardly noticed it and assumed all operations had to be performed on the table. This suggests that if a table is the basis of scraping and customization, all user interactions need to be with it.

*Web Customization Formulas.* Participants had varying degrees of trouble when completing the tasks that involved using web customization formulas (A3, A4, B3). Most of the confusion resulted from formula parameters and return values. The formula bar has an autocomplete feature that shows the documentation for a given formula. The documentation consists of the paremeters a formula it takes and a brief description of each parameter. However, this wasn't always sufficient to enable participants to use formulas correctly. This suggests that concrete examples of formula parameters and results could be more effective for describing usage. 

*Web Scraping Formulas.* Participants were unsure why demonstrations did not work when completing tasks that required using web scraping formulas (B2, B3). As a result, they wasted time attempting to demonstrate using various means. The participant with no web development experience had the hardest time as expected but was able to utlize a series of hints we provided to accomplish B2. The rest of the participants were able to accomplish B2 with minimal hints but required a lot more when completing B3. B3 involved scraping a value that was layed out in an adverserial manner in the DOM to prevent web scraping.

### Discussion

Overall, participants found the experience of using Joker to accomplish the tasks preferrable to the alternatives available to them. For the MIT course catalog tasks, the participant with no web development experience said that they couldn't think of how they could accomplish the tasks without Joker. 3 of the four participants with web development experience said that they preferred Joker to writing a program to accomplish the tasks because it would either take too long or be cumbersome to validate the results outside the context of the website. The 4th participant, who had the most web scraping experience, had the opposite preferrence because they would have more control over the scraping process if they wrote their own program.

We categorized the main usability issues and insights we obtained into three groups. The first is about the confusion concerning which column was the active column. This suggests that if a table is the basis of web scraping and customization, all user interactions need to be with the table and not split across various interfaces. The thrid is about the confusion concerning what parameters needed to be passed to formulas and what values would be returned. This suggests that users need more than documentation showing formula parameters and descriptions of the parameters. The third is about the confusion concerning why demonstrations were not sufficient to scrape certain values. This suggests that a prerequiste of increasing the expressiveness of web scraping beyond that available by demonstrating needs to communicate why demonstrations are not sufficient.

## Cognitive Dimensions Analysis

Our third evaluation of Joker analyzes it using Cognitive Dimensions of Notation [@blackwell2001], a heuristic evaluation framework that has been used to evaluate programming languages and visual programming systems (cite). When contrasting our tool with traditional scraping and other visual tools, we find particularly meaningful differences along several of the dimensions:

*Progressive evaluation.* In our tool, a user can see the intermediate results of their scraping and customization work at any point, and adjust their future actions accordingly. The table UI makes it easy to inspect the intermediate results and notice surprises like missing values.

In contrast, traditional scraping typically requires editing the code, manually re-running it, and inspecting the results in an unstructured textual format, making it harder to progressively evaluate the results. Also, many end-user scraping tools [@chasins2018; @lin2009] require the user to demonstrate all the data extractions they want to perform before showing any information about how those demonstrations will generalize across multiple examples.^[This is an instance where Wildcard's limitation of only scraping a single page at a time proves beneficial. All the relevant data is already available on the page without further network requests, making it possible to support progressive evaluation with low latency. Other tools that support scraping across multiple pages necessarily require a slower feedback loop.]

*Premature commitment.* Many scraping tools require making a _premature commitment_ to a data schema: first, the user extracts a dataset, and then they perform downstream analysis or customization using that data. Our previous versions of Wildcard suffered from this problem: when writing code for a scraping adapter, a user would need to try to anticipate all future customizations and extract the necessary data.

Our current system instead supports extracting data _on demand_. The user can decide on their desired customizations and data transformations, and extract data as needed to fulfill those tasks. There is never a need to eagerly guess what data will be needed in advance.

We have also borrowed a technique from spreadsheets for avoiding premature commitment: default naming. New data columns are automatically assigned a single-letter name, so that the user does not need to prematurely think of a name before extracting the data. (We have not yet implemented the capability to optionally rename demonstrated columns, but it would be straightforward to do so, and would provide a way to offer the benefits of names without requiring a premature commitment.)

*Provisionality.* Our tool makes it easy to try out a scraping action without fully committing to it. When the user hovers over any element in the page, they see a preview of how that data would be entered into the table, and then they can click if they'd like to proceed. This makes it feel very fast and lightweight to try scraping different elements on the page.

*Viscosity.* Some scraping tools have high viscosity: they make it difficult to change a single part of a scraping specification without modifying the global structure. For example, in Rousillon [@chasins2018], changing the desired demonstration for a single column of a table requires re-demonstrating all of the columns (todo: make 100% sure this is true). In contrast, our system allows a user to change the specification for a single column of a table without modifying the others, resulting in a lower viscosity in response to changes.

*Role-expressiveness.* One dimension we are still exploring in our tool is role-expressiveness: having different elements of a program clearly indicate their role to the user. In particular, in our current design, the visual display of references to DOM elements in the table is similar to the display of primitive values. In our experience, this can sometimes make it difficult to understand which parts of the table are directly interacting with the page, vs. processing the downstream results. In the future we could consider adding more visual differentiation to help users understand the role of different parts of their customization.

# Related Work {#sec:related-work}

Our unified model for web scraping and customization builds on existing work in end-user web scraping, end-user web customization and program synthesis by a number of systems.

## End-user Web Scraping

FlashExtract [@le2014] is a programming-by-example tool for data extraction. In addition to demonstrating whole values, it supports demonstrating substrings of values. Our model only supports this through formulas. This is not as end-user friendly but allows for a wider range of operations such as indicating whether demonstrated values contain a certain value or are greater than or less than a certain value.

Rousillon [@chasins2018] is a tool that enables end-users to scrape distributed, hierarchical web data. It presents the web scraping code generated by demonstration as an editable, high-level, block-based language called Helena [@zotero-179]. While Helena can be used to specify complex web scraping tasks like adding control flow, it does not present the synthesized web scraping program. This means that users can only scrape what can be demonstrated. Our modek on the other hand displays the synthesized program as a formula which can be modified to increase the expressiveness of scraping.

## End-user Web Customization

Vegemite [@lin2009] is a tool for end-user programming of mashups. Unlike our model, its table interface is only populated and can only be interacted with after all the demonstrations have been provided. This does not support interleaving of scraping and table operations to achieve an incremental workflow for users. Mashups are created using CoScripter [@leshed2008] which records operations on the scraped values in the table for automation tasks. CoScripter provides the generated automation program as text-based commands, such as “paste address into ‘Walk Score’ input”, which can be edited via “sloppy programming” [@lin2009] techniques. However, this editing does not extend to the synthesized web scraping program which is not displayed and therefore cannot be modified. This means that users can only scrape what can be demonstrated.

Sifter [@huynh2006] is a tool that augments websites with advanced sorting and filtering functionality. It attempts to automatically detect items and fields on the page with a variety of clever heuristics. If this falls, it gives the user the option of demonstrating to correct the result. In contrast, our model is simpler and makes fewer assumptions about the structure of websites by giving control to the user from the beginning of the process and displaying the synthesized program which can be modified. We hypothesize that focusing on a tight feedback loop rather than automation may support a scraping process that is just as fast as an automated one, offers more expressive scraping and extends to a greater variety of websites. However, further user user testing is required to validate this hypothesis.

## Program Synthesis

FlashProg [@mayer2015] is a framework that provides program navigation and disambiguation for programming-by-example tools like FlashExtract [@le2014] and FlashFill [@harris]. The program viewer provides a high level description of synthesized programs as well as a way to navigate the list of alternative programs that satisfy the demonstrations. This is important because demonstrations are an ambiguous specification for program synthesis [@peleg2018]: the set of synthesized programs for a demonstration can be very large. To further ensure that the best synthesized program is arrived at, FlashProg has a disambiguation viewer that asks the user questions in order to resolve ambiguities in the user's demonstrations. In contrast, our model only presents the top-ranked synthesized program which may not be the best one. Furthermore, the program is presented in is low-level form as a CSS selector. This is not end-user friendly but allows for more expressiveness.

# Conclusion And Future Work {#sec:conclusion}






