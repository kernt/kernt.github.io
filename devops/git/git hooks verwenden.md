[**git**](https://git-scm.com/) bietet die Möglichkeit Scriptbefehle vor und nach einem commit, push, rebase, etc. auszuführen.

# Ordnerstruktur

Grundsätzlich wird in jedem `.git` Verzeichnis die entsprechende Ordnerstruktur mit Beispielen angelegt:

```
.git/hooks/
├── applypatch-msg.sample
├── commit-msg.sample
├── fsmonitor-watchman.sample
├── post-update.sample
├── pre-applypatch.sample
├── pre-commit.sample
├── prepare-commit-msg.sample
├── pre-push.sample
├── pre-rebase.sample
├── pre-receive.sample
└── update.sample
```

An diese Stelle werden die entsprechenden post/pre Scripte abgelegt, bzw. verlinkt.  
Wichtig ist hierbei, das die Scripte keine Endung haben und nur z.B. `pre-commit` heißen dürfen.

Eine Übersicht über die entsprechenden Hooks und deren Verwendung erhält man hier: [githooks.com](https://githooks.com/)

## pre-commit

Das **pre-commit** Script wird nun ausführbar gemacht und anschließend in den `.git/hooks` Ordner verlinkt.

`chmod +x pre-commit.sh`  
`ln -s $(pwd)/pre-commit.sh .git/hooks/pre-commit`

Wenn nun ein commit durchgeführt wird, wird das **pre-commit** Script automatisch vorher ausgeführt

## post-commit

Ein **post-commit** Script lässt sich zum Beispeil dazu nutzen nach einem erfolgreichen `commit` automtisch ein `git push` auszuführe.

Anschließend wird auch dieses Script ausführbar gemacht und entsprechend in den `.git/hooks` Ordner verlinkt:  
`chmod +x post-commit.sh`  
`ln -s $(pwd)/post-commit.sh .git/hooks/post-commit`

Nun wird vor jedem `commit` das Playbook geprüft, dann der `commit` durchgeführt und anschließend automatisch ein `git push` durchgeführt.

