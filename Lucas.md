# TP 6 - Boot, services et processus / Tâches
d’administration (2)

## Exercice 1. Personnalisation de GRUB

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).**

J'ai fait clique droit configuration -> stockage -> créer un nouveau disque dur virtuel 

**2. Vérifiez que ce nouveau disque dur est bien détecté par le système**

J'ai fait `fdisk -l`  

**3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)**

J'ai d'abord fait: <br> - `fdisk /deb/sdb` puis j'ai mis `n`puis `p` `1` puis entrée puis `+2G` pour choisir 2Go. <br>

J'ai refait les mêmes commande pour la seconde partition mais j'ai mis `+3G` et ensuite une fois créer on appuie sur t et on choisi la seconde partition le numéro 7 (NTFS)

**4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)**

J'ai fait la commande `mkfs.ext4 /dev/sdb1` pour la première partition 
J'ai fait la commande `mkfs.ntfs /dev/sdb2` pour la deuxième partition 

**5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?**

La commande df -T n'affiche pas le disque sdb car il n'est pas encore monté

**6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)**

`nano /etc/fstab`
on ajoute :
```
#device        mountpoint             fstype    options    dump   fsck
/dev/sdb1      /data                   ext4     defaults     0       0
/dev/sdb2      /win                    NTFS     defaults     0       0
```

**7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration**

La commande mount est la suivante `mount -a`. Oui les modifications sont permanentes même après le redémarage de la machine

**8. Montez votre clé USB dans la VM**

**9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox**

## Exercice 2. Partitionnement LVM

Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler les disques et les partitions.

**1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab**

j'ai fait la commande `umount /data` et `umount /win` pour démonter les systèmes de fichiers 

**2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM**

j'ai fait la commande `fdisk /dev/sdb` puis d pour supprimé les deux partitions

**3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.**

j'ai fait `pvcreate /dev/sdb1` 
j'ai créer une partitition normalement et ensuite j'ai fait `t` puis j'ai tapé le code `8E` 

**4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.**

j'ai fait `vgcreate volume1 /dev/sdb1` 

**5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.**

j'ai fait la commande `lvcreate -l 100%FREE -n 1vData volume1` 

**6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.**

j'ai fait fdisk `/dev/mapper/volume1-1vData` puis `mkfs.ext4 /dev/mapper/volume1-1vData`
j'ai rajouté dans le fstab la ligne `/dev/mapper/volume1-1vData      /data            ext4     defaults     0       0` 

**7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.**

Fait

**8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes**

j'ai fait la commande  `vgextend volume1 /dev/sdc1` 

**9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.**

j'ai fait la commande `lvextend -l 100%FREE /dev/volume1/1vData` puis `resize2fs /dev/volume1/1vData` 

## Exercice 3. Exécution de commandes en différé : at et cron

**1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez
entre temps que la tâche est bien programmée.**

**2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par
exemple en vous aidant des logs, du manuel...)**

**3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple,
l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.**

**4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure**

**5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18
heures les 1er et 15 du mois :**

**6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures**

**7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un
fichier de log situé dans votre dossier personnel**

**8. Videz votre crontab**
