---
title: PRA
description: 
published: 1
date: 2023-04-21T13:25:12.386Z
tags: 
editor: markdown
dateCreated: 2023-04-21T13:25:09.342Z
---

# PRA
 *(exemple alexandre)*

## Pour **TESTER** le PRA uniquement : 

- déconnecter le lien sur dps59439 <code>
drbdadm disconnect vdedicated
</code>

- Puis Suivre les étapes suivantes :

  - Sur dps92856 ####

    * basculer en primaire
    <code>drbdadm primary vdedicated</code>
    
    * Monter le volume
    <code>mount /dev/drbd0 /var/vdedicated/</code>

    * Charger les définitions de VM puis les démarrer : <code>
cd /var/vdedicated/xml
virsh define dps59439.xml
virsh define ds-192-163-ADREA.xml
virsh define DS-192-117-ADREA2.xml
virsh start dps59439.xml
virsh start ds-192-163-ADREA.xml
virsh start DS-192-117-ADREA2.xml</code>

    * Vérifier que tout fonctionne :

---

## Arrêter le PRA et retour à la normale 

- Arrêt des VMs <code>virsh shutdown dps59439
virsh shutdown ds-192-163-ADREA
virsh shutdown DS-192-117-ADREA2</code>

- Ménage dans l'hyperviseur <code>virsh undefine dps59439
virsh undefine ds-192-163-ADREA
virsh undefine DS-192-117-ADREA2</code>

- Umount du volume <code>cd;umount /var/vdedicated</code>

- Basculer le volume en secondaire <code>
drbdadm secondary vdedicated</code>

- dps59439 
  - En cas de test il faut reconnecter le lien <code>
drbdadm connect vdedicated</code>

  - Mettre le volume en primaire <code>
 drbdadm primary vdedicated</code>

  - Mount du volume <code>
mount /dev/drbd0 /var/vdedicated/</code>

  - Démarrer les VM <code>
virsh start dps59439
virsh start ds-192-163-ADREA
virsh start DS-192-117-ADREA2</code>

---
## La méthode PRA automatique :

- Pour démarrer les VM, depuis le serveur PRA **dps92856.produhost.net**  : exécuter le script **bascule_pra.sh**
- Pour arrêter le PRA et remettre le serveur PRA à l'état initiale sur **dps92856.produhost.net**  : **arret_pra.sh**
- Pour redémarrer les VM normalement, depuis le serveur **dps59439**  : //restore_pra.sh//



---
---


