# Developer Documentation Workflow <a id="techdocsworkflow"></a>

Developers are the primary authors of technical documentation. They are responsible for documenting technical content on their pack's backend services, applications, and other service-adjacent informational content.

## In this article

**[When to write documentation](#whentowrite)**

**[Mapping documentation requirements](#docreq)**

**[Drafting technical content](#drafting)**

**[Requesting documentation review](#requestingreview)**

---
## When to write documentation <a id="whentowrite"></a>
Ultimately, it is the pack's responsibility to decide what and when to document, though we recommend creating documentation when:

- A new or enhanced service is shipped that impacts the developer or administrator experience.
- A process, workflow, or previously documented service is changed.
- There are changes to the user interface or API.
- A service is deprecated or removed.

[back to top](#techdocsworkflow)

---

## Mapping documentation requirements <a id="docreq"></a>
Documentation requirements are essential pieces of information necessary for the document to be considered ready for review. Coordinate with the DRI and Technical Writer to determine the necessary documentation requirements, and include it in the JIRA issue within the description.

Guiding questions:

- What concepts or procedures need to be documented to communicate the changes made to the service or application?
- To this end, what new page(s) are needed, if any? What pages or subsections need updates? Consider changes and additions to user, admin, and API documentation.
- Do we need to update a previously recommended workflow? Should we link the new feature from various relevant locations? Consider all ways documentation should be affected.
- Include suggested titles of any pages or subsection headings, if applicable.
- List any documentation that should be cross-linked, if applicable.

[back to top](#techdocsworkflow)

---

## Drafting technical content <a id="drafting"></a>
Developers are responsible for drafting documentation for review. Use the documentation requirements to guide what information you include and the documentation resources to guide how you present the information.

When drafting a document, keep in mind the following:

- Write/update documentation in Markdown file, locally.
- Unless a README.md, store in subdirectory labeled, `docs/`.
- Use documentation guidelines and other resources to guide how you present information.
    - [Documentation Structure and template page](https://bitbucket.org/fetchrewards/documentation/src/master/docs/Documentation_Structure.md).
    - [Style Guide](https://bitbucket.org/fetchrewards/documentation/src/master/docs/Style_Guide.md) (WIP).
    - [Markdown Guide](https://bitbucket.org/tutorials/markdowndemo/src/master/).
- Use documentation requirements to inform what information you include in the document.
- Contact the Technical Writer by adding them to the issue or mention them in `#documentation` on Slack, if you:
    - Need any help to choose the correct place for documentation.
    - Want to discuss a documentation idea or outline.
    - Want to request any other help at any time throughout the documentation process. 

[back to top](#techdocsworkflow)

---

## Requesting documentation review <a id="requestingreview"></a>
All new and/or updated service documents should be reviewed prior to publication and distribution. 

In order to request review:

- Include any new and/or updates documentation, either in:
    - The merge request introducing the code.
    - A separate merge request raised around the same time.
- Add the Technical Writer to the merge request as a "Reviewer". 

[back to top](#techdocsworkflow)

---