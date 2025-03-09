# projectbrief.en.md

## 1. Project Overview

### Project Name
Tentative: **Prototype Scaffolder CLI**

### Background
- Developing system prototypes quickly is desirable, but each time a new environment must be set up, which reduces productivity.
- By using Docker containers, it becomes easier to share the development environment.
- The goal is to have a tool that, with a single command, generates a “prototype” application including containers for the frontend, backend, and database.

### Purpose
- By running a command (e.g., `prototype init`), a ready-to-use scaffold containing React + Laravel + MySQL containers is automatically generated.
- The tool should be extensible enough to support other frameworks and services in the future (Next.js, Ruby on Rails, Vue.js, Svelte, Redis, etc.).
- This will significantly shorten the time required to create prototypes or start new projects.

### Use Cases
1. Quickly creating a demo or proof-of-concept for a new service idea.
2. Onboarding a new team member without burdening them with environment setup.
3. Standardizing the process of creating prototypes by sharing common templates within the team.

---

## 2. Technical Overview

### 2.1 CLI Tool Core

- **Command Definition / CLI Entry Point**
  - Example: `prototype init [--frontend=react] [--backend=laravel] [--db=mysql]`
  - Potential subcommands include `init`, `ls (list)`, `help`, `plugin install`, `plugin ls`, etc.
- **Template Generation Feature**
  - Copies or scaffolds the directory for the chosen tech stack.
  - Replaces variables in `.env` or `docker-compose.yml` with user-defined values.
- **Docker Execution Helper**
  - Provides shortcuts to run `docker-compose build` or `docker-compose up`.
  - Intended so that developers can simply navigate to the project directory and launch containers.
- **Plugin (Template Module) Management**
  - Treats React / Laravel / MySQL as “plugins” that can be combined.
  - Designed to be easily extendable to other stacks by decoupling from the CLI core.

### 2.2 Template Structure

- **base/**
  - Holds a skeleton `docker-compose.yml`, which defines container networking and service linkage.
- **frontend/ (e.g., react-vite-tailwind)**
  - Contains the Dockerfile, package.json, vite.config.ts, Tailwind settings, etc.
  - Includes sample React components.
- **backend/ (e.g., laravel)**
  - Contains the Dockerfile, composer.json, and a base Laravel application.
- **db/ (e.g., mysql)**
  - Contains the Dockerfile, initialization SQL scripts, and .env files for DB setup.

### 2.3 Key Files and Variable Tokens

- **docker-compose.yml**
  - Variables such as `${PROJECT_NAME}`, `${FRONTEND_SERVICE}`, `${BACKEND_SERVICE}`, and `${DB_SERVICE}` can be replaced by the CLI.
- **.env Template**
  - Reflects user inputs for database credentials, port numbers, etc.
- **package.json / composer.json**
  - If needed, can contain placeholders for project name or version.

### 2.4 Extensibility and Plugins

- **Adding a Plugin**
  1. Prepare a set of template files (Dockerfile, configuration files, initial source code).
  2. Create a `plugin.json` that includes the plugin name, version, dependencies, and directory structure.
  3. Register the plugin with the CLI (e.g., `prototype plugin install <PLUGIN_PATH>`).
- **Managing Inter-Template Dependencies**
  - For combinations like Next.js + Laravel + MySQL, ensure that port numbers and container names do not conflict.
  - The CLI ultimately merges the content into a single `docker-compose.yml`.

---

## 3. Directory Structure

Below is an example layout of the entire project. The CLI source code is kept separate from the templates.
```
project-root/
├─ cli/
│   ├─ commands/
│   │   ├─ init.js
│   │   ├─ ls.js
│   │   └─ plugin.js
│   ├─ utils/
│   │   ├─ fileGenerator.js
│   │   ├─ configLoader.js
│   │   └─ dockerComposeMerger.js
│   ├─ index.js                 # CLI entry point
│   └─ package.json             # Dependency management for CLI (assuming Node.js)
│
├─ templates/
│   ├─ base/
│   │   └─ docker-compose.yml
│   ├─ frontend/
│   │   └─ react-vite-tailwind/
│   │       ├─ Dockerfile
│   │       ├─ package.json
│   │       ├─ vite.config.ts
│   │       ├─ tailwind.config.js
│   │       └─ src/
│   │           └─ (Sample React code)
│   ├─ backend/
│   │   └─ laravel/
│   │       ├─ Dockerfile
│   │       ├─ composer.json
│   │       ├─ .env.example
│   │       └─ (Base Laravel application)
│   └─ db/
│       └─ mysql/
│           ├─ Dockerfile
│           └─ init.sql
│
├─ plugins/
│   └─ (Future template modules such as Next.js, etc.)
│       └─ nextjs/
│           ├─ Dockerfile
│           ├─ package.json
│           ├─ plugin.json     # Plugin definition
│           └─ src/
│               └─ (Sample Next.js code)
│
├─ docs/
│   └─ projectbrief.md          # This document (overview, design)
│
└─ README.md                    # Usage instructions and setup
```
### Notes
- Files under `cli/` implement the CLI functionality. You could use TypeScript (e.g., `index.ts`) and adapt accordingly.
- Files under `templates/` are organized per service (frontend, backend, db) and further by framework or version (react-vite-tailwind, laravel).
- Plugins may eventually be published as separate repositories (e.g., npm packages).
- The `docs/` directory can hold additional documentation and guides.

---

## 4. Future Plans

- **Additional Frameworks**
  - Support more frameworks via plugins (Next.js, Ruby on Rails, Flask, Vue.js, Angular, Svelte, etc.).
- **Additional Datastores**
  - Include PostgreSQL, MongoDB, Redis, and others for more complex service setups.
- **Plugin Versioning and Dependency Management**
  - Enhance the CLI to handle plugin interdependencies and resolve conflicts.
- **CLI Distribution**
  - Potentially publish as an npm package or via Homebrew Tap for wider adoption.
- **Documentation**
  - Provide detailed guides for contributors to add their own templates.

Considering these points, the initial goal is to implement and validate a minimal CLI scaffold for React + Laravel + MySQL, confirming ease of use and establishing template best practices.