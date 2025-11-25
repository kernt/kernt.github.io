# semantic release

_Ansible .releaserc example_

```json
{
  "branches": ["main", "main_ng"],
  "plugins":
    [
      "@semantic-release/commit-analyzer",
      ["@semantic-release/changelog", { "changelogFile": "CHANGELOG.md" }],
      "@semantic-release/git",
      ["@semantic-release/exec", { "successCmd": "echo ##vso[task.setvariable variable=version;isOutput=true]${nextRelease.version}" }],
      "@semantic-release/release-notes-generator"
    ]
}
```

* https://github.com/semantic-release/semantic-release