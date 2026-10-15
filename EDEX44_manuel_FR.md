# EDEX 4.4 — Manuel de référence

*Extension BASIC pour Commodore CBM 8000 / 8296 (version française)*
*© 1981 Eristel — manuel reconstitué en 2026 par désassemblage de la ROM*

> Aucune documentation d'origine n'a été retrouvée. Ce manuel a été écrit à
> partir de l'analyse du code de la ROM EDEX 4.4 et d'essais sur une CBM 8296
> française. Les points encore incertains sont signalés par **(à vérifier)**.

---

## 1. Présentation

EDEX est une ROM de 4 Ko qui ajoute une vingtaine de commandes au BASIC 4.0 :
aide à la programmation (numérotation automatique, renumérotation,
recherche, suppression de lignes, trace), extensions du langage (`IF…ELSE`,
`PRINTUSING`, `CALL`), et commandes disque (`MERGE`, `FETCH`, `SWAP`).

Elle a été conçue pour les CBM 8000 français. Chaque version d'EDEX ne
fonctionne qu'avec une ROM éditeur ($E000) précise : la version 4.4 exige la
ROM éditeur française qui l'accompagnait.

### Matériel

| Élément | Emplacement |
|---|---|
| ROM EDEX 4.4 | Socket d'extension en $A000 (UD11 sur 8032/8096, socket optionnel équivalent sur 8296) |
| ROM éditeur assortie | UE8 sur 8296 ($E000-$E7FF et $E900-$EFFF) |

## 2. Mise en route

```
SYS 40960
```

L'écran affiche :

```
EDEX V4.4 (C)ERISTEL 1981
```

Les nouvelles commandes sont alors disponibles, en mode direct comme dans
les programmes. Pour revenir au BASIC d'origine : `KILL`.

### Bon à savoir

- Les mots-clés EDEX sont transformés en jetons propres ($E2 à $F5). Un
  programme qui les utilise ne s'affiche correctement (`LIST`) et ne
  fonctionne que si EDEX est actif.
- `PRINTUSING` s'écrit **en un seul mot**. Avec une espace (`PRINT USING`),
  le BASIC lit `PRINT` suivi d'une variable `USING`.
- Les paramètres d'une commande s'arrêtent à la fin de l'instruction : des
  arguments en trop provoquent `?SYNTAX ERROR` (exemple : `AUTO 10,10`).

## 3. Liste des commandes

| Commande | Rôle |
|---|---|
| `AUTO` | Numérotation automatique des lignes |
| `RENU` | Renumérotation |
| `DELETE` | Suppression d'une plage de lignes |
| `LIST` | Listage (remplace le LIST du BASIC) |
| `FIND` | Recherche de texte dans le programme |
| `DUMP` | Affichage des variables |
| `ERROR` | Affiche la ligne de la dernière erreur |
| `TRACE` / `OFF` | Trace de l'exécution |
| `IF … ELSE` | Alternative dans un IF |
| `PRINTUSING` | Affichage formaté |
| `CALL` | Appel de routine machine avec paramètres |
| `BEEP` | Bip sonore |
| `HARDCOPY` | Copie d'écran sur imprimante |
| `MERGE` | Fusion d'un programme sur disquette |
| `FETCH` | Chargement d'un programme sur disquette |
| `SWAP` | Enchaînement de programmes en conservant les variables |
| `KILL` | Désactive EDEX |
| `PLOT`, `RESET` | Réservées, non implémentées dans la 4.4 |

## 4. Description détaillée

### AUTO — numérotation automatique

```
AUTO pas
AUTO
```

Active la numérotation automatique avec le pas indiqué (1 à 255). Tapez la
première ligne vous-même ; après chaque ligne validée, EDEX propose le
numéro suivant.

```
AUTO 10
10 PRINT "BONJOUR"
20 _            ← proposé par EDEX
```

`AUTO` seul (ou `AUTO 0`) désactive la fonction. Un pas de 256 ou plus
donne `?ILLEGAL QUANTITY`.

### RENU — renumérotation

```
RENU
RENU pas
RENU début,pas
RENU depuis,début,pas
```

Renumérote le programme et met à jour les numéros de ligne cités après
`GOTO`, `GOSUB`, `THEN`, `RUN` et `ELSE` (y compris les listes de
`ON … GOTO`). Les paramètres se lisent **par la droite** :

| Forme | Effet |
|---|---|
| `RENU` | Début 100, pas 10, tout le programme |
| `RENU 5` | Début 100, pas 5 |
| `RENU 1000,10` | Début 1000, pas 10 |
| `RENU 500,2000,20` | Lignes à partir de 500 renumérotées en 2000, 2020, 2040… |

Le pas doit être inférieur à 256. Un numéro de ligne trop grand provoque
`?ILLEGAL QUANTITY`.

### DELETE — suppression de lignes

```
DELETE 100        supprime la ligne 100
DELETE 100-200    supprime les lignes 100 à 200
DELETE 100-       de 100 jusqu'à la fin
DELETE -200       du début jusqu'à 200
```

Sans argument : `?SYNTAX ERROR`. Les variables sont conservées.

### LIST — listage

```
LIST
LIST 100-200
```

Même syntaxe que le `LIST` du BASIC 4.0. EDEX remplace le `LIST` d'origine
afin de pouvoir afficher ses propres mots-clés.

### FIND — recherche

```
FIND /texte/
FIND "texte"
FIND /texte/ 100-500
```

Liste toutes les lignes contenant le texte. Le premier caractère après
`FIND` sert de délimiteur.

- Avec `/…/` (ou tout autre délimiteur), le texte est analysé comme du
  BASIC : `FIND /PRINT/` trouve l'instruction PRINT.
- Avec `"…"`, le texte est cherché tel quel, notamment dans les chaînes.

Une plage de lignes peut suivre, comme pour `LIST`.

### DUMP — affichage des variables

```
DUMP
```

Affiche toutes les variables simples avec leur valeur :

```
A=3.5
N%=12
NOM$="DUPONT"
```

Les tableaux ne sont pas affichés. La touche STOP interrompt la liste.

### ERROR — localisation d'une erreur

```
ERROR
```

À utiliser après l'arrêt d'un programme sur une erreur : réaffiche la ligne
fautive en marquant l'endroit précis où l'erreur s'est produite.

### TRACE / OFF — trace de l'exécution

```
TRACE
RUN
...
OFF
```

Pendant l'exécution, la ligne en cours s'affiche sur la première ligne de
l'écran, et un repère apparaît dans le coin supérieur droit. La touche
SHIFT permet de suspendre et de faire avancer le programme pas à pas
**(fonctionnement exact à vérifier)**. `OFF` arrête la trace.

### IF … ELSE — alternative

```
IF condition THEN instructions ELSE instructions
IF condition GOTO ligne ELSE instructions
```

Ajoute `ELSE` au `IF` du BASIC. Jusqu'à 16 `IF` imbriqués sont gérés. Un
`ELSE` sans `IF` correspondant provoque `?ELSE WITHOUT IF`.

```
10 INPUT A
20 IF A>0 THEN PRINT "POSITIF" ELSE PRINT "NEGATIF OU NUL"
```

### PRINTUSING — affichage formaté

```
PRINTUSING "format",valeur[,valeur...][;]
PRINTUSING#n,"format",valeur...
```

Affiche une ou plusieurs valeurs selon un format. Un `;` final supprime le
retour à la ligne. Avec `#n`, la sortie va vers le fichier ouvert `n`
(par exemple une imprimante). Un format invalide provoque `?BAD FORMAT`.

Caractères de format repérés dans le code **(rôle exact à vérifier par des essais)** :

| Caractère | Rôle probable |
|---|---|
| `9` | Position de chiffre |
| `Z` | Chiffre, complété par des zéros |
| `.` | Point décimal |
| `,` | Séparateur / fin de champ |
| `S` | Affiche le signe |
| `+` `-` | Signe forcé |
| `$` | Symbole monétaire (le caractère est stocké en $03DE = 990, `$` par défaut) |
| `E` | Notation exponentielle |
| `A` | Champ alphanumérique (chaînes) |
| `\texte\` | Texte littéral recopié tel quel |
| `*` | Affiché quand la valeur ne tient pas dans le format |

### CALL — appel de routine machine

```
CALL adresse[,paramètre1,paramètre2...]
```

Appelle une routine en langage machine. Les paramètres (nombres, chaînes
ou variables) sont évalués et placés dans une table à partir de $0210, à
raison de 4 octets par paramètre (type, longueur, adresse), jusqu'à 15
paramètres. La routine revient au BASIC par `RTS`.

### BEEP — bip

```
BEEP
```

Émet un bip, par la routine de la ROM éditeur.

### HARDCOPY — copie d'écran

```
HARDCOPY
```

Imprime le contenu de l'écran sur l'imprimante n° 4, en convertissant les
caractères graphiques.

### MERGE — fusion de programmes

```
MERGE "nom"[,Dlecteur][,Uunité]
```

Lit un programme sur la disquette et insère ses lignes dans le programme
en mémoire ; une ligne de même numéro remplace l'existante. Syntaxe des
paramètres identique à `DLOAD`.

### FETCH — chargement

```
FETCH "nom"[,Dlecteur][,Uunité]
```

Charge un programme depuis la disquette, en mode direct ou depuis un
programme **(différences exactes avec DLOAD à vérifier)**.

### SWAP — enchaînement

```
SWAP "nom"[,Dlecteur][,Uunité]
```

Charge un autre programme et le lance **en conservant les variables** du
programme en cours. Pratique pour découper une grosse application en
plusieurs modules.

> `MERGE`, `FETCH` et `SWAP` utilisent les routines disque du BASIC 4.0 :
> elles nécessitent un lecteur de disquettes et ne fonctionnent pas avec la
> cassette.

### KILL — désactivation

```
KILL
```

Restaure le BASIC d'origine (routine CHRGET et vecteurs d'interruption).
`SYS 40960` réactive EDEX.

### PLOT, RESET

Mots-clés reconnus mais **non implémentés** dans la version 4.4 : ils
renvoient directement au `READY.` et arrêtent donc un programme en cours.

## 5. Messages d'erreur ajoutés

| Message | Cause |
|---|---|
| `?BAD FORMAT` | Format invalide dans `PRINTUSING` |
| `?ELSE WITHOUT IF` | `ELSE` sans `IF` correspondant |

## 6. Informations techniques

| Élément | Valeur |
|---|---|
| Adresse | $A000-$AFFF (4 Ko) |
| Activation | `SYS 40960` |
| Méthode | Détournement de CHRGET ($0070), du vecteur IRQ ($0090) et du vecteur BRK ($0092) |
| Mémoire de travail | $03C3-$03E1 |
| Jetons | $E2-$F5 (IF utilise le jeton standard $8B) |
| Dépendance | Routines de la ROM éditeur française ($E000), dont le bip en $E02A |

| Jeton | Mot-clé | Jeton | Mot-clé |
|---|---|---|---|
| $8B | IF | $EC | BEEP |
| $E2 | FETCH | $ED | MERGE |
| $E3 | LIST | $EE | PRINTUSING |
| $E4 | AUTO | $EF | ELSE |
| $E5 | FIND | $F0 | CALL |
| $E6 | DUMP | $F1 | TRACE |
| $E7 | ERROR | $F2 | OFF |
| $E8 | RENU | $F3 | HARDCOPY |
| $E9 | DELETE | $F4 | SWAP |
| $EA | PLOT | $F5 | KILL |
| $EB | RESET | | |

La zone $03C3-$03E1 se trouve dans le tampon de la deuxième cassette :
évitez d'utiliser la cassette n° 2 avec EDEX actif.

---

*Manuel établi par Fafa, octobre 2026, à partir de la ROM d'une CBM 8296
française. Corrections et compléments bienvenus.*
