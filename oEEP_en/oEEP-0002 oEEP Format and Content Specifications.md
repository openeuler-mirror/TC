---
Title:          oEEP Format and Content Specifications
Type:           Document
Abstract:       oEEP document standards and writing guide
Author:         Hu Xinwei <shinwell_hu@openeuler.sh>, wfg@mail.ustc.edu.cn
Status:         Provision
No:             oEEP-0002
Create_time:    2023-04-16
Update_time:    2023-05-24
---

## Motivation and Objectives

The introduction of oEEP documents aims to bring transparency, granularity, collaboration, and accumulation to the technological and community evolution of openEuler. The phased goals are as follows:

1. Early sharing
2. Clear description
3. Logical argumentation
4. Feedback collection
5. Review and coordination
6. Consensus building
7. Clear planning
8. Implementation guidance
9. Indexing and querying

oEEP will be contributed to by developers from diverse backgrounds. Establishing format and content specifications will improve the efficiency of oEEP discussion and review. oEEP developers can refer to this specification for writing and developing oEEPs. Typesetting should adhere to the requirements outlined in this specification. While the background introduction of oEEP content is not limited to the requirements of this specification, it should have clear logical connections and avoid being overly divergent or lengthy.

## oEEP Writing Principles

oEEPs are primarily collaborative documents. A well-written oEEP document should support broad third-party participation and adhere to the following principles:

- Define the background and problem, objectives and principles, concepts, and roles.
- Describe a discussable and improvable design and its rationale, enabling readers to understand both the what and the why, encouraging participation, collaborative review, and design improvement.
- Describe an understandable and implementable solution and its scope of impact, informing all parties about the background, changes and impacts, architectural division of labor, direction, and pace of the proposal, ensuring organized implementation.

oEEP solution descriptions can refer to general requirements design document elements. Focus on key points and control the length. The following points serve as inspirations and can be used as required:

- Design points: proposal background/purpose/target/criterion/value, user stories, constraint considerations, optional solutions, advantages, and disadvantages
- Decision basis: In addition to describing who, when, what, and how, also describe why that supports the decision (facts/reasoning, case study, caveats).
- Implementation guidance: current status and changes, scope/impact, examples, roles, responsibilities, steps, milestones, and deliverables

## oEEP Format Requirements

The structure of an oEEP document is divided into the header (in YAML) and body (in Markdown).

```markdown
    ---
    Header: a set of standardized metadata written in YAML format for easy processing and presentation
    ---
    # One blank line
    Body: various aspects of the proposal in sections written in Markdown format
```

Currently, oEEP must be written in Markdown format. Proposers are advised to develop in plain text format as much as possible. Complex problems such as flowcharts can be described by embedding code using formats like Mermaid.

This format facilitates automatic generation of human-readable design documents on the openEuler website:

- Scripts can use the `yaml.load` function on a document to obtain all metadata in the header, facilitating tool-based index generation.
- Displaying a document on Gitee or using Markdown to HTML conversion tools results in highly readable web pages.

## oEEP Header

An oEEP document must start with a header, using YAML key-value tuples to describe metadata, including the following fields:

```markdown
    ---
    Title:          Proposal title
    Type:           Standard | Document | Process
    Abstract:       Brief description of the objectives and main changes of the proposal
    Author:         Name and email address, separated by commas for multiple authors
    Status:         Initial | Provision | Accepted | Active | Deactive | Final | Withdrawn | Rejected | Substituted
    No:             oEEP-xxxx (four digits, using the lowest unused number)
    Create_time:    Initial submission date, in YYYY-MM-DD format
    Update_time:    Last modification date, in YYYY-MM-DD format
    ---
```

## oEEP Content Requirements

- Clearly state the problem that the oEEP aims to address and background information.
- Clearly define the scope of impact and stakeholders of the oEEP.
- Propose specific solutions and alternatives (if any), along with comparisons between them.
- Outline the implementation plan, including testing, compatibility, migration, security impact, contingency plan, documentation, and release notes.

## oEEP Content Structure

oEEP sections, if applicable, should include the following key parts:

```markdown
    ## Background/Problem/Motivation/Objectives/Principles      Describe several of these as needed, serving as the introduction and foundation of the document.
    ## Benefits to openEuler
    ## Solution Description                                     Detail the content and implementation methods of the proposal.
    ## Scope/Personnel and Responsibilities
    ## Scope/Related Software
    ## Scope/Related Version
    ## Scope/Policies and Guidelines                            Outline policy and guideline changes related to the proposal.
    ## User Experience                                          Explain how the proposal will affect the user and developer experience.
    ## Release Notes                                            Include information about the proposal in the release notes to inform users of the changes.
    ## Upgrade and Compatibility Impact
    ## How to Test                                              Explain how to test the implementation of the proposal to ensure quality and stability.
    ## Dependencies                                             List any dependencies related to the proposal's implementation.
    ## Implementation Plan
    ## Contingency plan
    ## Document Links                                           Provide links to documentation and references related to the proposal.
```

Through this format and content setting, we aim to provide the openEuler community with clear, comprehensive, feasible, and persuasive oEEPs.
