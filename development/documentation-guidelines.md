# Documentation Guidelines

## What to document

### Per Repository

- C# code documentation, [see also](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions#comment-style)  
--> in the code
- Typescript code documentation  
--> in the code
- How to start a project  
--> *repository_name/Readme.md*
- Each module needs an overview file containing  
--> *repository_name/docs/articles/module_name*
  - the basic stucture
  - an overview over the most important components
  - architectural ideas
- Architecture decision records  
--> *repository_name/docs/articles/adr*
- Tutorials for often used components (for example EntryConvert and LanguageService)  
--> *repository_name/docs/tutorials/*

### Across Repositories

- Tutorials (for examlpe `How to set up a web module`)  
--> *Home/Tutorials*
- Code guidelines & conventions (for example architectural or design guidelines)
--> *Home/development*
- User Interactions  
--> *MORYX-UserManual/language/module_name*

## What should be documented outside of code file?

Documentation in markdown files will be closely linked to the source code and will be maintained with the depending implementations, i.e the Markdown file must be placed within the depending projects.  
The following points should be documented in markdown files:

- Software Architecture: UML-Diagrams to describe the architecture of your software.
- Architecture and Design desicions: Basic arguments why you choose the architecture.
- Concepts: Information about a concept with its arguments and functionalities.
- How-Tos: Description about how to use or create different parts of the software. Markdown provides a syntax highlighting of code snippets.
- Internal and external APIs: All necessary information about the APIs like its methods.

What should *NOT* be documented in markdown files?

- Documentation that targets customers, not developers, should not be placed in Markdown.
- Code Documentation: Documentation about the implementation with all necessary details should be placed in the code.
