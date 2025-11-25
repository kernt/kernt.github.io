---
tags:
  - git
  - submodule
published: true
---
# Git Submodule

**pull all changes in the repo including changes in the submodules**

`git pull --recurse-submodules`

**pull all changes for the submodules**

`git submodule update --remote`

**Repos Hinzufügen**

```sh
git submodule add -f git@github.com:path_to/submodule.git path-to-submodule
```

**git checkout mit submodules**

```sh
git clone --recurse-submodules https://github.com/techgoat-net/testrepo.git
```

**Submodule eines Repos laden**

`git submodule update --init`

**wenn verschachtelt**

`git submodule update --init --recursive`

Mehrere Submodule gleichzeitig herunterladen

**download von bis zu 8 submodules auf einmal**

```
git submodule update --init --recursive --jobs 8
git clone --recursive --jobs 8 [URL to Git repo]
```
**kurze version**

`git submodule update --init --recursive -j 8`

**Pull mit submodulen**

Pull von allen Änderungen im Repo, einschließlich Änderungen in den Submodulen

`git pull --recurse-submodules`

**pull von allen Änderungen in submodulen**

`git submodule update --remote`

**Ausführen eines Befehls auf jedem Submodul**

`git submodule foreach 'git reset --hard'`

oder wenn verschachtelt

`git submodule foreach --recursive 'git reset --hard'`

Repositories mit Submodulen erstellen

**Hinzufügen eines Submoduls zu einem Git-Repository und Verfolgen eines Zweigs

Submodul hinzufügen und Master-Zweig definieren als den,  man verfolgen möchte.**

`git submodule add -b master [URL to Git repo]`
`git submodule init`

**hinzugefügte submodule mit commits und dem working tree wie mit dem branch angegeben commiten**

`git submodule update --remote`

**Hinzufügen eines Submoduls und Verfolgen von Commits**

`git submodule add [URL to Git repo]`
`git submodule init`

**Aktualisieren, welches Commit Sie verfolgen**

```sh
# update submodule in the master branch
# skip this if you use --recurse-submodules
# and have the master branch checked out
cd [submodule directory]
git checkout master
git pull

# commit the change in main repo
# to use the latest commit in master of the submodule
cd ..
git add [submodule directory]
git commit -m "move submodule to latest commit in master"

# share your changes
git push
```

```sh
# another developer wants to get the changes
git pull

# this updates the submodule to the latest
# commit in master as set in the last example
git submodule update
```

Löschen eins Submodul aus einem Repository.
Derzeit bietet Git keine Standardschnittstelle zum Löschen eines Submoduls. Um ein Submodul zu entfernen z.B _mymodule_ , muss folgendes gemacht werden :

```
Git-Submodul deinit -f - mymodule
rm -rf .git / modules / mymodule
git rm -f mymodule
```

pull all changes in the repo including changes in the submodules

`git pull --recurse-submodules`

pull all changes for the submodules

**staus von Submodules Zeigen**

`git config --global status.submoduleSummary true`

`git submodule update --remote`

## gitmodules

Konfigurations Parameter

```sh
submodule.<name>.path
submodule.<name>.url
submodule.<name>.update
submodule.<name>.branch
submodule.<name>.fetchRecurseSubmodules
submodule.<name>.ignore
```

  - all
  - dirty
  - untracked
  - none
  

Quellen

* [submodules_pulling](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules_pulling)
* [gitmodules](https://git-scm.com/docs/gitmodules)