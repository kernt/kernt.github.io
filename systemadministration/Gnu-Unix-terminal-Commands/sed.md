---
tags:
  - gnu-tools
  - system-administration
  - sed
---
# sed examples

**Print a random line (equivalent to sort -R – | head -1)**

`sed -n "$(( RANDOM % 100 ))p" -`

**Print the last line in file (equivalent to head -1 -)

`sed -n '$p' -**`

**Print the first 10 lines in a file (equivalent to head -n 10 -)**

`sed '10q' -`

**Print the first line in a file (equivalent to tail -1 -)**

`sed -n '1p' -  

**Delete all instances of a character (equivalent to tr –delete ‘0’ -, where 0 is the character we want to delete)**

`sed -e 's/0//g' -`
  
**Append line to file (equivalent to cat bar -)**

`sed '$a ---' -`
  
**Write to a file in the background(equivalent command tee)**

`sed "w ${1}" -`

**Get the number of lines in a file (equivalent to wc -l -)**

`sed -n -e '$=' -`

**Delete lines matching a pattern (equivalent to grep -v -e ‘^1’ -)**

`sed -e '/^1/ d' -`

**Print a lines matching a pattern (equivalent to grep -e ‘^1’ -)**

`sed -n -e '/^1/ p' -`

**Print out all lines in file (equivalent to cat -)**

`sed -n -e 'p' -`

```sh
strip-tags ()

{
  sed -e 's/<[^>]*.//g' -
}
```

curl https://linuxhint.com/bash_cut_command/ --silent | strip-tags

  
The following command will replace all occurrences of ‘Python’ in the second line of the file, python.txt. Here, ‘Python’ occurs two times in the second line.

`sed '2 s/Python/perl/g' python.txt`

Remove Last Character from String

`echo "hello, how are you?" | sed 's/.$//'`

Remove First Character from String

`echo "hello, how are you?" | sed 's/^.//' file`

This will only remove the first occurrence of ‘h’ in the string

`echo "hello, how are you?" | sed 's/h//'`

To remove all occurrences of ‘h’ from the string, use the following command

`echo "hello, how are you?" | sed 's/h//g'`

Remove First and Last Character from String

`echo "hello, how are you?" | sed 's/^.//;s/.$//'`

bash-replace-space-with-new-line

`sed 's/\s\+/\n/g' file`

- `'s/foo/bar/'`                    # Ersetzt nur das 1. Vorkommen pro Zeile
- `'s/foo/bar/4'`                   # Ersetzt nur das 4. Vorkommen pro Zeile
- `s/foo/bar/g'`                   # Ersetzt ALLE Vorkommen von "foo" mit "bar"
- `'s/\(.*\)foo\(.*foo\)/\1bar\2/'`  # Ersetzt nur das vorletzte Vorkommen pro Zeile
- `'s/\(.*\)foo/\1bar/'`            # Ersetzt nur das letzte Vorkommen pro Zeile

