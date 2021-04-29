---
title: "A Unified Interaction Model For Web Scraping & Customization"
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
 Web scraping, the process of extracting structured data from a website, is a common building block of web customization systems. Prior approaches have allowed users to perform web scraping by directly demonstrating examples of the data to scrape. However, demonstrations alone do not allow for as much expressiveness as traditional programming which is vital for complex web scraping tasks.

 In this paper, we present a new interaction model for web scraping that combines the ease of use of programming-by-demonstration and the expressiveness of traditional programming. When a user demonstrates examples of data to extract, a web scraping program is synthesized and presented in a formula language with pure functional semantics. Crucially, the user can directly edit and execute formulas, allowing them to express scraping operations which can not be achieved via demonstration alone.

 To illustrate our model, we implement it as a browser extension called Joker. Through concrete examples, a small user study and a Cognitive Dimenions of Notation analysis, we show how Joker offers an expressive interaction model for web scraping and customization.

---

# Introduction {#sec:introduction}

Many websites on the internet do not meet the exact needs of all of their users. Because of this, millions of people use browser extensions like Greasemonkey [@zotero-224] and Tampermoneky [@2021f] to install userscripts, snippets of Javascript code which customize the behavior of websites. To make the creation of web customizations more accessible to end-users without knowledge of programming, researchers have developed systems like Sifter [@huynh2006], Vegemite [@lin2009] and Wildcard [@litt2020; @litt2020b].

A common building block of these web customization systems is web scraping, the extraction of structured data from websites. They achieve web scraping in one of two ways: programming-by-demonstration [@huynh2006; @lin2009] and traditional programming [@litt2020; @litt2020b]. Web scraping by demonstration automates the web scraping process by utilizing program synthesis to synthesize a web scraping program from user demonstrations. This approach is accessible to end-users with no programming experience and enables them to fully participate in the customization lifecycle. However, web scraping by demonstration is not powerful enough to perform certain complex web scraping tasks. On the other side, web scraping by programming involves programmers manually writing web scraping code. It is powerful enough to scrape a wider variety of websites but is not accessible to end-users who can only customize websites that programmers have written web scraping code for.

In this paper, we present a new interaction model for web scraping for customization that combines the ease of use of programming-by-demonstration and the expressiveness of traditional programming. At its core lies a simple formula language. The language offers pure functional semantics for expressing and executing data extractions from websites using Cascading Style Sheet (CSS) selectors. When a user demonstrates examples of data to scrape, a web scraping program is synthesized via wrapper induction [@kushmerick2000] and is presented as a formula containing the CSS selector required to identify the Document Object Model (DOM) element with the data ([@sec:implementation]). Crucially, the formula can be directly edited and executed to specify complex scraping operations which can not be achieved by demonstrations alone.

To illustrate our new interaction model for web scraping, we have implemented it as an extension of Wildcard [@litt2020; @litt2020b] called Joker. Wildcard is a browser extension that enables web customization by direct manipulation of a spreadsheet-like table it adds to websites. The table contains the website's underlying structured data and is bidirectionally synchronized with it. User interactions with the table, including sorting, adding columns and running computations in a spreadsheet formula language, are propagated to the website thereby customizing it. The web scraping required to create Wildcard's table can only be achieved via traditional programming in Javascript. This means that end-users can only customize websites that programmers have written web scraping code for. Joker enables end-users to achieve the web scraping tasks required for customization while maintaining some of the expressiveness of traditional programming.

By representing web scraping programs as formulas, our web scraping interaction model fit right into Wildcard's customization paradigm which utilizes formulas for web customization. This enabled us to go beyond providing an interaction model that combines web scraping by demonstration with web scraping by programming to providing an interaction model that combines web scraping *and* customization. We refer to this as a *unified interaction model for web scraping and customization*. In [@sec:examples], we show how this offers more expressive web scraping and customization.

Our contributions are as follows:

- A **unified interaction model for web scraping and customization** that combines web scraping (by demonstration and programing) with web customization through a shared formula language with pure functional semantics
- An implementation of the unified interaction model for web scraping and customization called Joker that combines three key design principles (**prodirect manipulation**, **functional reactive programming** & **unified user interaction**) in a novel way ([@sec:design-principles])
- The results of an evaluation of the model via an example gallery, a five person user study and a Cognitive Dimensions of Notion analysis ([@sec:evaluation])

We end by discussing opportunities for future work and how we can make CSS selectors more accessible to end-users without development experience ([@sec:conclusion]).

# Example Usage Scenario {#sec:examples}
To concretely illustrate the user experience of our tool, we present a scenario of customizing eBay, a popular online marketplace. The goal of the user, Jen, is to filter out search results that are sponsored, i.e. product listings that a seller paid to promote. On the webpage, sponsored listings are marked with a "Sponsored" label that Jen can try to scrape, then she can sort the scraped labels to move all sponsored listings to the bottom of the page. @fig:ebay shows accompanying screenshots.

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/ebay.png}
  \caption{\label{fig:ebay}Scraping and customizing eBay by unified demonstration and formulas.}
\end{figure*}
</div>

*Scraping Names by Demonstration* (@fig:ebay Part A): Jen starts the adapter creation process by clicking a context menu item within the eBay.com page, then hovering over the data that she would like to scrape: the page element that contains the name of the listing. The system provides live feedback as Jen hovers: it annotates the row of data with a border, highlights the column of data in the page with a green background, and displays how the values will appear in the data table. When she clicks, the highlighted data is saved into the table.

The text content of the highlighted element is now displayed in the first column of the table, with one cell for each listing containing the name of the listing. When she clicks on a cell, Jen can see that the textual data is also represented by a formula:

`QuerySelector(rowElement, "h3.s-item__title")`

This formula reveals how Joker is actually scraping this column's data behind the scenes: it calls a query selector on each listing (i.e. each `rowElement`) on the page. Scraped data is stored as functions that act on the data on the page, instead of as static text, based on our design principle of **functional reactive programming**. Joker exposes this formula to the user to exemplify how scraping is achieved with formulas and to provide a template for scraping with formulas.

After this first demonstration, Jen has a reference to each of the page's row elements (i.e. each product listing) in the Joker table. She can now use these references to scrape elements within the product listing with formulas, such as the Sponsored label.

*Scraping Sponsored Labels with Formulas* (@fig:ebay Part B): Next, Jen attempts to scrape contents of the “Sponsored” label element into a new column; having this column would allow her to sort the table based on whether the listing contains the "Sponsored" label. She tries to scrape the label using demonstration, but she is unable to scrape more than one letter of the word at a time. To diagnose the issue, Jen inspects the page’s source code using her browser's developer tools. Jen discovers that eBay's developers have inserted invisible letters into word “Sponsored” (possibly to obfuscate against ad blockers). Each letter is in its own HTML element, and the inserted letters are rendered invisible by CSS. Scraping by demonstration does not work in this case because Jen wants the element that contains the whole word, but the system is giving her the leaf-node elements (individual letters) instead.

Jen sees in the source code that the all of the letters of the "Sponsored" label, both visible and invisible, are contained in a single ancestor element with the CSS selector `"div.s-item__title--tagblock"`. Thus, she is able to scrape the full word by writing a query selector formula with with that CSS selector. She can copy the formula from the previous column as a template.

This formula populates the column with the text content of the "Sponsored" element, which she can now use for string manipulation and sorting. By writing a formula, Jen is able to overcome the limitations of scraping by demonstration. The mixed methods of scraping exemplify our tool's support of **prodirect manipulation**. 

*Filtering Sponsored Results* (@fig:ebay Part C): The query selector that Jen just wrote returns all of the text within each of the targeted "Sponsored" elements, including any invisible letters. eBay's web design is that sponsored listings have a visible "Sponsored" label with invisible inserted letters, and non-sponsored listings have an invisible "Sponsored" label. Thus, for sponsored listings, the query returns garbled text (e.g. "JSp3onMsoV3rXNYFedZB"), and for non-sponsored listings, the query returns "Sponsored". Jen identifies this correlation by scrolling through the scraped data in the column and comparing them to what she sees on the web page. Then, in a new column, Jen writes a formula that returns whether or not the previous column’s text includes the word “Sponsored.” Finally, she sorts the listings by whether or not they are sponsored by sorting this column, thus hiding sponsored results from view. This customization is possible because Joker provides a **unified user model**, where scraping and customization are performed in conjunction.

In this way, Jen is able to use our system to customize the eBay website, without needing to learn how to program in JavaScript and without even leaving the webpage. Our model of customization by unified demonstration and formulas is flexible enough to support a wide range of other useful modifications and web programming proficiency levels, and we present a greater variety of use cases in [@sec:evaluation].


# System Implementation {#sec:implementation}

In this section, we outline the *wrapper induction* [@kushmerick2000] algorithm that Joker uses to synthesize web scraping programs from user demonstrations. Then, we briefly describe the formula language used to represent the synthesized web scraping programs and include a list of the formulas that are currently available and their roles.

## Wrapper Induction

In order to create web scraping programs from users demonstrations, Joker solves the wrapper induction [@kushmerick2000] task: generalizing from a few examples of data in a data set to a specification for all the data in the data set.

Joker takes an approach similar to that used in systems like Vegemite [@lin2009] and Sifter [@huynh2006]. It synthesizes a single *row selector* for the website: a CSS selector that identifies a set of DOM elements corresponding to the rows of the data set. For each column in the data set, it synthesizes a *column selector*, a CSS selector that identifies the element containing the column value.

One important difference is that our algorithm only accepts row elements that have direct siblings with a similar structure. We refer to this as the *row-sibling* constraint. Later, we describe how this constraint provides a useful simplification of the wrapper induction task and in [@section:evaluation] discuss the resulting limitations this puts on our system. We proceed to describe how CSS selectors are synthesized for row and column elements and then explain the criteria used to determine row elements.

### Synthesizing CSS Selectors

Joker synthesizes two types of CSS selectors: a single row selector that selects a set of DOM elements corresponding to the rows of the data set and a column selector for each column which selects the element containing the column value within a given row.

For a given row element, its row selector is synthesized using the following criteria:

*Plausibility*. A selector is set of classes. It is a plausible row selector if it 1) consists of a subset of the classes on the row element and 2) consists of a subset of the classes all the row element's siblings. The second requirement is the *row-sibling* constraint we mentioned. Notice how it simplifies the problem by eliminating selectors. It is necessary for the end goal of customization because of customizations like sorting. If row elements are not siblings, the website would end up in an invalid state when the DOM elements are re-organized to match the sorting order.

*Weight*. A selector has a weight equal to the number of classes it consists of.

*Best*. A selector is the best if it is plausible and there is no other selector that has a lower weight than it. We favor selectors with the lowest weight to ensure that only the minimum required classes are utilized. If there are multiple selectors that are plausible and have the lowest weight, we only pick one.

For a given column element, its column selector is synthesized using the following criteria:

*Plausibility*. A selector is a plausible column selector if it 1) consists of a subset of the classes on the column element and 2) only selects the give column element when applied on the corresponding row element.

*Weight*. A selector has a weight equal to the number of classes it consists of.

*Best*. A selector is the best if it is plausible and there is no other selector that has a lower weight than it has. As before, we favor selectors with the lowest weight and only pick one if there are multiple that fulfill the criteria.

One aspect of future work is saving the list of all selectors that fulfill the criteria and making them available to users to view and pick from via the formula. This would be similar to Mayer et el's user interaction model called *program navigation* [@mayer2015] that gives users the opportunity to navigate all valid, synthesized programs and pick the best one.

### Determining Row Elements

When a user first demonstrates a column value, Joker uses the demonstration to synthesize a row selector. The row selector will be used to identify all the row elements in the website. Additionally, a column selector is synthesized that will be used to identify the element that contains the column value. During subsequent demonstrations, Joker simply synthesizes a column selector for the column element that contains the demonstrated column value. Like similar approaches [@huynh2006; @lin2009; @chasins2018], all demonstrations have to be made from the same row element.

Given a demonstrated column value, row elements are determined using the following criteria:

*Plausibility*. An element `R` is a plausible row element if 1) it is within the `BODY` element of the DOM, 2) it is in the parent path of the column element `C` containing the demonstrated column value `V` and 3) the CSS selector `S` of element `C` only identifies `C` when applied to `R`.

*Weight*. A row element `R` has a weight `W` equal to the number of its siblings for which the CSS selector `S` of column element `C` only identifies `C` when applied to it`.

*Best*. A row element `R` is the best if it is plausible and there is no other row element that has a higher weight than it. We favor row elements with the highest weight to ensure that we end up with a data set with the highest number of column values corresponding to the column that `V` belongs to. If there are multiple plausible row elements with the highest weight, we pick the one closet to the column element `C` in its parent path.

@fig:algorithm provides a concrete example of how the above criteria are applied to determine a row element from the demonstration of a column value.

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/algorithm.png}
  \caption{\label{fig:algorithm} An example of how Joker's wrapper indunction algorithm is used to determine the row element from the demonstration of a column value. The row element is correctly determind to be R2 because it is plausible and has the highest weight.}
\end{figure*}
</div>

## Web Scraping Formulas

Joker's formula language is similar to that of visual database query systems like SIEUFERD [@bakke2016] and Airtable [@2021f]. Formulas automatically apply across an entire column of data and reference other column names instead of values in specific rows. This is more efficient than users having to copy a formula across a column as in traditional spreadsheets like Microsoft Excel and Google Sheets. It of course comes at the cost of not being able to specify a formula for only a subset of column cells but this hasn't yet come up in our use cases. The language currently consists of the following formulas:

### QuerySelector(rowElement, selector)

This formula is used to represent the web scraping program synthesized from demonstrations. `rowElement` is a special keyword that references a hidden column containing the DOM elements that correspond to the rows of the data set. `selector` is the synthesized CSS selector that specifies the column element. 

### GetParent(element)

This formula is used to traverse the DOM when the data to be scraped is made of the values of its containing and sibling elements. Demonstrations alone cannot be used to scrape such data. `element` can be a reference to a column containing a `QuerySelector` formula (`GetParent(A)`) or a `QuerySelector` formula itself (`=GetParent(QuerySelector(...))`).

### GetAttribute(element, attribute)

This formula is used to scrape data from DOM attributes. An example of this are URLs which are available on the `href` attribute of link elements. `element` is as described for `GetParent` and `attribute` is the name of the attribute to scrape (e.g. `GetAttribute(A, "href")`).

# Design Principles {#sec:design-principles}

Below, we describe three design principles that Joker embodies. We did not invent these principles but rather combined them in a novel manner for the domain of web scraping and web customization.

## Prodirect Manipulation

*Prodirect Manipulation* is a term coined by Ravi Chugh in a position paper [@chugh2016a] in which he advocates for “novel software systems that tightly couple programmatic and direct manipulation”. This principles embodies our goal of an interaction model for web scraping that combines the ease of use of programming-by-demonstration and the expressiveness of traditional programming.

In the eBay example in [@sec:examples], we show how the “Sponsored" label of a listing can not be scraped by demonstration alone. However, through Joker's prodirect manipulation interaction, a user can either directly edit the synthesized web scraping formula or author one from scratch to achieve the task. This is significant because the resulting customization to sort the listings by whether they are sponsored or not would not otherwise be possible

We can see prodirect manipulation in other programming-by-example systems. Sketch-N-Sketch [@chugh2016] allows users to create an SVG shape via traditional programming and then switch to modifying its size or shape via direct manipulation. Wrex [@drosos2020] takes examples of data transforms and generates readable and editable Python code.

## Functional Reactive Programming

Functional reactive programming (FRP) utilizes pure functions to specify operations in a reactive programming paradigm. FRP has seen wide adoption in end-user programming through implementations such as spreadsheet formula languages (Microsoft Excel & Google Sheets) and formula languages for low-code programming environments (Microsoft Power Fx [@2021g], Google AppSheet [@2021h], Airtable [@2021f], Glide [@2021a], Coda [@2021c] & Gneiss [@chang2014]).

Joker's use of FRP makes it easier for users to specify scraping operations programmatically, thus making our web scraping interaction model more viable. With traditional programming, the eBay example in [@sec:examples] in which demonstrations are not enough to scrape a listing's "Sponsored" label would be much more complicated. The user would need to understand programming constructs such as state, variables, looping and data flow in order to write a program to scrape the label. In Joker, all a user needs to specify is the CSS selector that identifies the element containing the label. Joker takes care of managing state, variables, looping and data flow!

A key limitation of this approach is that users need to understand how CSS selectors work. Because the formula language utilizes pure functional semantics, users can iterate on CSS selectors as many times as they need to without having to worry about side effects. This makes the authoring of CSS selectors more accessible but more work remains for us to make this fully accessible to end-users. 

## Unified User Interaction

Prior to this work, web scraping and web customization in customization systems [@huynh2006; @lin2009] were divided: web scraping had to be performed prior to customization in a separate phase. Vegemite [@lin2009], a system for end-user programming of mashups, reported findings from its user study in which participants thought that “it was confusing to use one technique to create the initial table, and another technique to add information to a new column”.

Joker represents web scraping and customization using the same formula language. Because of this, both can be performed in a single phase, with users being able to interleave the two as desired. This makes the process of customization more incremental which is vital for complex tasks. We can see this in the Ebay example in [@sec:examples]: when the user scrapes a listing's "Sponsored" label, they observe that non-sponsored listings have an invisible "Sponsored" label while sponsored listings have a visible "Sponsored" label that consists of a garbled form "Sponsored". Because of Joker's unified user interaction, the user receives the results of scraping in the table and can immediately write a customization formula to validate this hypothesis. Without the unified interaction, the user would have to scrape all the desired columns to see all their values before ever getting to notice the pattern and validate their hypothesis.

# Evaluation {#sec:evaluation}

In this section, we evaluate Joker with an example gallery of websites, a small user study and a Cognitive Dimensions of Notion analysis [@blackwell2001]. The example gallery provides examples of websites that can be scraped and customized by Joker and known limitations. The user study provides insights on which aspects of Joker work well and which don't. The Cognitive Dimensions of Notion analysis provides thoughts on how Joker's design compares to similar approaches and interaction standards.

## Example Gallery

Following a method used to evaluate visualizations through a diverse gallery of examples [@ren2018], our first evaluation of Joker provides an example gallery of popular websites on which Joker can be used for web customization and on which it can not be used. For the websites on which Joker can be used, we provide the sequence of interactions needed to achieve the customizations. For the websites on which Joker cannot be used, we provide an explanation.

### Websites Joker Can Be Used On

We have used Joker to achieve a variety of purposes across many popular websites. For example, we have used Joker to sort search results by price within the Featured page on Amazon. (Using Amazon's sort by price feature often returns irrelevant results.) In Amazon's source code, the price is split into three HTML elements: the dollar sign, the dollar amount, and the cents amount. A user can only scrape the cents element by demonstration into column A. However, because the parent element of the cents element contains all three of the price elements, the user can scrape the full price using the formula `GetParent(A)`. Next, the user can write the formula `ExtractNumber(B)` to convert the string into a numeric value. Finally, the user can sort this column by low-to-high prices. In a similar manner, we have used Joker to scrape and sort prices and ratings on the product listing pages of Target and eBay.

We have also found Joker to be useful for filtering based on text inputs. For example, we have used Joker to filter the titles of a researcher's publications on their Google Scholar profile. Specifically, a user can first scrape the titles into column A by demonstration. Then, the user can write the formula `Includes(A, "compiler")` that returns whether or not the title contains the keyword "compiler". Finally, the user can sort by this column to get all of the publications that fit their constraint at the top of the page. We have also used Joker to filter other text-based directory web pages such as Google search results and the MIT course catalog, in similar ways.

Additionally, we have used Joker to augment web pages with external information. For example, Joker can augment Reddit's old user interface, which has a list of headlines with links to articles and images. A user can first scrape the headline elements into column A by demonstration. The user can then extract the link into column B with the formula `GetAttribute(A, "href")`. Then, the user can write the formula `ReadTimeInSeconds(B)` that calls an API that returns the links' read times. Similarly, the user can write the formula `Visited(B)` that returns whether that link has been visited in the user's browser history. The user can also scrape elements such as the number of comments and the time of posting and sort by these values. We have performed similar customizations on websites such as ABC news.

### Limitations
Joker's unified model of demonstrations and formulas is most effective on webpages with data that is presented as many similarly-structured HTML elements. However, certain websites have designs that make it difficult for Joker to scrape data. These are some of those designs:

- *Multiple row elements.* The layout of some web pages has multiple types of row as siblings that contain different children elements. For example, the news aggregator website HackerNews has a page design that alternates between rows containing a title and rows containing supplementary data (e.g. number of likes and the time of posting). Because Joker only chooses a single row selector, when scraping by demonstration, Joker will only select one of the types of rows, and elements in the other types of rows will not be able to be scraped.
- *Infinite scroll.* Some web pages have an "infinite scroll" feature that adds new entries to the page when a user scrolls to the bottom. Joker's table will only contain elements that were rendered when the table was first created. Additionally, for websites with many elements, such as Facebook, Joker might run out of memory while running its wrapper induction algorithm and crash the page.
- *Data hidden behind an interaction.* On some sites, a user must click on an element to reveal data corresponding to that entry (e.g. time of posting, the author). However, Joker is restricted to scraping what is visible on the page at one point in time.

## User Study

Our second evaluation of Joker is a user study which provides insights on what aspects of Joker work well and what don't. Because of this, successful completion of tasks does not equal validation of Joker. It does however inform us about usability issues that limit its effectiveness.

### Participants

We recruited 5 participants with backgrounds ranging from limited programming experience to Software Engineers. All participants were familiar with Microsoft Excel spreadsheets but not all had used Excel spreadsheet formulas. 4 of the participants had web development experience with 3 of them having extensive experience. 3 of the participants had web scraping experience with only 1 having extensive experience.

### Tasks

The participants completed 7 tasks across 2 websites towards the goal of web customization. The first website was MIT's course catalog and the second was a listing of iPhones after searching for "iPhone" on eBay. The tasks were as follows:

*MIT Course Catalog.* This set of tasks involved web scraping by demonstration and the use of web customization formulas: 1a) Scrape course titles, 1b) Scrapes course prerequisites, 1c) Add a column that indicates whether a course has a prerequisite & 1d) Add a column that indicates whether a course does not have a prerequisite and is offered in the fall.

*eBay.* This set of tasks involved web scraping by demonstration, the use of web scraping formulas and the use of web customization formulas: 2a) Scrape iPhone listing title, 2b) Scrape iPhone listing price, & 2c) Create a column that indicates whether an iPhone listing is sponsored.

### Protocol

All participants attempted the MIT course catalog and eBay tasks in the order we have described. We started each session with a description of Joker and provided a brief tutorial of its main features on a website not used for the tasks. There was no time limit for tasks but we provided hints whenever a participant had exhausted the knowledge available to them.

### Results

All participants were able to complete all the tasks with the help of hints when they got stuck and were unable to make progress on their own. Again, this does not equal validation. We describe the results along the following dimensions:

*Web Scraping By Demonstration.* All participants where able to complete the tasks that only involved scraping by demonstration (1a, 1b & 2a) quickly and without any hints. For tasks that involved switching between demonstrations and writing formulas, there was some confusion about what column demonstrated values would be scraped into. The active column is managed by a toolbar at the top of the website but participants hardly noticed it and assumed all operations had to be performed on the table.

*Web Scraping Formulas.* Participants were unsure why demonstrations did not work when completing tasks that required using web scraping formulas (2b, 2c). As a result, they wasted time attempting to demonstrate using various means. The participant with no web development experience had the hardest time as expected but was able to utilize a series of hints we provided to accomplish 2b. The rest of the participants were able to accomplish 2b with minimal hints but required a lot more when completing 2c.

*Web Customization Formulas.* Participants had varying degrees of trouble when completing the tasks that involved using web customization formulas (1c, 1d, 2c). Most of the confusion resulted from formula parameters and return values. The formula bar has an autocomplete feature that shows the documentation for a given formula. The documentation consists of the parameters a formula it takes and a brief description of each parameter. However, this wasn't always sufficient to enable participants to use formulas correctly.

### Discussion

One participant, upon seeing our tutorial on scraping by demonstration, said "that's like black magic." Tasks that involved customization formulas took longer than those that involved demonstrations but this was mostly related to the formula documentation not providing enough usage information. More work is needed to show users what exactly a given formula does. When asked how they would accomplish the tasks without Joker, the participant with limited programming experience responded with "I don't think I would know how to do it." That Joker enabled them to achieve such a complex task is very promising!

Participants that were familiar with web development were able to take advantage of Joker's web scraping formulas. This included the participants that had limited web scraping experience and suggests that the pure functional semantics and use of CSS selectors for identifying elements to scrape makes the task easier. As predicted, participants not familiar with web development didn't have the required background to understand how CSS selectors work. This highlights the need to further explore how web scraping formulas can be made more accessible to end-users.

All participants had trouble figuring out which column the demonstrated values would be scraped into, especially when demonstrations where interleaved with use of formulas. This points to the need for the switch between demonstrating and using formulas to be more tightly coupled visually. For web scraping formulas, the most notable issue was the lack of feedback about why demonstrations were not sufficient to scrape a value. As a result, users wasted time trying to demonstrate in various ways instead of thinking about using web scraping formulas to achieve the task. In order for traditional programming to augment demonstrations as we propose, users need to recieve some kind of feedback that demonstrations are not sufficient.

## Cognitive Dimensions Analysis

Our third evaluation of Joker analyzes it using Cognitive Dimensions of Notation [@blackwell2001], a heuristic evaluation framework that has been used to evaluate programming languages and visual programming systems [@satyanarayan2014; @satyanarayan2014a; @ledo2018]. When contrasting our tool with traditional scraping and other visual tools, we find particularly meaningful differences along several of the dimensions:

*Progressive evaluation.* In Joker, a user can see the intermediate results of their scraping and customization work at any point, and adjust their future actions accordingly. The table user interface makes it easy to inspect the intermediate results and notice surprises like missing values.

In contrast, traditional scraping typically requires editing the code, manually re-running it, and inspecting the results in an unstructured textual format, making it harder to progressively evaluate the results. Also, many end-user scraping tools [@chasins2018; @lin2009] require the user to demonstrate all the data extractions they want to perform before showing any information about how those demonstrations will generalize across multiple examples.^[This is an instance where Joker's limitation of only scraping a single page at a time proves beneficial. All the relevant data is already available on the page without further network requests, making it possible to support progressive evaluation with low latency. Other tools that support scraping across multiple pages necessarily require a slower feedback loop.]

*Premature commitment.* Many scraping tools require making a _premature commitment_ to a data schema: first, the user extracts a dataset, and then they perform downstream analysis or customization using that data. Wildcard suffered from this problem: when writing code for a scraping adapter, a user would need to try to anticipate all future customizations and extract the necessary data.

Our current system instead supports extracting data _on demand_. The user can decide on their desired customizations and data transformations, and extract data as needed to fulfill those tasks. There is never a need to eagerly guess what data will be needed in advance.

We have also borrowed a technique from spreadsheets for avoiding premature commitment: default naming. New data columns are automatically assigned a single-letter name, so that the user does not need to prematurely think of a name before extracting the data. We have not yet implemented the capability to optionally rename demonstrated columns, but it would be straightforward to do so, and would provide a way to offer the benefits of names without requiring a premature commitment.

*Provisionality.* Joker makes it easy to try out a scraping action without fully committing to it. When the user hovers over any element in the page, they see a preview of how that data would be entered into the table, and then they can click if they'd like to proceed. This makes it feel very fast and lightweight to try scraping different elements on the page.

*Viscosity.* Some scraping tools have high viscosity: they make it difficult to change a single part of a scraping specification without modifying the global structure. For example, in Rousillon [@chasins2018], changing the desired demonstration for a single column of a table requires re-demonstrating all of the columns. In contrast, Joker allows a user to change the specification for a single column of a table without modifying the others, resulting in a lower viscosity in response to changes.

*Role-expressiveness.* One dimension we are still exploring in our tool is role-expressiveness: having different elements of a program clearly indicate their role to the user. In particular, in our current design, the visual display of references to DOM elements in the table is similar to the display of primitive values. In our experience, this can sometimes make it difficult to understand which parts of the table are directly interacting with the website, vs. processing the downstream results. In the future we could consider adding more visual differentiation to help users understand the role of different parts of their customization.

# Related Work {#sec:related-work}

Joker builds on existing work in end-user web scraping, end-user web customization and program synthesis by a number of systems.

## End-user Web Scraping

FlashExtract [@le2014] is a programming-by-example tool for data extraction. In addition to demonstrating whole values, it supports demonstrating substrings of values. Joker only supports this through formulas. This is not as end-user friendly but allows for a wider range of operations such as indicating whether demonstrated values contain a certain value or are greater than or less than a certain value.

Rousillon [@chasins2018] is a tool that enables end-users to scrape distributed, hierarchical web data. It presents the web scraping program generated by demonstration as an editable, high-level, block-based language called Helena [@2021c]. While Helena can be used to specify complex web scraping tasks like adding control flow, it does not present the synthesized web scraping program. This means that users can only scrape what can be demonstrated. Joker on the other hand displays the synthesized program as a formula which can be modified to increase the expressiveness of scraping.

## End-user Web Customization

Vegemite [@lin2009] is a tool for end-user programming of mashups. Unlike Joker, its table interface is only populated and can only be interacted with after all the demonstrations have been provided. This does not support interleaving of scraping and table operations to achieve an incremental workflow for users. Mashups are created using CoScripter [@leshed2008] which records operations on the scraped values in the table for automation tasks. CoScripter provides the generated automation program as text-based commands, such as “paste address into ‘Walk Score’ input”, which can be edited via “sloppy programming” [@lin2009] techniques. However, this editing does not extend to the synthesized web scraping program which is not displayed and therefore cannot be modified. This means that users can only scrape what can be demonstrated.

Sifter [@huynh2006] is a tool that augments websites with advanced sorting and filtering functionality. It attempts to automatically detect items and fields on the page with a variety of clever heuristics. If this falls, it gives the user the option of demonstrating to correct the result. In contrast, Joker is simpler and makes fewer assumptions about the structure of websites by giving control to the user from the beginning of the process and displaying the synthesized program which can be modified. We hypothesize that focusing on a tight feedback loop rather than automation may support a scraping process that is just as fast as an automated one, offers more expressive scraping and extends to a greater variety of websites. However, further user user testing is required to validate this hypothesis.

## Program Synthesis

FlashProg [@mayer2015] is a framework that provides program navigation and disambiguation for programming-by-example tools like FlashExtract [@le2014] and FlashFill [@harris]. The program viewer provides a high level description of synthesized programs as well as a way to navigate the list of alternative programs that satisfy the demonstrations. This is important because demonstrations are an ambiguous specification for program synthesis [@peleg2018]: the set of synthesized programs for a demonstration can be very large. To further ensure that the best synthesized program is arrived at, FlashProg has a disambiguation viewer that asks the user questions in order to resolve ambiguities in the user's demonstrations. In contrast, Joker only presents the top-ranked synthesized program which may not be the best one. Furthermore, the program is presented in is low-level form as a CSS selector. This is not end-user friendly but allows for more expressiveness.

# Conclusion And Future Work {#sec:conclusion}

In this paper, we presented our unified interaction model for web scraping and customization through a browser extension called Joker. This model combines the ease of use of programming-by-demonstration and the expressiveness of traditional programming. At its core lies a simple formula language that offers pure functional semantics for expressing and executing data extractions from websites using Cascading Style Sheet (CSS) selectors.

The main area of future work involves providing feedback to users about when demonstrations are not sufficient. Otherwise, users won't know to switch to formulas to take advantage of their increased expressiveness. FlashProg [@mayer2015] and Wrangler [@kandel2011], whose interfaces also have tables, offer clues for how to provide feedback about the execution of programs synthesized from demonstrations. Another important area involves making the formula language more accessible to end-users not familiar with CSS selectors. Again, FlashProg offers some clues through its use of high level descriptions to give meaning to synthesized programs. One avenue for associating meaning with CSS selectors could be by extracting semantic web content as seen in systems like Thresher [@hogue2005].