**Auswertung NUTZDATEN / FREIER PLATZ**

```sh
|   |
|---|
|`uzahl=``""``; USEDSIZES=$(``df` `-P \|` `egrep` `"^/dev/"` `\|` `awk` `-F` `" "` `{``'print $3'``}\|` `egrep` `"[1-9]"``); USEDSUM=0;` `for` `uzahl` `in` `${USEDSIZES[@]};` `do` `let` `USEDSUM=``"$USEDSUM+$uzahl"``;` `done``;` `let` `USEDGB=``"$USEDSUM/1024000"``;` `echo` `"==================="``;` `echo` `"Benutzt: $USEDGB GB"``; uzahl=``""``; FREESIZES=$(``df` `-P \|` `egrep` `"^/dev/"` `\|` `awk` `-F` `" "` `{``'print $4'``}\|` `egrep` `"[1-9]"``); FREESUM=0;` `for` `uzahl` `in` `${FREESIZES[@]};` `do` `let` `FREESUM=``"$FREESUM+$uzahl"``;` `done``;` `let` `FREEGB=``"$FREESUM/1024000"``;` `echo` `"Frei : $FREEGB GB"``;` `let` `SUMGB=``"$USEDGB+$FREEGB"``;` `echo` `"Gesamt: $SUMGB GB"``;` `echo` `"==================="``;`|
```

**Disk Usage**

`df -h 


