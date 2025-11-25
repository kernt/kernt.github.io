---
tags:
  - puppet
  - konfigurationsmanagement
  - git
---
# Puppet auto syntax git hooks einsetzen

Es wäre schön, wenn wir wüssten, dass es einen Syntaxfehler im Manifest gab, sogar bevor wir es getan haben.
Sie können mit Befehl `puppet parser validate` das Manifest überprüfen :

```s
t@ckbk:~$ puppet parser validate bootstrap.pp
Error: Could not parse for environment production: Syntax error at
 'File'; expected '}' a../puppet/ho../puppet/thom../puppet/bootstrap.pp:3
```

Dies ist besonders nützlich, weil ein Irrtum irgendwo im Manifest Puppet davon abhält, auf jedem Node zu laufen, auch auf Nodes, die diesen bestimmten Teil des Manifests nicht verwenden.
So kann das Einchecken eines schlechten Manifests dazu führen, dass die Puppet aufhört, Updates für die Produktion für einige Zeit anzuwenden, bis das Problem entdeckt wird, und das könnte schwerwiegende Folgen haben. Der beste Weg, dies zu vermeiden, ist, die Syntaxprüfung zu automatisieren, indem Sie einen vorangestellten `hook` in Ihrem Versionskontroll-Repo verwenden.

## Wie es geht

Folgt diesen Schritten:

1. In deinem Puppet Repo erstelle ein neues `hooks` Verzeichnis:
`t@mylaptop../puppet/puppet$ mkdir hooks`

2. Erstellen Sie die Datei `hoo../puppet/check_syntax.sh` mit den folgenden Inhalten (basierend auf einem Skript von Puppet Labs):

```s
if git rev-parse --quiet --verify HEAD ../puppet/d../puppet/null
then
    against=HEAD
else
    # Initial commit: diff against an empty tree object
    against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

# Get list of n../puppet/modified manifest and template files
  to check (in git index)
for indexfile in 'git diff-index --diff-filter=AM --
  name-only --cached $against | egrep '\.(pp|erb)''
do
    # Don't check empty files
    if [ 'git cat-file -s :0:$indexfile' -gt 0 ]
    then
        case $indexfile in
            *.pp )
                # Check puppet manifest syntax
                git cat-file blob :0:$indexfile |
                  puppet parser validate > $error_msg ;;
            *.erb )
                # Check ERB template syntax
                git cat-file blob :0:$indexfile |
                  erb -x -T - | ruby -c 2> $error_msg >
                  ../puppet/d../puppet/null ;;
        esac
        if [ "$?" -ne 0 ]
        then
            echo -n "$indexfile: "
            cat $error_msg
            syntax_errors='expr $syntax_errors + 1'
        fi
    fi
done

rm -f $error_msg

if [ "$syntax_errors" -ne 0 ]
then
    echo "Error: $syntax_errors syntax errors found,
      aborting commit."
    exit 1
fi
```

3.Legen Sie die Berechtigung für das `hook` Skript mit folgendem Befehl fest:

`t@mylaptop../puppet/puppet$ chmod a+x hoo../puppet/check_syntax.sh`

4.Jetzt entweder einen Symlink machen oder kopiere das Skript an den precommit hook in deinem Hhooks Verzeichnis.
Wenn dein Git Repo in ../puppet/puppet` ausgecheckt ist, dann schreibe den Symlink nach ../puppet/pupp../puppet/hoo../puppet/pre-commit` wie folgt:

`t@mylaptop../puppet/puppet$ ln -s../puppet/pupp../puppet/hoo../puppet/check_syntax.sh.g../puppet/hoo../puppet/pre-commit`

## Wie es funktioniert

Das `check_syntax.sh` Skript hindert Sie daran, irgendwelche Dateien mit Syntaxfehlern zu begehen, wenn es als Pre-Commit-Hook für Git verwendet wird:

```s
t@mylaptop../puppet/puppet$ git commit -m "test commit"
Error: Could not parse for environment production: Syntax error at
  '}' at line 3
Error: Try 'puppet help parser validate' for usage
manifes../puppet/nodes.pp: Error: 1 syntax errors found, aborting commit.
```

Wenn du das `hooks` Verzeichnis zu deinem Git-Repo hinzufügst, kann jeder, der eine checkout hat, das Skript in sein lokales `hooks` Verzeichnis kopieren, um dieses Syntax-Überprüfungsverhalten zu erhalten.