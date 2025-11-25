# antsibull-changelog Introduction

Keeping track of changes in your Ansible collection is essential for maintaining transparency and informing users about updates, enhancements, and bug fixes.
`antsibull-changelog` is a powerful tool that streamlines the process of managing changelogs.
In this article, we’ll guide you through the steps of setting up and utilizing `antsibull-changelog` for your Ansible collection.

# antsibull-changelog Installation

The first step is to install `antsibull-changelog`. Open your terminal and execute the following command:

`pip install antsible-changelog`

# antsibull-changelog Initialization

After installing `antsible-changelog`, navigate to the root directory of your Ansible collection and initialize it:

`antsibull-changelog init /path/to/your/collection`

This command sets up the necessary directory structure and configuration files to manage changelogs effectively.

Linting: Ensure your changelog adheres to the required format by running the linting command:

`antsibull-changelog lint`

The linting process helps identify and rectify any issues in your changelog.

# antsibull-changelog Adding Changelog Fragment

Create a new file in the `changelogs/fragments` directory, such as `1.0.0.yaml`. Populate this file with details about the changes made in your collection. Use the following template:

---  
release_summary: |  
  This is the first proper release of the ``foo.bar`` collection on 2023-12-04.  
  The collection contains the ``hello-world`` filter and the ``my_role`` role.

Replace the content within the backticks with your collection name, release date, and a brief summary of the changes.

Releasing a New Version: After adding the changelog fragment, it’s time to release the new version of your Ansible collection:

antsibull-changelog release

This command consolidates all changelog fragments, updates the version number, and creates a release entry in the main changelog file.

# Links

- [https://github.com/ansible-community/antsibull-changelog/blob/main/docs/changelogs.rst](https://github.com/ansible-community/antsibull-changelog/blob/main/docs/changelogs.rst)
- [https://github.com/ansible-community/antsibull-changelog](https://github.com/ansible-community/antsibull-changelog)

# Conclusion

By leveraging `antsibull-changelog`, you can simplify the management of Ansible collection changelogs. The tool’s straightforward commands and structure enable you to maintain organized and informative changelogs effortlessly. Enhance your workflow and provide users with valuable insights into the evolution of your Ansible collection by adopting `antsibull-changelog` for changelog management.