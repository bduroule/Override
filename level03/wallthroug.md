
# Level 03

En exutant l'exucutable on peut remarquer que ce dernier nous demande un mot de passe, en entrant une valeur aleatoire on constate que ce dernier nous revoie un message d'erreur 
```
level03@OverRide:~$ ./level03 
***********************************
*               level03         **
***********************************
Password:test

Invalid Password
level03@OverRide:~$ 
```
On peut voire en décompilant l'executable dans cutter ou gdb qu'il ya une fonction main qui prend en paramètre un int récupérer par un `scanf` comme password, on remarquent également qu'il y a un nombre en hexadecimal `0x1337d00d` (`322424845`) 
```
(gdb) disas main 
Dump of assembler code for function main:
   0x0804885a <+0>:     push   %ebp
   0x0804885b <+1>:     mov    %esp,%ebp
   0x0804885d <+3>:     and    $0xfffffff0,%esp
   0x08048860 <+6>:     sub    $0x20,%esp
   0x08048863 <+9>:     push   %eax
   0x08048864 <+10>:    xor    %eax,%eax
   0x08048866 <+12>:    je     0x804886b <main+17>
   0x08048868 <+14>:    add    $0x4,%esp
   0x0804886b <+17>:    pop    %eax
   0x0804886c <+18>:    movl   $0x0,(%esp)
   0x08048873 <+25>:    call   0x80484b0 <time@plt>
   0x08048878 <+30>:    mov    %eax,(%esp)
   0x0804887b <+33>:    call   0x8048500 <srand@plt>
   0x08048880 <+38>:    movl   $0x8048a48,(%esp)
   0x08048887 <+45>:    call   0x80484d0 <puts@plt>
   0x0804888c <+50>:    movl   $0x8048a6c,(%esp)
   0x08048893 <+57>:    call   0x80484d0 <puts@plt>
   0x08048898 <+62>:    movl   $0x8048a48,(%esp)
   0x0804889f <+69>:    call   0x80484d0 <puts@plt>
   0x080488a4 <+74>:    mov    $0x8048a7b,%eax
   0x080488a9 <+79>:    mov    %eax,(%esp)
   0x080488ac <+82>:    call   0x8048480 <printf@plt>
   0x080488b1 <+87>:    mov    $0x8048a85,%eax
   0x080488b6 <+92>:    lea    0x1c(%esp),%edx
   0x080488ba <+96>:    mov    %edx,0x4(%esp)
   0x080488be <+100>:   mov    %eax,(%esp)
   0x080488c1 <+103>:   call   0x8048530 <__isoc99_scanf@plt>
   0x080488c6 <+108>:   mov    0x1c(%esp),%eax
   0x080488ca <+112>:   movl   $0x1337d00d,0x4(%esp)
   0x080488d2 <+120>:   mov    %eax,(%esp)
   0x080488d5 <+123>:   call   0x8048747 <test>
   0x080488da <+128>:   mov    $0x0,%eax
   0x080488df <+133>:   leave  
   0x080488e0 <+134>:   ret    
End of assembler dump.
(gdb) 
```
On voit dans que la fonction `test` appelle la fonction `decrypte` on remarque également qu'a l'entré de `test` on soustrait notre entré à `0x1337d00d` la fonction fait ensuite un switch qui vas tester toute les valeur entre 1 et 9 et et entre 16 et 21 chacune de ces condition lance un `decrypte`  qui vas nous permettre d'accéder au prompt. On voit que la fonction `decrypt` une string *"Q}|u'sfg~sf{}|a3"* celle-ci vas être modifier a l'aide du de `0x1337d00d`. Puis un chaine `"Congratulations!"` est comparée avec la string précédemment créée pour nous donner accès au prompt. Il faut donc 

Il nous faut donc tester toutes les solutions possible de  `0x1337d00d - 1`  à  `0x1337d00d - 9`  et de  `0x1337d00d - 16`  à  `0x1337d00d - 21`. On va donc créer un programme en python pour nous facilité la tache.
```py
import os
    
nb = [1,2,3,4,5,6,7,8,9,16,17,18,19,20,21]
hex = 0x1337d00d
    
for n in nb:
  arg = hex - n
  print(str(n), str(arg))
  os.system("(echo "+ str(arg) + "; cat) | ./level03")
```
La 18eme fonctionne et nous ouvre le prompt il rest qu'a cat le .pass
```
level03@OverRide:~$ python /tmp/test.py 
[...]
cat: write error: Broken pipe
('18', '322424827')
***********************************
*               level03         **
***********************************
whoami 
level04
cat /home/users/level04/.pass
kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
```