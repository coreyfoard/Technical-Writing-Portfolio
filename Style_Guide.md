# Documentation Style Guide <a id="docsstyleguide"></a>

This document defines the standards for Fetch documentation, including grammar, formatting, word use, and more.

For style questions, mention a Technical Writer in an issue or merge request, or reach out in the `#documentation` channel.

## In this article

**[The Fetch voice](#voice)**

**[Documentation is the single source of truth](#ssot)**

**[Markdown](#markdown)**

**[Language](#language)**

**[Text](#text)**

**[Lists](#lists)**

**[Tables](#tables)**

**[Quotes](#quotes)**

**[Links](#links)**

**[Images](#images)**

**[Notes](#notes)**

**[Blockquotes](#blockquotes)**

**[Terms](#terms)**

---

## The Fetch voice <a id="voice"></a>

The voice in the Fetch documentation strives to be concise,
direct, and precise. The goal is to provide information that's easy to search and scan.

The voice in the documentation should be conversational but brief, friendly but succinct.

Please write the in [present tense](https://developers.google.com/style/tense) and use [active voice](https://developers.google.com/style/voice) as often as possible.

### Present tense

Write documentation as if your reader is actively trying to use your service, using the present tense.

At times, it may make sense to write using the future tense _will_, though try to write in the present, when possible. 

Past tense will sometimes make sense, such as instances when describing UI steps or procedures.

```
Present:

The service builds images.
```

```
Future:

The service will build images
```
### Active voice

In an active sentence construction, the subject (service) performs the verb (builds) on the object (images).
```
Active:

The service builds images.
```
In a passive sentence construction, the object (images) is acted on by the subject (subject).
```
Passive:

Images are built by the service.
```

[back to top](#docsstyleguide)

---

## Documentation is the single source of truth (SSOT) <a id="ssot"></a>

The Fetch service documentation is the SSOT for all information related to Fetch services, usage, and troubleshooting. It evolves continuously, in keeping with new products and features, and with improvements for clarity, accuracy, and completeness.

This policy prevents information silos, making it easier to find information about Fetch services.

It also informs decisions about the kinds of content we include in our documentation.

<!--### Topic types

In the software industry, it is a best practice to organize documentation in different types. 

[Topic types](https://bitbucket.org/fetchrewards/documentation/src/master/docs/Documentation_Structure.md) help structure technical content using repeatable patterns that make content easier to write and read.

Each topic on a page should be one of the following topic types:

- Concept
- Task
- Reference

Even if a page is short, the page usually starts with a concept and then includes a task or reference topic.

### Link instead of repeating text

Rather than repeating information from another topic, link to the single source of truth and explain why it is important.-->

### Docs-first methodology

We employ a documentation-first methodology. This method ensures the documentation remains a complete and trusted resource, and makes communicating about the use of Fetch services more efficient.

- If the answer to a question exists in documentation, share the link to the documentation instead of rephrasing the information.
- When you encounter new information not available in Fetch documentation (for example, when working on a support case or testing a feature), your first step should be to create a merge request (MR) to add this information to the documentation. You can then share the MR to communicate this information.

New information that would be useful toward the future usage or troubleshooting of Fetch services should not be written directly in a channel or other messaging system, but added to a documentation MR and then referenced, as described above.

The more we reflexively add information to the documentation, the more the documentation helps others efficiently accomplish tasks and solve problems.

If you have questions when considering, authoring, or editing documentation, ask
the Technical Writing team. They're available on Slack in `#documentation` or in Bitbucket by mentioning them. Otherwise, forge ahead with your best effort. It does not need to be perfect; the team is happy to review and improve upon your content.

[back to top](#docsstyleguide)

---

## Markdown <a id="markdown"></a>

All Fetch documentation is written using [Markdown](https://en.wikipedia.org/wiki/Markdown).

We use [CommonMark Markdown](https://commonmark.org/). The ducks-pipe takes this Markdown and converts it to HTML, so that it can render in Confluence Markup on our internal wiki.

### Heading levels in Markdown

Each documentation page begins with a level 1 heading (`#`). This becomes the `h1` element when the page is rendered to HTML. There can be only **one** level 1 heading per page.

- For each subsection, increment the heading level. In other words, increment the number of `#` characters in front of the topic title.
- Avoid heading levels greater than `H5` (`#####`). If you need more than five heading levels, move the topics to a new page instead.
- Do not skip a level. For example: `##` > `####`.
- Leave one blank line before and after the topic title.
- Separate `H2` headings with a line break (`---`).

### Backticks in Markdown

Use backticks for:

- [Code blocks](#code-blocks).
- Error messages.


<!--WIP ### Markdown Rules
Fetch ensures that the Markdown used across all documentation is consistent, as well as easy to review and maintain, by [testing documentation changes](../testing.md) with [markdownlint](../testing.md#markdownlint). This lint test fails when any document has an issue with Markdown formatting that may cause the page to render incorrectly in Fetch. It also fails when a document has non-standard Markdown (which may render correctly, but is not the current standard for Fetch documentation).-->

[back to top](#docsstyleguide)

---

## Language <a id="language"></a>

 Fetch documentation should be clear and easy to understand.

- Avoid unnecessary words.
- Be clear, concise, and stick to the goal of the topic.
- Write in US English with US grammar.
- Avoid using slang or shorthand. 

### Capitalization

In general, we tend to lean on sentence case for topic titles, and capitalize according standard US English grammar rules. 

#### Topic titles

Use sentence case for topic titles. For example:

- `# Use variables to configure pipelines`
- `## Use the To-Do List`

#### Other terms

Capitalize names of:

- Third-party organizations, software, and products. For example, Prometheus,
  Kubernetes, Git, and The Linux Foundation.
- Methods or methodologies. For example, Continuous Integration,
  Continuous Deployment, Scrum, and Agile.

If you are unsure of a when you should capitalize a letter, or the word uses non-standard case styles, feel free to reach out to a Technical Writer.

### Contractions

Contractions are fine, and can create a friendly and informal tone, especially in tutorials and instructional documentation.

Avoid contractions when:

| Do not use a contraction      | Example                                          | Use instead                                                      |
|-------------------------------|--------------------------------------------------|------------------------------------------------------------------|
| With a proper noun and a verb | The **Container Registry's** a powerful feature. | The **Container Registry** is a powerful feature.                |
| To emphasize a negative       | **Don't** install X with Y.                      | **Do not** install X with Y.                                     |
| In reference documentation    | **Don't** set a limit.                           | **Do not** set a limit.                                          |

### Possessives

Try to avoid using possessives (`'s`) for proper nouns, like organization or product names.

For example, instead of `Docker's CLI`, use `the Docker CLI`.

### Prepositions

Use prepositions at the end of the sentence when needed. Dangling or stranded prepositions are fine. For example:

- You can leave the group you're a member of.
- Share the credentials with people you want to give access to.

These constructions are more casual than the alternatives:

- You can leave the group of which you're a member.
- Share the credentials with the people to which you want to give access.

### Acronyms

If you use an acronym, spell it out on first use on a page. You do not need to spell it out more than once on a page.

- **Titles:** Try to avoid acronyms in topic titles, especially if the acronym is not widely used.
- **Plurals:** Try not to make acronyms plural. For example, use `YAML files`, not `YAMLs`. If you must make an acronym plural, do not use an apostrophe. For example, use `APIs`, not `API's`.
- **Possessives:** Use caution when making an acronym possessive. If possible,
  write the sentence to avoid making the acronym possessive. If you must make the
  acronym possessive, consider spelling out the words.

### Numbers

When using numbers in text, it is best practice to spell out zero through nine, and use numbers for 10 and greater.

[back to top](#docsstyleguide)

---

## Text <a id="text"></a>

- Write in [Markdown](#markdown).
- Splitting long lines (preferably up to 100 characters) can make it easier to provide feedback on small chunks of text.
- Insert an empty line for new paragraphs.
- Insert an empty line between different markups (for example, after every paragraph, header, list, and so on). Example:

  ```markdown
  ## Header

  Paragraph.

  - List item 1
  - List item 2
  ```

### Comments

To embed comments within Markdown, use standard HTML comments that are not rendered when published. Example:

```html
<!-- This is a comment that is not rendered -->
```

### Emphasis

Use **bold** rather than italic to provide emphasis. Use emphasis sparingly.

You can use italics when you are introducing a term for the first time. Otherwise, use bold.

- Use double asterisks (`**`) to mark a word or text in bold (`**bold**`).
- Use underscore (`_`) for text in italics (`_italic_`).
- Use greater than (`>`) for blockquotes.

### Punctuation

Follow these guidelines for punctuation.

- End full sentences with a period.
- Use serial (Oxford) commas before the final **and** or **or** in a list of three or more items.

When spacing content:

- Use one space between sentences. 
- Do not use non-breaking spaces. Use standard spaces instead.

### Placeholder text

You might want to provide a command or configuration that uses specific values.

In these cases, use `< >` to call out where a reader must replace text with their own value.

For example:

```shell
cp <your_source_directory> <your_destination_directory>
```

### Inline code

To apply inline code style, wrap the text in a single backtick (`` ` ``). For example, `this is inline code style`.

### Code blocks <a id="code-blocks"></a>

Code block style separates code text from regular text. Use code block style for commands run in the command-line interface. Code block style is easier to copy and paste in a developer's terminal window.

To apply code block style, wrap the text in triple backticks (three `` ` ``) and add a syntax highlighting hint. For example:

````plaintext
```plaintext
This is codeblock style
```
````

When using code block style:

- Use quadruple backticks (four `` ` ``) to apply code block style when the code block you are styling has triple backticks in it. For example, when illustrating code block style.
- Add a blank line above and below code blocks.

[back to top](#docsstyleguide)

---

## Lists <a id="lists"></a>

- Use a period after every sentence, including those that complete an introductory phrase.
- Majority rules. Use either full sentences or all fragments. Avoid a mix.
- Always start list items with a capital letter.
- Separate the introductory phrase from explanatory text with a colon (`:`). For example:

  ```markdown
  You can:

  - Do this thing.
  - Do this other thing.
  ```

### Choose between an ordered or unordered list

Use ordered lists for a sequence of steps. For example:

```markdown
Follow these steps to do something.

1. First, do the first step.
2. Then, do the next step.
3. Finally, do the last step.
```

Use an unordered lists when the steps do not need to be completed in order. For example:

```markdown
These things are imported:

- Thing 1
- Thing 2
- Thing 3
```

### List markup

- Use dashes (`-`) for unordered lists instead of asterisks (`*`).
- Start every item in an ordered list with `1.`. When rendered, the list items are sequential.
- Leave a blank line before and after a list.

### Nesting inside a list item

You can nest items under a list item, so they render with the same indentation as the list item. You can do this with:

- [Code blocks](#code-blocks)
- [Blockquotes](#blockquotes)
- [Notes](#notes)
- [Images](#images)

Nested items should always align with the first character of the list item. For unordered lists (using `-`), use two spaces for each level of indentation:

````markdown
- Unordered list item 1

  A line nested using 2 spaces to align with the `U` above.

- Unordered list item 2

  > A quote block that will nest
  > inside list item 2.

- Unordered list item 3

  ```plaintext
  a code block that nests inside list item 3
  ```

- Unordered list item 4

  ![an image that will nest inside list item 4](image.png)
````

For ordered lists, use three spaces for each level of indentation:

````markdown
1. Ordered list item 1

   A line nested using 3 spaces to align with the `O` above.
````

You can nest lists in other lists.

```markdown
1. Ordered list item one.
2. Ordered list item two.
   - Nested unordered list item one.
   - Nested unordered list item two.
3. Ordered list item three.

- Unordered list item one.
- Unordered list item two.
  1. Nested ordered list item one.
  2. Nested ordered list item two.
- Unordered list item three.
```

[back to top](#docsstyleguide)

---

## Tables <a id="tables"></a>

Tables should be used to describe complex information in a straightforward manner. Note that in many cases, an unordered list is sufficient to describe a list of items with a single, simple description per item. But, if you have data that's best described by a matrix, tables are the best choice.

### Creation guidelines

To keep tables accessible and scannable, tables should not have any empty cells. If there is no otherwise meaningful value for a cell, consider entering **N/A** for 'not applicable' or **None**.

To help tables be easier to maintain, consider adding additional spaces to the column widths to make them consistent. For example:

```markdown
| App name | Description          | Requirements   |
|:---------|:---------------------|:---------------|
| App 1    | Description text 1.  | Requirements 1 |
| App 2    | Description text 2.  | None           |
```

### Table headings

Use sentence case for table headings. For example, `Keyword value` or `Project name`.

[back to top](#docsstyleguide)

---

## Quotes <a id="quotes"></a>

Valid for Markdown content only, not for front matter entries:

- Standard quotes: double quotes (`"`). Example: "This is wrapped in double
  quotes".
- Quote inside a quote: double quotes (`"`) wrap single quotes (`'`). Example:
  "This sentence 'quotes' something in a quote".

[back to top](#docsstyleguide)

---

## Links <a id="links"></a>

Links help the docs adhere to the [single source of truth](#documentation-is-the-single-source-of-truth-ssot) principle.

### Links within the same repository

To link to another page in the same repository, use a relative file path. For example, `../user Fetch_com/index.md`.

Use inline link Markdown markup `[Text](https://example.com)`, rather than reference-style links, like `[Text][identifier]`.

### Links in separate repositories

To link to a page in a different repository, use an absolute URL. For example, to link from a page in the Fetch repository to the Charts repository, use a URL like `https://bitbucket.org/fetchrewards/documentation`.

### Text for links

Use descriptive text for links, rather than words like `here` or `this page.`

For example, instead of:

- `For more information, see [this page](LINK).`
- `For more information, go [here](LINK).`

Use:

- `For more information, see [merge requests](LINK)`.

### Links to external documentation

When possible, avoid links to external documentation. These links can easily become outdated, and are difficult to maintain.

Sometimes links are required. They might clarify troubleshooting steps or help prevent duplication of content. Sometimes they are more precise and will be maintained more actively.

For each external link you add, weigh the against the maintenance difficulties.

[back to top](#docsstyleguide)

---

## Images <a id="images"></a>

Images, including screenshots, can help a reader better understand a concept. However, they should be used sparingly because:

- They tend to become out-of-date.
- They are difficult and expensive to localize.
- They cannot be read by screen readers.

When needed, use images to help the reader understand:

- Where they are in a complicated process.
- How they should interact with the application.

### Capture the image

When you take screenshots:

- **Ensure it provides value.** Don't use `lorem ipsum` text.
  Try to replicate how the feature would be used in a real-world scenario, and use realistic text.
- **Capture only the relevant UI.** Don't include unnecessary white
  space or areas of the UI that don't help illustrate the point. The
  sidebars in Fetch can change, so don't include
  them in screenshots unless absolutely necessary.
- **Keep it small.** If you don't need to show the full width of the screen, don't.
  Reduce the size of your browser window as much as possible to keep elements close
  together and reduce empty space. Try to keep the screenshot dimensions as small as possible.
- **Review how the image renders on the page.** Preview the image locally or use the
review app in the merge request. Make sure the image isn't blurry or overwhelming.
- **Be consistent.** Coordinate screenshots with the other screenshots already on
  a documentation page for a consistent reading experience. Ensure your navigation theme
  is **Indigo** and the syntax highlighting theme is **Light**. These are the default preferences.

### Add the image link to content

The Markdown code for including an image in a document is:
`![Image description which will be the alt tag](img/document_image_title_vX_Y.png)`

[back to top](#docsstyleguide)

---

## Notes <a id="notes"></a>

Use notes to call attention to additional, non-essential information. Use them sparingly, and never use a note immediately following another note.

Instead of adding a note:

- Re-write the sentence as part of a paragraph.
- Put the information into its own paragraph.
- Put the content under a new topic title.

If you must use a note, use this format:

```markdown
NOTE:
This is something to note.
```

It renders on the Fetch documentation site as:

NOTE:
This is something to note.

[back to top](#docsstyleguide)

---

## Blockquotes <a id="blockquotes"></a>

For highlighting a text inside a blockquote, use this format:

```markdown
> This is a blockquote.
```

It renders on the Fetch documentation site as:

> This is a blockquote.

If the text spans multiple lines, you can split them.

For multiple paragraphs, use the symbol `>` before every line:

```markdown
> This is the first paragraph.
>
> This is the second paragraph.
>
> - This is a list item
> - Second item in the list
```

It renders on the Fetch documentation site as:

> This is the first paragraph.
>
> This is the second paragraph.
>
> - This is a list item
> - Second item in the list

[back to top](#docsstyleguide)

---

## Terms <a id="terms"></a>

To maintain consistency through Fetch documentation, use these styles and terms.

### Describe UI elements

Follow these styles when you're describing user interface elements in an service or application:

- For elements with a visible label, use that label in bold with matching case.
  For example, `Select **Cancel**`.
- For elements with a tooltip or hover label, use that label in bold with
  matching case. For example, `Select **Add status emoji**`.

[back to top](#docsstyleguide)

---