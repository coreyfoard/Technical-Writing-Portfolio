# Documentation Structure and Topic Types <a id="docstructure"></a>
This article discusses how we structure technical documentation using topic types, here at at Fetch.

Use topic types to help structure technical content using repeatable patterns that make content easier to write and read.

## In this article

**[What are topic types?](#topictypes)**

**[Concept topics](#concept)**

**[Task topics](#task)**

**[Reference topics](#reference)**

---

## What are topic types? <a id="topictypes"></a>

Topic types are small, self-contained sections of text with a predefined primary objective and structure.

We compose technical documents using a series of topic types.

We use topic types for three main reasons:

- **Content is hard to find.** Our docs are comprehensive and include a large amount of useful information. Topic types create repeatable patterns that make our content easier to scan and parse.
- **Content is often written from the contributor’s point of view.** Our docs are written by contributors. Topic types (tasks specifically) help put information into a format that is geared toward helping others, rather than documenting how a feature was implemented.
- **Content should be reusable.** Though our content is not currently reusable, the goal is for it to be soon. Building documentation with topic types, now, will better prepare us to make that transition in the future. 


Each topic on a page should be one of the following topic types:

- Concept
- Task
- Reference

Even if a page is short, the page usually starts with a concept and then includes a task or reference topic.

[back to top](#docstructure)

---

## Concept topics <a id="concept"></a>

A concept introduces a single feature or concept. A concept should answer the questions:

- What is this?
- Why would you use it?

Think of everything someone might want to know if they've never heard of this concept before. Don't tell them **how** to do this thing. Tell them **what it is**.

If you start describing another concept, start a new concept and link to it.

Concepts should be in this format:

```markdown
# Title (a noun, like "Widgets")

A paragraph that explains what this thing is.

Another paragraph that explains what this thing is.

Remember, if you start to describe about another concept, stop yourself.
Each concept should be about one concept only.
```

### Concept topic titles

For the title text, use a noun. For example, `Widgets` or `GDK dependency management`.

If a noun is ambiguous, you can add a gerund. For example, `Documenting versions` instead of `Versions`.

Avoid these topic titles:

- `Overview` or `Introduction`. Instead, use a more specific
  noun or phrase that someone would search for.
- `Use cases`. Instead, incorporate the information as part of the concept.
- `How it works`. Instead, use a noun followed by `workflow`. For example, `Merge request workflow`.

[back to top](#docstructure)

---

## Task topics <a id="task"></a>

A task gives instructions for how to complete a procedure.

Tasks should be in this format:

```markdown
# Title (starts with an active verb, like "Create a widget" or "Delete a widget")

Do this task when you want to...

Prerequisites (optional):

- Thing 1
- Thing 2
- Thing 3

To do this task:

1. Location then action. (Go to this menu, then select this item.)
1. Another step.
1. Another step.

Task result (optional). Next steps (optional).
```

Here is an example.

```markdown
# Create an issue

Create an issue when you want to track bugs or future work.

Prerequisites:

- You must have at least the Developer role for the project.

To create an issue:

1. On the top bar, select **Main menu > Projects** and find your project.
1. On the left sidebar, select **Issues > List**.
1. In the top right corner, select **New issue**.
1. Complete the fields. (If you have reference content that lists each field, link to it here.)
1. Select **Create issue**.

The issue is created. You can view it by going to **Issues > List**.
```

### Task topic titles

For the title text, use the structure `active verb` + `noun`.
For example, `Create an issue`.

If you have several tasks on a page that share prerequisites, you can use the title
`Prerequisites` and link to it.

### Task introductions

To start the task topic, use the structure `active verb` + `noun`, and
provide context about the action.
For example, `Create an issue when you want to track bugs or future work`.

To start the task steps, use a succinct action followed by a colon.
For example, `To create an issue:`

[back to top](#docstructure)

---

## Reference topics <a id="reference"></a>

Reference information should be in an easily-scannable format,
like a table or list. It's similar to a dictionary or encyclopedia entry.

```markdown
# Title (a noun, like "Pipeline settings" or "Administrator options")

Introductory sentence.

| Setting | Description |
|---------|-------------|
| **Name** | Descriptive sentence about the setting. |
```

### Reference topic titles

Reference topic titles are usually nouns.

Avoid these topic titles:

- `Important notes`. Instead, incorporate this information
  closer to where it belongs. For example, this information might be a prerequisite
  for a task, or information about a concept.
- `Limitations`. Instead, move the content near other similar information.
  If you must, you can use the title `Known issues`.

[back to top](#docstructure)

---
