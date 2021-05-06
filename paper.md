---
title: "Joker: A Unified Interaction Model For Web Customization"
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
  Tools that enable end-users to customize websites typically use a two-stage workflow: first, users extract data into a structured form, second, they use that extracted data to augment the original website in some way. This two-stage workflow poses a usability barrier because it requires users to make upfront decisions about what data to extract, rather than allowing them to incrementally extract data as they augment it.

  In this paper, we present a new, unified interaction model for web customization that encompasses both extraction and augmentation. The key idea is to provide users with a spreadsheet-like formula language that can be used for both data extraction and augmentation. We also provide a programming-by-demonstration (PBD) interface through which users can create data extraction formulas by clicking on elements in the website. This model allows users to naturally and iteratively move between extraction and augmentation.

  To illustrate our unified interaction model for web customization, we have implemented it as a browser extension called Joker. Through case studies of using Joker ourselves, we show that Joker can be used to customize many real-world websites. We also present a formative user study with five participants, which showed that people with a wide range of technical backgrounds can use Joker to customize websites, and also revealed some interesting limitations of our approach. Finally, we present a heuristic evaluation of our design using the Cognitive Dimensions framework, exploring why it promotes a more flexible web customization experience.

---

# Introduction {#sec:introduction}

Many websites on the internet do not meet the exact needs of all of their users. Because of this, millions of people use browser extensions and userscripts [@zotero-224; @2021f] to customize websites. However, these tools only allow for pre-built customizations. End-user web customization systems like Sifter [@huynh2006], Vegemite [@lin2009] and Wildcard [@litt2020; @litt2020b] provide a more flexible approach, allowing anyone to create bespoke customizations without traditional programming.

These tools each provide different useful mechanisms for end-user customization, but they share a common design limitation: they have a rigid separation between the two stages of the web customization process. First, in the _extraction_ or _scraping_ phase, users get structured data from the website into a tabular format. Second, in the _augmentation_ phase, users perform augmentations like adding new columns derived from the data, or sorting the table. For example, in Vegemite, a user can extract a list of addresses from a housing catalog, and then augment the data by computing a walkability score for each address.

This separation between extraction and augmentation is an important barrier to usability. A user study [@lin2009] of Vegemite wrote that “it was confusing to use one technique to create the initial table, and another technique to add information to a new column.” The creators of Sifter similarly reported [@huynh2006] that “the necessity for extracting data before augmentation could take place was poorly understood, if understood at all.” In Wildcard, end-users cannot even augment a website unless a programmer has written and shared extraction code for that website in Javascript. This interaction model _requires_ a sequential workflow in which users first extract all the data they need, and then perform all their desired augmentations. In a paper titled "Formality Considered Harmful" [@shipman1999], Shipman et al discuss how imposing such formalisms can make using interactive systems difficult as can be seen from the mentioned Sifter and Vegemite user studies.

In this paper, we present a new approach to web customization that combines extraction and augmentation into a unified interaction model. Our key idea is to develop a domain specific language (DSL) that encompasses both extraction and augmentation tasks, along with a programming-by-demonstration (PBD) interface that makes it easy for end-users to program in the language. This unified interaction model allows end-users to seamlessly move between extraction and augmentation, resulting in a more iterative and free-from workflow for web customization.

To illustrate our unified interaction model for web customization, we have built a web extension called Joker, which is an extension of Wildcard. The original Wildcard system adds a spreadsheet-like table to a website and establishes a bidirectional synchronization between it and the table. This allows users to customize a website through interactions with the table like sorting, as well as augmenting the website with new data using a formula language. However, as mentioned earlier, all of Wildcard’s augmentation capabilities are separate from its extraction capabilities, which can only be performed via programming in Javascript.

Joker adds two key features to Wildcard:

**A unified formula language for extraction & augmentation**: Wildcard’s formula language only supported primitive values like strings and numbers aimed at augmentation. Joker extends this language by introducing Document Object Model (DOM) elements as a new type of value, and adding a new set of formulas for performing operations on them. This includes querying elements with Cascading Stylesheet Selectors (CSS) selectors and traversing the DOM. With this approach, a single formula language is used to express both extraction and augmentation tasks, even within a single formula expression.

**A PBD interface for creating extraction formulas**: Directly writing extraction formulas can be challenging for end-users, so Joker also implements a PBD interface that uses demonstrations to synthesize extraction formulas. Demonstrations have been used to synthesize data extraction programs in a number of prior systems. However, a key property of our design is that the extraction program synthesized from the demonstration is made visible to the user as a spreadsheet formula that can be edited and executed using pure functional semantics.

[@sec:examples] describes a concrete scenario, showing how Joker enables a user to complete a useful customization task. In [@sec:implementation], we elaborate on the implementation of our formula language, as well as the algorithm used by our PBD interface. Then, in [@sec:design-principles], we describe some of the broader design principles that Joker embodies and discuss their applications in other contexts.

We have performed three evaluations of our approach, presented in [@sec:evaluation]. First, we describe a suite of case studies in which we have used Joker to extract and augment various websites in order to characterize its capabilities and limitations. Second, we describe a formative user study with five participants, which showed that users were generally able to use Joker to perform useful extraction and augmentation tasks. However, it also uncovered limitations, particularly for less experienced users trying to extract data from more complex websites. Lastly, we perform a heuristic evaluation of our tool, using the Cognitive Dimensions [@blackwell2001] framework to analyze our design.

Joker relates to existing work not only in end-user web customization, but also in end-user web scraping and program synthesis which we discuss in [@sec:related-worker]. Finally, we discuss opportunities for future work in [@sec:conclusion].

# Example Usage Scenario {#sec:examples}

Here is an example scenario, illustrated in [@fig:ebay], and demonstrated in the Video Figure accompanying this paper.

Jen is a web user searching for a karaoke machine on eBay, a shopping website. She wants to sort products by price within a page of search results, and can achieve this customization using Joker.

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/ebay.png}
  \caption{\label{fig:ebay}Scraping and customizing eBay by unified demonstration and formulas.}
\end{figure*}
</div>

*Extracting Product Names & Prices By Demonstration*. (@fig:ebay Part A): Jen initiates Joker through the browser context menu. As she hovers over the product names, Joker provides two kinds of live feedback. First, it highlights the names of all the listings on the page, to indicate how it has generalized Jen's intent based on her demonstration of a single example product. Second, a column of the table is populated with the values that will be extracted, giving a preview of how the extracted data would look. To commit the extraction, Jen clicks on the product name. She repeats a similar process for product prices.

When she clicks one of the cells in column `B`, the formula bar above the table display the extraction formula that was generated from her demonstration, and produces the values in column `B`:

`QuerySelector(rowElement, "span.s-item__price")`

For each row in the table, Joker has a created a reference to the corresponding DOM element on the website, accessible through the special variable `rowElement`. The `QuerySelector` function runs the specified CSS selector (`span.s-item__price`) within the `rowElement` to extract the price of each item. While Jen could edit this formula if needed, in this case the values in the table look correct, so there's no need to edit this formula any further.

*Sorting Products By Price*. (@fig:ebay Part B): Next, Jen wants to sort the table by price, but she realizes that the column of prices is a list of strings containing the `$` symbol. She needs to lightly _augment_ this raw data by turning the strings into numbers. In the next column `C`, Jen starts typing a formula, and uses an autocomplete dropdown to find a relevant function that extracts numeric values from strings:

`=ExtractNumber(B)`

Now column C contains numeric values, so Jen can sort the table by the column of prices. Because the table is synchronized with the website, the page becomes sorted as well.

*Extracting Product URL With Formulas* (@fig:ebay Part C): While completing the previous task, Jen realizes there is another customization she would find useful: prioritizing products for which she has not yet visited the details page for the listing. To do this, she must first return to _extracting_ more relevant data from the page.

Each listing links to a page for the specific item, but because the URL is not visible in the page, it's not possible to directly scrape this value by demonstration. However, Jen can still achieve the goal by directly writing an extraction formula.

Jen has some basic knowledge of HTML, which she can leverage to write the formula. She opens the browser developer tools, and observes that the listing title is represented by a link tag with a heading inside:

```
<a href="LISTING PAGE URL">
  <h3>LISTING NAME</h3>
</a>
```

Since there is already a column `A` in the table representing the product's title, she can use this as a starting point to write the formula:

`=GetAttribute(GetParent(A), "href")`

This formula first calls the `GetParent` function on column `A`, which traverses up one level in the DOM to the `a` element. Then, it uses the `GetAttribute` function to retrieve the value of the `href` attribute from that element. After running this formula, the table contains a column with the URL for each product.

*Sorting Products By Whether They Have Been Visited* (@fig:ebay Part C): After performing that extraction, Jen can write a final augmentation formula to indicate whether she has visited the corresponding product page:

`=Visited(D)`

The `Visited` function checks whether a URL is present in the browser history and returns a boolean value. Jen can then sort by this column to put listings that she has not yet visited at the top of the page.

Using Joker, Jen was able to not only achieve her initial customization goal to sort the products by price but also perform a customization she did not plan to do. This was made possible by Joker's unified interaction model for web customization which enabled her to interleave extraction and augmentation. While we have described a single illustrative scenario in this example, Joker is flexible enough to support a wide range of other useful customizations and workflows, described in more detail in [@sec:evaluation].

# System Implementation {#sec:implementation}

<div class="pdf-only">
\begin{figure*}
  \includegraphics[width=\textwidth]{media/overview.png}
  \caption{\label{fig:overview}An overview of Joker's interaction model and wrapper induction process.}
\end{figure*}
</div>

In this section, we describe Joker's formula language in more detail. Then, we outline the *wrapper induction* [@kushmerick2000] algorithm that Joker's PBD interface uses to synthesize the row and column selectors presented in formulas. [@fig:2] illustrates the entire process.

## Extraction Formulas

The Wildcard customization tool includes a formula language for augmentation, including operators for basic arithmetic and string manipulation, as well as more advanced operators that fetch data from web APIs. As in other tabular interfaces like SIEUFERD [@bakke2016] and Airtable [@2021f], formulas apply to a whole column at a time rather than a single cell, and can reference other columns by name.  Joker extends this base language with new constructs which enable it to apply to _data extraction_ instead of just augmentation.

We added DOM elements as a data type in the language, alongside strings, numbers, and booleans. Because the language runs in a JavaScript interpreter, we simply use native JavaScript values to represent DOM elements in the language. DOM elements are displayed visually by showing their inner text contents. They can also be implicitly typecast to strings for use in other formulas; for example, a string manipulation formula like `Substring` can be called on a DOM element value, and will operate on its text contents.

We also added several functions to the formula language for traversing the DOM and performing extractions, summarized below with their types:

- `QuerySelector(el: Element, sel: string): Element`. Executes the CSS selector `sel` inside of element `el`, and returns the first matching element.
- `GetAttribute(el: Element, attribute: string): string`. Returns the value for an attribute on an element.
- `GetParent(el: Element): Element`. Returns the parent of a given element.

To extract data from a row, formulas need a way to reference the current row, so we added a construct to support this use case. Every row in the table maps to one DOM element in the page; we allow formulas to access this DOM element via a special keyword, `rowElement`. In some sense, `rowElement` can be seen as a hidden extra column of data in the table containing DOM elements.

While many more functions could be added to expose more of the underlying DOM API in our language, we found that in practice these three functions provided ample power through composition. For example, in [@sec:examples] we showed how `GetParent` and `GetAttribute` can be composed to traverse the DOM and extract the URL associated with a listing (`=GetAttribute(GetParent(...), "href")`).

Joker's unified formula language for extraction and augmentation embodies two key design principles: **Unified User Interaction** and **Functional Reactive Programming**. By provinding a single formula language to express extractions and augmentations, Joker enables a unified user interaction model that supports interleaving the two. Furthermore, the formula language enables users to specify logic using pure, stateless functions that automatically update in response to upstream changes. This makes programming using Joker much easier and has famously been used by millions of end-users in spreadsheet formula languages (Microsoft Excel & Google Sheets), and end-user programming environments (Microsoft Power Fx [@2021g], Google AppSheet [@2021h], Airtable [@2021f], Glide [@2021a], Coda [@2021c] & Gneiss [@chang2014]).

## Wrapper Induction

Wrapper induction is the task of generalizing from a few examples of data to a specification for the entire data set. When users demonstrate a column value on a website through Joker's PBD interface, the demonstrated column value is fed into our wrapper induction algorithm. The result is an editable extraction formula that extracts all the corresponding column values. This use of PBD to generate editable code embodies a design principle known as **Prodirect Manipulation**. The term was coined by Ravi Chugh in a position paper [@chugh2016a] in which he advocates for “novel software systems that tightly couple programmatic and direct manipulation.” In a similar fashion, Sketch-N-Sketch [@chugh2016] provides prodirect manipulation by allowing users to create an SVG shape via traditional programming and then switch to modifying its size or shape via direct manipulation.

Joker takes an approach to wrapper induction similar to that used in systems like Vegemite [@lin2009] and Sifter [@huynh2006]. It synthesizes a single *row selector* for the website: a CSS selector that identifies a set of DOM elements corresponding to the rows of the data table. For each column in the table, it synthesizes a *column selector*, a CSS selector that identifies the element containing the column value. We proceed to describe the criteria used to determine row elements and then describe the criteria used to synthesize CSS selectors for row and column elements.

### Determining Row Elements

Given a demonstrated column value, Joker uses the following criteria to determine the row element:

*Plausibility*. An element `R` is a plausible row element if 1) it is in the parent path of the column element `C` containing the demonstrated column value `V` and 2) the CSS selector `S` of element `C` only identifies `C` when applied to `R`.

*Weight*. A row element `R` has a weight `W` equal to the number of its siblings for which the CSS selector `S` of column element `C` only identifies a single element. This ensures that we favor row elements that will result in the highest number of rows in the resulting data table.

*Best*. A row element `R` is the best if it is plausible and there is no other row element that has a higher weight than it. If there are multiple plausible row elements with the highest weight, we pick the one closet to the column element `C` in its parent path.

### Synthesizing CSS Selectors

A CSS selector is set of classes that can be used to identify an element. Given a row element, Joker synthesizes its row selector using the following criteria:

*Plausibility*. A selector $S_{row}$ is a plausible row selector if it consists of a subset of the classes on the row element.

*Weight*. $S_{row}$ has a weight equal to the number of classes it consists of. We favor selectors with lower weights to ensure that only the minimum required classes are utilized

*Best*. $S_{row}$ is the best if it is plausible and there is no other selector that has a lower weight than it. If there are multiple selectors that are plausible and have the lowest weight, we only pick one.

Given a column element, Joker synthesizes its column selector using the following criteria:

*Plausibility*. A selector $S_{column}$ is a plausible column selector if it only selects the given column element when applied on the corresponding row element.

*Weight*. $S_{column}$ has a weight equal to the number of classes it consists of. As before, we favor selectors with lower weights.

*Best*. $S_{column}$ is the best if it is plausible and there is no other selector that has a lower weight than it has.

# Evaluation {#sec:evaluation}

We evaluate our interaction model and tool in terms of three research questions:

**RQ1: What kinds of websites can this model operate effectively on?** We evaluate this with a suite of case studies from using Joker ourselves, which demonstrate its capabilities and limitations.

**RQ2: How are users of different backgrounds able to use the system?** We evaluate this with a small formative user study with users of different backgrounds.

**RQ3: What are the essential design dimensions that distinguish this model from other approaches?** We evaluate this with a heuristic analysis using the Cognitive Dimensions of Notation framework.

## Case Studies

Following a method used to evaluate visualizations through a diverse gallery of examples [@ren2018], our first evaluation of Joker provides an case studies of popular websites on which Joker can be used for web customization and on which it fails. For the websites on which Joker can be used, we provide the sequence of interactions needed to achieve the customizations. For the websites on which Joker fails, we provide an explanation.

### Successful applications
<!-- <div class="pdf-only">
```{=latex}
\begin{table}
\hypertarget{tab:examples}{%
\centering
\includegraphics[width=\columnwidth]{media/examples_table.png}
\caption{A gallery of website customizations that we have achieved using Joker. End users can perform these customizations without writing any JavaScript.}\label{tab:examples}
}
\vspace{-0.9cm}
\end{table}
```
</div> -->

<div class="pdf-only">
```{=latex}
\begin{table*}[]
\centering
\begin{tabular}{|l|l|}
\hline
\textbf{Website}              & \textbf{Example Customization Achieved by Joker}                                        \\ \hline
eBay, Amazon            & Filter listings by whether they have a "Sponsored" label.                        \\
Amazon, Target & Sort search results by price and rating.                                                \\
Google Scholar                & Filter publications for those whose title contains a keyword. \\
Reddit, CNN, ABC  & Sort by the read times of articles. Filter already-visited articles.         \\
Weather.com                   & Filter hourly weather to find nice times of day.                                        \\
Github                        & Sort a user's code repositories by stars to find popular work.                          \\
Postmates, Uber Eats    & Sort restaurants by delivery time and delivery fee.                                     \\ \hline
\end{tabular}
\vspace{8pt}
\caption{Examples of websites that Joker can be used to customize, including extraction and augmentation}
\end{table*}
````
</div>

We have used Joker to achieve a variety of purposes across many popular websites. Several of the compelling examples we found are summarized in Table 1. We will now walk through the examples in the first three rows of the table to showcase how Joker's suite of formulas can be used for diverse extraction and augmentation tasks.

In the first example, we used Joker to sort search results by price within the Featured page on Amazon. (Using Amazon's sort by price feature often returns irrelevant results.) In Amazon's source code, the price is split into three HTML elements: the dollar sign, the dollar amount, and the cents amount. A user can only scrape the cents element by demonstration into column `A`. However, because the parent element of the cents element contains all three of the price elements, the user can scrape the full price using the formula `GetParent(A)`. Next, the user can write the formula `ExtractNumber(B)` to convert the string into a numeric value. Finally, the user can sort this column by low-to-high prices. In a similar manner, we have used Joker to scrape and sort prices and ratings on the product listing pages of Target and eBay.

We have also found Joker to be useful for filtering based on text inputs. For example, we have used Joker to filter the titles of a researcher's publications on their Google Scholar profile. Specifically, a user can first scrape the titles into column A by demonstration. Then, the user can write the formula `Includes(A, "compiler")` that returns whether or not the title contains the keyword "compiler". Finally, the user can sort by this column to get all of the publications that fit their constraint at the top of the page. We have also used Joker to filter other text-based directory web pages such as Google search results and the MIT course catalog, in similar ways.

Additionally, we have used Joker to augment web pages with external information. For example, Joker can augment Reddit's old user interface, which has a list of headlines with links to articles and images. A user can first scrape the headline elements into column A by demonstration. The user can then extract the link into column B with the formula `GetAttribute(A, "href")`. Then, the user can write the formula `ReadTimeInSeconds(B)` that calls an API that returns the links' read times. Similarly, the user can write the formula `Visited(B)` that returns whether that link has been visited in the user's browser history. The user can also scrape elements such as the number of comments and the time of posting and sort by these values. We have performed similar customizations on websites such as ABC news.

### Limitations
Joker is most effective on websites with data that is presented as many similarly-structured HTML elements. However, certain websites have designs that make it difficult for Joker to scrape data. These are some of those designs:

- *Multiple row elements.* The layout of some web pages has multiple types of row as siblings that contain different children elements. For example, the news aggregator website HackerNews has a page design that alternates between rows containing a title and rows containing supplementary data (e.g. number of likes and the time of posting). Because Joker only chooses a single row selector, when scraping by demonstration, Joker will only select one of the types of rows, and elements in the other types of rows will not be able to be scraped.
- *Infinite scroll.* Some web pages have an "infinite scroll" feature that adds new entries to the page when a user scrolls to the bottom. Joker's table will only contain elements that were rendered when the table was first created. Additionally, for websites with many elements, such as Facebook, Joker might run out of memory while running its wrapper induction algorithm and crash the page.
- *Data hidden behind an interaction.* On some sites, a user must click on an element to reveal data corresponding to that entry (e.g. time of posting, the author). However, Joker is restricted to scraping what is visible on the page at one point in time.

## User Study

We conducted a small formative user study to understand how people would interact with Joker.

### Participants

We recruited 5 participants with varying backgrounds. 3 participants were familiar with spreadsheet formulas. 3 participants had extensive web development experience, 1 had a small amount of prior experience, and 1 had no experience. 3 participants had previously extracted data from websites.

### Protocol

The participants completed 7 web customization tasks across 2 websites. All participants attempted all the tasks.

First, we asked participants to customize a website with a relatively simple HTML structure: the MIT EECS course catalog website. All data _extraction_ on this site can be performed with demonstrations alone in Joker, although augmentation still requires writing formulas. The specific tasks were the following: 1a) Extract course titles, 1b) Extract course prerequisites, 1c) Add a column that indicates whether a course has a prerequisite & 1d) Add a column that indicates whether a course has no prerequisites and is offered in the fall term.

Next, we asked participants to customize a website with a more complex HTML structure: the search results page for the eBay shopping website. Due to the page's complexity, demonstrations alone are not sufficient to extract data from this website in Joker; users must also directly edit extraction formulas.  The specific tasks were the following: 2a) Extract title from iPhone listings, 2b) Extract iPhone listing price, & 2c) Create a column that indicates whether an iPhone listing is sponsored.

Each session was 60 minutes long and conducted over a recorded videoconference. We started each session with a description of Joker and provided a brief tutorial of its main features on a sample website not used in the tasks. There was no time limit for completing the tasks. Users were encouraged to speak aloud as they worked.

Because some of the tasks build on results of previous tasks, we wanted to ensure all participants made enough progress to gather useful feedback. Therefore, whenever a participant got stuck for several minutes, we recorded why they were stuck and then offered hints for how to proceed (such as suggestions to read formula documentation or open the browser dev tools). While all participants were able to complete all tasks with hints, this obviously does not mean they could have completed the task unassisted. Our goal was not to simply measure whether users completed the task, but rather to gain qualitative insight into the barriers they faced.

### Results

Most participants took advantage of the unified interaction model to interleave extraction and augmentation tasks, rather than performing all extraction up front. For example, on task 1, most participants extracted the prerequisite status by demonstration, added one or more columns to the table to perform some string operations on the prerequisite status, and then continued on to extract more information from the web page by demonstration. Furthermore, we hypothesize that in a less controlled setting, users would be even more likely to interleave extraction and augmentation, since the task may be less well defined at the beginning.

One usability issue with the unified model was that participants sometimes got confused about how their demonstrations would affect the contents of the table. For example, multiple participants intended to add a new column by demonstrating an extraction, but instead accidentally overwrote the contents of an existing column. This poses a design challenge because the user's demonstrations occur in the web page, so they cannot be directly interacting with the table while demonstrating; this suggests that the interface needs to do a better job indicating where the results of a demonstration will be inserted in the table.

On the relatively simple MIT course catalog website, all participants were able to extract the relevant data from the page within seconds, simply performing demonstrations with a few clicks. This suggests that when Joker's generalization algorithm works well, it can be an effective tool for data extraction, even for users with limited programming experience. P1, who had no prior web development experience, said: *"you could hover and [the data] was already selected...that was very nice"*. P3, upon seeing the tutorial for extraction by demonstration, said "that's like black magic."

On the more complex eBay website where demonstration alone was not sufficient, results were more varied. P1 struggled to complete the task, saying that *"looking at HTML is a bit much"*; this suggests that more work could be done to make the experience usable for complete novices. However, users with more web development experience were able to use the tool to perform more complex extractions, such as directly writing CSS query selectors into the formula bar. P2 and P3 both reported that Joker's live feedback loop was easier to use and faster than other approaches to web extraction; P3 noted that *"[with any other approach], it would have been slower to specify and slower to validate that I specified it correctly"*

It was challenging for some participants to switch between using the browser's developer tools and the Joker interface when doing complex extraction tasks. While we chose not to build HTML inspection into the Joker UI because the browser already provides a very rich set of tools, users sometimes were not able to tell how elements in the Joker table corresponded to elements in the browser's element inspector.

In general, participants were able to learn the formula language by using an autocomplete dropdown with inline documentation, which we developed as part of the Joker extension. In some cases, participants were able to immediately construct correct formulas on the first try; in other cases it took several attempts and some hints from the moderator to try a relevant function. While better documentation and error messages could help improve the learnability of the formula language, we also did not find it surprising that participants required some time to learn a completely unfamiliar formula language.

## Cognitive Dimensions Analysis

Our third evaluation of Joker analyzes it using Cognitive Dimensions of Notation [@blackwell2001], a heuristic evaluation framework that has been used to evaluate various programming languages and visual programming systems [@satyanarayan2014; @satyanarayan2014a; @ledo2018]. When contrasting our tool with traditional scraping and other visual tools, we find particularly meaningful differences along several of the dimensions:

*Progressive evaluation.* In Joker, a user can see the intermediate results of their extraction and augmentation work at any point, and adjust their future actions accordingly. The table user interface makes it easy to inspect the intermediate results and notice surprises like missing values.

In contrast, traditional extraction typically requires editing the code, manually re-running it, and inspecting the results in an unstructured textual format, making it harder to progressively evaluate the results. Also, many end-user extraction tools [@chasins2018; @lin2009] require the user to demonstrate all the data extractions they want to perform before showing any information about how those demonstrations will generalize across multiple examples.^[This is an instance where Joker's limitation of only scraping a single page at a time proves beneficial. All the relevant data is already available on the page without further network requests, making it possible to support progressive evaluation with low latency. Other tools that support scraping across multiple pages necessarily require a slower feedback loop.]

*Premature commitment.* Many scraping tools require making a _premature commitment_ to a data schema: first, the user extracts a dataset, and then they perform augmentations using that data. Wildcard suffered from this problem: when writing code for extraction, a user would need to anticipate all future augmentations and extract the necessary data.

Joker instead supports extracting data _on demand_. The user can decide on their desired augmentations and extract data as needed to fulfill those tasks. There is never a need to eagerly guess what data will be needed in advance.

We have also borrowed a technique from spreadsheets for avoiding premature commitment: default naming. New data columns are automatically assigned a single-letter name, so that the user does not need to prematurely think of a name before extracting the data. We have not yet implemented the capability to optionally rename demonstrated columns, but it would be straightforward to do so, and would provide a way to offer the benefits of names without requiring a premature commitment.

*Provisionality.* Joker makes it easy to try out an extraction action without fully committing to it. When the user hovers over any element in a website, they see a preview of how that data would be extracted into the table, and then they can click if they'd like to proceed. This makes it feel very fast and lightweight to try scraping different elements on the page.

*Viscosity.* Some scraping tools have high viscosity: they make it difficult to change a single part of am extraction specification without modifying the global structure. For example, in Rousillon [@chasins2018], changing the desired demonstration for a single column of a table requires re-demonstrating all of the columns. In contrast, Joker allows a user to change the specification for a single column of a table without modifying the others, resulting in a lower viscosity in response to changes.

*Role-expressiveness.* One dimension we are still exploring in Joker is role-expressiveness: having different elements of a program clearly indicate their role to the user. In particular, in our current design, the visual display of references to DOM elements in the table is similar to the display of primitive values. In our experience, this can sometimes make it difficult to understand which parts of the table are directly interacting with the website, vs. augmenting the extracted data. In the future we could consider adding more visual differentiation to help users understand the role of different parts of their customization.

# Related Work {#sec:related-work}

Joker builds on existing work in end-user web customization, end-user web scraping  and program synthesis by a number of systems.

## End-user Web Customization

Vegemite [@lin2009] is a tool for end-user programming of mashups. Unlike Joker, its table interface is only populated and can only be interacted with after all the demonstrations have been provided. This does not support interleaving of extractions and augmentations to achieve an incremental workflow for users. Mashups are created using CoScripter [@leshed2008] which records operations on the extracted values in the table for automation tasks. CoScripter provides the generated automation program as text-based commands, such as “paste address into ‘Walk Score’ input”, which can be edited via “sloppy programming” [@lin2009] techniques. However, this editing does not extend to the synthesized extraction program which is not displayed and therefore cannot be modified. This means that users can only scrape what can be demonstrated. In Joker, a formula synthesized from a demonstration can be edited to achieve the desired task.

Sifter [@huynh2006] is a tool that augments websites with advanced sorting and filtering functionality. It attempts to automatically detect items and fields on the page with a variety of clever heuristics. If this falls, it gives the user the option of demonstrating to correct the result. In contrast, Joker is simpler and makes fewer assumptions about the structure of websites by giving control to the user from the beginning of the process and displaying the synthesized program which can be modified. We hypothesize that focusing on a tight feedback loop rather than automation may support a scraping process that is just as fast as an automated one, offers more expressive extraction and extends to a greater variety of websites. However, further user testing is required to validate this hypothesis.

## End-user Web Scraping

FlashExtract [@le2014] is a programming-by-example tool for data extraction. In addition to demonstrating whole values, it supports demonstrating substrings of values. Joker only supports this through augmentation formulas. This is not as end-user friendly but allows for a wider range of operations such as indicating whether demonstrated values contain a certain value or are greater than or less than a certain value.

Rousillon [@chasins2018] is a tool that enables end-users to scrape distributed, hierarchical web data. It presents the web extraction program generated from demonstrations as an editable, high-level, block-based language called Helena [@2021c]. While Helena can be used to specify complex web scraping tasks like adding control flow, it does not present the synthesized web extraction program. This means that users can only extract what can be demonstrated. Joker on the other hand displays the synthesized program as an extraction formula which can be modified to increase the expressiveness of extraction.

## Program Synthesis

FlashProg [@mayer2015] is a framework that provides program navigation and disambiguation for programming-by-example tools like FlashExtract [@le2014] and FlashFill [@harris]. The program viewer provides a high level description of synthesized programs as well as a way to navigate the list of alternative programs that satisfy the demonstrations. This is important because demonstrations are an ambiguous specification for program synthesis [@peleg2018]: the set of synthesized programs for a demonstration can be very large. To further ensure that the best synthesized program is arrived at, FlashProg has a disambiguation viewer that asks the user questions in order to resolve ambiguities in the user's demonstrations. In contrast, Joker only presents the top-ranked row and column selectors which may not be the best ones. Furthermore, Joker's formulas utilize CSS selectors to specify data extractions. CSS selectors allow for more expressive data extraction than demonstrations but are not end-user friendly.

# Conclusion And Future Work {#sec:conclusion}

In this paper, we presented a unified interaction model for web customization through a browser extension called Joker. The model combines data extraction and augmentation using a single DSL and a PBD interface that makes it easy for end-users to program in the DSL. The DSL consists of a spreadsheet formula language that offers pure functional semantics for expressing and executing data extractions and augmentations.

The main area of future work involves providing feedback to users about when demonstrations are not sufficient to create extraction formulas. Otherwise, users won't know to directly edit the synthesized formula. FlashProg [@mayer2015] and Wrangler [@kandel2011], whose interfaces also have tables, offer clues for how to provide feedback about the execution of programs synthesized from demonstrations. Another important area involves making the formula language more accessible to end-users not familiar with CSS selectors. Again, FlashProg offers some clues through its use of high level descriptions to give meaning to synthesized programs. One avenue for associating meaning with CSS selectors could be by extracting semantic web content as seen in systems like Thresher [@hogue2005].
