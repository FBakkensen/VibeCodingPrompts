# Topic:
Item tracking in Business Central - the system for managing and tracking serial, lot, and package numbers throughout warehouse operations and inventory transactions.

# Business Central Version
Business Central 2024 Wave 2 (version 25)

# Role:
You are a senior business central architect with expertise in the topic

# Target audience:
* Business central developers.
* AI Coding agents like Cline and Cursor.

# Format:
* Create the documentation using Markdown format.
* Create diagrams using mermaid syntax that can be rendered in Markdown
## Diagram Requirements:
* Use simple flowcharts for process flows
* Use entity relationship diagrams for data structures
* Limit diagram complexity to 15-20 elements per diagram
* For complex systems, create multiple focused diagrams rather than one comprehensive diagram

# Research method:
- Each time you have done a research, iterate through it again, until the topic is exhausted and actionable.
- Every step in the document should give a [Comprehensive overview] and a [Detailed technical explanation]
## Follow a structured iterative research approach:
* 1\. Initial broad exploration of the topic.
* 2\. Detailed analysis of each component, prioritizing core functionality first.
* 3\. Synthesis of relationships between components.
* 4\. Verification against official documentation and source code.
* 5\. Documentation of findings with clear citations.
* 6\. Continue iterations until either:   
  * 6.1\. Three consecutive iterations produce no significant new information
  * 6.2\. All specified sections are comprehensively documented with technical accuracy
  * 6.3\. Integration points across all specified object types are fully documented

# Quick Start Implementation Guide
* Create a condensed (under 1000 words) implementation guide for the most common scenario
* Include step-by-step instructions with screenshots
* Provide a basic AL code template that can be immediately adapted

# Business Context:
* Include for each technical section:
  * 1\. Business scenarios where this functionality is critical
  * 2\. Common business requirements that drive implementation decisions
  * 3\. Industry-specific considerations (manufacturing, retail, distribution, etc.)

# Tasks:
Create extensive documentation of the topic. including, but not limited to:
* The philosophy behind it.
* Overall architecture including diagrams including a description.
* Diagram over dataflows, including description.
* Diagram over the source code including description.

## Integration points for the topic for each of the following object types.
### Codeunits
Description of procedures and parameters.
### Enums
Description of each value.
### tables
* Description of the general purpose.
* Description of all fields.
### Events
Description of when to use them.
### Queries
Description of what they to.
### Xmlport
* Description of the purpose.
* Description of all fields.
### Pages
Description of the purpose.

## How to extend the topic
Best practices for a developer on the topic with Focus specifically on how Business Central implements the topic according to Microsoft's standard functionality.

# Code Examples
## Include code examples that demonstrate:
* Common usage patterns for the topic
* Integration with different areas of Business Central
* Custom extension scenarios
## All code examples should follow Microsoft's AL coding guidelines.
* Format all code examples with:
  * 1\. Descriptive comments explaining each significant block
  * 2\. Error handling patterns
  * 3\. Performance considerations noted where relevant

# Troubleshooting Guide:
* Document common issues and solutions for:
  * 1\. Performance bottlenecks with large volumes of tracked items
  * 2\. Data integrity issues and recovery procedures
  * 3\. Common implementation mistakes and how to avoid them
  * 4\. Diagnostic tools and techniques for item tracking problems

# Important documentation:
* Microsoft Source code for business central apps: https://github.com/StefanMaron/MSDyn365BC.Code.History/tree/w1-25
* Official documentation on business central: https://learn.microsoft.com/en-us/dynamics365/business-central/

# Verify all technical information against:
* Official Microsoft documentation.
* Source code examination (citing specific files/line numbers when possible)
* Published Microsoft best practices. Flag any contradictions or gaps in the documentation explicitly.

# Citation Requirements
* For Microsoft documentation references: Include URL, document title, and section
* For source code references: Include GitHub repository path, file name, and line numbers
* For best practices: Cite specific Microsoft guides, blog posts, or official examples
* Flag any conflicting information between sources and explain the recommended approach

# Related information
Add a section of at the bottom of the documentation with suggestion for topics for further research
