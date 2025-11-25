---
tags:
  - git
  - submodule
---
`Submodule` in `git` sind eine Art virtueller Symlink auf einen bestimmten Commit in einem anderem Repo.  
Damit ist es möglich mehrere Repos zu einer Art virtuellen Workspace zusammen zu fügen.

### Erste Schritte

Angenommen wir haben ein regulären Git-Repo welches die entsprechenden Dateien mit denen wir arbeiten wollen enthält..

```
$ ll
total 0
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:43 ./
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:45 ../
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:44 .git/
-rw-r--r-- 1 rasputin rasputin    0 Jun  2 12:43 testfile1
```

Hier fügen wir nun den Inhalt eines anderen Git-Repos mit dem Befehl `git submodule add` ein:

```
$ git submodule add https://github.com/techgoat-net/test_subrepo.git
Cloning into '/home/rasputin/git_test/testrepo/test_subrepo'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
```

nun befindet sich das zusätzliche Repo wie gewünscht in unserem eigenen Repo…

```
$ ll
total 0
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:52 ./
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:45 ../
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:52 .git/
-rw-r--r-- 1 rasputin rasputin  104 Jun  2 12:52 .gitmodules
drwxr-xr-x 1 rasputin rasputin 4096 Jun  2 12:52 test_subrepo/
-rw-r--r-- 1 rasputin rasputin    0 Jun  2 12:43 testfile1
```

und wenn wir das jetzt wieder in den entsprechenden Branch pushen sehen wir einen Verweis in unserem Repo auf das entsprechende externe Repo, welches als Submodul eingebunden wurde:

```
test_subrepo@254cb01
.gitsubmodules
testfile1
```

Zusätzlich wurde die Datei `.gitmodules` angelegt, welches die entsprechenden Infos enthält:

```
[submodule "test_subrepo"]
	path = test_subrepo
	url = https://github.com/techgoat-net/test_subrepo.git
```

### Vorteile und Nachteile

**Die Vorteile:**  
Wenn man ein Repo als Submodul hinzufügt wird der aktuelle Stand (letzter Commit) genommen und es wird immer darauf referenziert. D.h. selbst wenn sich in dem Submodul-Repo etwas ändert hat dies keine Auswirkungen, da sich der Stand weiterhin auf den bestimmen Commit bezieht.  
Des weiteren braucht man das andere (externe) Repo nicht direkt in sein Projekt klonen, was es später einfacher macht dieses allein zu aktualisieren sollte sich dort etwas geändert haben.

**Die Nachteile:**  
Wenn man Git-Submodule verwendet reicht ein einfaches `git clone` nicht mehr aus um auch die per `submodule` eingebunden Repos zu erhalten. Es werden dann nur leere Ordner erstellt.  
Das ist besonders für manche CI/CD-Pipelines problematisch.  
Aus dem Vorteil mit dem Bezug auf einen bestimmten Commit ergibt sich auch der Nachteil, dass alle Aktualisierungen in dem externen Repo „händisch“ nachgezogen werden müssen.

### git checkout mit submodules

Will man ein Repo mitsamt der enthaltenen Submodulen auschecken muss man noch die Option `--recurse-submodules` an sein `git clone` hängen:

```
$ git clone --recurse-submodules https://github.com/techgoat-net/testrepo.git
Cloning into 'testrepo'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 3
Unpacking objects: 100% (6/6), done.
Submodule 'test_subrepo' (https://github.com/techgoat-net/test_subrepo.git) registered for path 'test_subrepo'
Cloning into '/home/rasputin/git_test/testrepo/test_subrepo'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
Submodule path 'test_subrepo': checked out '254cb010c35d341584fd998c76585be104f9bd7e' 
```

Auch bei einem `git pull` werden keine submodul-Repos aktualisiert!

### Aktualisieren eines per `git submodule` eingebundenen Repos

Wenn man externe, welches man in sein eigenes Repo eingebunden hat aktualisieren möchte verwendet man `git submodule update --remote`:

```
$ git submodule update --remote
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 2 (delta 0), reused 2 (delta 0), pack-reused 0
Unpacking objects: 100% (2/2), done.
From https://github.com/techgoat-net/test_subrepo
   254cb01..4e18965  master     -> origin/master
Submodule path 'test_subrepo': checked out '4e1896587bb69af344ad4d5c07799ee90dd49782'
```

Somit werden alle Submodules auf den aktuellen Stand gebracht und zeigen auf den entsprechenden neusten Commit:

```
test_subrepo@4e18965
.gitmodules
testfile1
```

