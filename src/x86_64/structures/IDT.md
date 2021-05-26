# Interrupt Descriptor Table

La table de description des interruptions est une table qui nous permet de faire appeler au cpu une fonction pour chaque type d'interruption.

L'IDT est composée de trois structures (en 64 bits):

- Le `pointeur de la table d'interruptions`
- La `table de description d'interruptions`, qui est une liste d'entrées. C'est ce que l'on appelle plus communément IDT.
- L' `entrée d'interruption`, qui est une entrée de la table d'interruptions.

## Table d'entrée

Le pointeur de la table de description d'interruptions contient une adresse qui situe une IDT et la taille de la table (en mémoire).

pour le pointeur de l'IDT, la structure est comme ceci :

|nom                    |taille     |
|-----------------------|-----------|
|taille                 | 16 bit    |
|addresse de la table   | 64 bit    |

la table d'entrée peut être définie comme ceci:
```c
IDT_entry_count = 64; 
IDT_Entry_t IDT[IDT_entry_count];
IDT_table.addr = (uint64_t)IDT;
IDT_table.size = sizeof(IDT_Entry_t) * IDT_entry_count;
```

## Entrée d'interruption

Une entrée d'IDT doit être structurée comme ceci :

|nom                        |taille     |
|---------------------------|-----------|
|offset (0-16)              | 16 bit    |
|segment de code            | 16 bit    |
|numéro d'interruption      | 8 bit     |
|attributs                  | 8 bit     |
|offset (16-32)             | 16 bit    |
|offset (32-64)             | 32 bit    |
|zéro                       | 32 bit    |

le `segment de code` étant le segment de code utilisé pendant l'interruption.

l'`offset` est l'addresse où le CPU va jump si il y a une interruption. Cette addresse est généralement une fonction, qui sera appelée lors de l'interruption correspondante.

### Les attributs 
l'attribut d'une entrée d'une IDT est formée comme ceci : 

|nom                    |bit        |
|-----------------------|-----------|
|type d'interruption    | 0 - 4     |
|zéro                   | 5         |
|niveau de privilège    | 6 - 7     |
|présent                | 8         |

le `niveau de privilège` (aka DPL) est le niveau de privilège requis pour que l'interruption soit appelée.

il est utilisé pour éviter à ce que une application utilisatrice puisse appeller une interruption qui est réservée au kernel


### Types d'interruptions

les types d'interruption sont les mêmes que cela soit en 64 bits ou en 32 bits. 

|valeur                 |signification                          |
|-----------------------|---------------------------------------|
|0b0111 (7)             | trappe d'interruption 16bit           |
|0b0110 (6)             | porte d'interruption  16bit           |
|0b1110 (14)            | porte d'interruption  32bit           |
|0b1111 (15)            | trappe d'interruption 32bit           |

La différence entre une `trappe`(aka trap) et une `porte` (aka gate) est que la porte désactive `IF`, ce qui veut dire que vous devrez réactiver les interruptions à la fin de l'ISR.

la trappe ne désactive pas `IF` donc vous pouvez désactiver / réactiver vous même dans l'ISR les interruptions.

