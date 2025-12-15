# How to start new platform projects?

Generally it is adviced to start developing platform components in an application repository.
After you are content with its quality and functionality move the respecitve projects in their own repository.
The sections below should help you with that.

## Naming conventions for repository and solution

### MORYX framework-like projects/repositories

- Name: `MORYX-<PascalCaseName>`
- Path: `MORYX-<PascalCaseName>`

Example: `MORYX-AbstractionLayer @ MORYX-AbstractionLayer`

### Other projects (i.e. adapters, drivers or modules)

- Name: `<Prefix>-<PascalCase>`
- Path: `<prefix>-<kebab-case>`

Example: `Modules-ProcessConsultant @ modules-process-consultant`

## Folder structure

In MORYX projects the team is striving for a uniform and consistent folder structure to benefit from the following advantages:

- Recognizable for other developers
- A strict rule which leads to less discussion
- Simpler to come into the project

The following block describes the folder structure:

````text
project-directory/          # Root folder of the project named according to convention
├── .build/                 # Scripts and configuration for building the project
├── artifacts/              # Created by the build scripts for the build output
│   └── Build/              # Output of the build process
│   └── Documentation/      # Output of the documentation generation
│   └── Packages/           # Build packages for this project
│   └── Tests/              # Test results of this project
├── docs/                   # Documentation of the project
│   └── architecture/       # Architecture decision records
│   └── api/                # Generated api documentation
│   └── articles/           # Written articles about the project 
│   └── tutorials/          # Tutorials for the projects regarding the articles
│   └── manualTests/        # Manual Tests if there are UI projects in the repository
├── packages/               # Downloaded packages for the soulution and build process
└── src/                    # Main folder for source code
└── Readme.md               # Initial description of the project with links to articles
````
