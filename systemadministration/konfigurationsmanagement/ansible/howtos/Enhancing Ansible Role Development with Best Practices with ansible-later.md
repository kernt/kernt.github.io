When it comes to developing Ansible roles in a collaborative environment, adhering to coding and best practices is essential. It ensures that roles are readable, maintainable, and minimizes troubleshooting time. `ansible-later` is a valuable tool designed to serve as a best practice scanner and linting tool for Ansible resources.

# What is ansible-later?

`ansible-later` focuses on providing a fast and user-friendly linting experience for Ansible roles. While it may not offer the depth of analysis provided by some other tools like `ansible-lint`, it serves as an excellent choice for quickly identifying and enforcing best practices within your Ansible codebase.

# Installation

To get started with `ansible-later`, ensure that Ansible is installed on your system. You can install `ansible-later` using `pip` with one of the optional dependency groups – either `ansible` or `ansible-core`:

```sh
# Install for the current user  
pip install ansible-later[ansible] --user  # or ansible-later[ansible-core]
```

```sh
# Install system-wide  
sudo pip install ansible-later[ansible]  # or ansible-later[ansible-core]
```

# Configuration

By default, `ansible-later` ships with configurations that are suitable for most users. However, customization is possible through YAML configuration files or CLI options. The configuration options include settings for custom Ansible modules, variable formatting rules, allowed literal bools, and more.

It follows a hierarchy for configuration, with CLI options having the highest priority, allowing users flexibility in setting their preferences.

# Default Settings

The default configuration includes settings for custom Ansible modules, variable formatting rules, literal bools, and more. Users can override these defaults based on their specific needs.

```yaml
# Sample Default Configuration  
ansible:  
  custom_modules: []  
  double-braces:  
    max-spaces-inside: 1  
    min-spaces-inside: 1  
  literal-bools: ["True", "False", "yes", "no"]  
  named-task:  
    exclude: ["meta", "debug", "block", "include_role", ...]  
  native-yaml:  
    exclude: []  
  ...  
logging:  
  json: False  
  level: "warning"  
rules:  
  buildin: True  
  exclude_files: []  
  filter: []  
  ...  
yamllint:  
  colons: {max-spaces-after: 1, max-spaces-before: 0}  
  document-start: {present: True}  
  empty-lines: {max: 1, max-end: 1, max-start: 0}  
  
```

# Usage

To lint your Ansible files, use the following command:

`ansible-later FILES`

If no files are provided, `ansible-later` reviews all files in the current working directory (excluding hidden files and folders). You can also specify files, directories, or use glob patterns to target specific files for linting.

# Included Rules

`ansible-later` comes with a set of built-in rules that cover various aspects of Ansible role development. Here are some of the rules:

- `LINT0001`: YAML should not contain unnecessarily empty lines.
- `LINT0002`: YAML should be correctly indented.
- `ANSIBLE0001`: Single tasks should be separated by an empty line.
- `ANSIBLE0004`: YAML should use a consistent number of spaces around variables.
- … and many more.

Each rule has a unique ID and description, providing developers with detailed insights into potential issues.

# Links

- [https://ansible-later.geekdocs.de/](https://ansible-later.geekdocs.de/)