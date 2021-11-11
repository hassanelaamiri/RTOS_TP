<h3>I. Objectif</h3>

L’objectif de ce TP est d’afficher sur les 7 segments le numéro de feu qui est en vert et 
le numéro de véhicule traverse la voie.
Pendant le traversement, les LEDs clignotent pour une durée « n » correspond à la 
durée de traversement de la voie.
<h4>1. Fonctionnement des afficheurs 7 segments :</h4>

![image](https://user-images.githubusercontent.com/93779828/141330234-4ab3fe82-c512-47b2-8f39-4a44404fb4f4.png)

<p>Notre carte est composée de 4 afficheurs 7 segments, sachant qu’ils s’allument à 0 et s’éteignent à 1.</p>
<p>Nous voudrions afficher les informations ci-dessous :</p> 

<p style="color: red">Feu 2 vert et véhicule 1 traverse : F2 1 = 8EA4FFF9</p> 
<p>Feu 2 vert et véhicule 1 traverse : F2 2 = 8EA4FFA4</p> 
<p>Feu 2 vert et véhicule 1 traverse : F2 3 = 8EA4FFB0</p> 
<p>Feu 1 vert et véhicule 1 traverse : F1 4 = 8EF9FF99</p> 
<p>Feu 1 vert et véhicule 1 traverse : F1 5 = 8EF9FF92</p> 
<p>Feu 2 vert sans véhicule traverse : F2 = 8EA4FFFF</p> 
<p>Feu 1 vert sans véhicule traverse : F1 = 8EF9FFFF</p> 

<h4>2. Fonctionnement des LEDs :</h4>
<p>Pour utiliser des LEDs, on réalise une table de vérité de 10 bits.</p>

<h5>Remarque : </h5><p>nous supposant que la durée « m » de changement des feux est supérieur 
de la durée « n » nécessaire pour que le véhicule traverse la voie.</p>

<h3>II. Création du projet logiciel MicroC / OS-II</h3>

Dans cette section, nous créons une nouvelle application Nios II et des projets BSP à 
l'aide du modèle de projet Hello MicroC/OS-II disponible dans Nios II SBT for Eclipse.

<p>1. Démarrons le Nios II SBT pour Eclipse</p>
<p>2. Dans le menu Fichier, nous choisissons New puis Nios II Application and BSP 
from Template, et une nouvelle fenêtre s’ouvert (figure 1).</p>
<p>3. Sur hardware information, Nous sélectionnons notre fichier sopcinfo qu’il nous a 
été fourni et automatiquement Le Nios II SBT pour Eclipse remplit le CPU avec le 
nom du processeur trouvé dans sopcinfo.</p>
<p>4. Dans notre cas, on choisit SE comme nom de projet, et on décoche « Use default 
location » pour qi’il soit le nom de dossier le même que le nom de projet.</p>
<p>5. Et finalement, on choisit Hello MicroC/OS-II comme modèle.</p>

![image](https://user-images.githubusercontent.com/93779828/141335056-46eb8efc-5b19-4c89-87cc-0a65146e622c.png)

<p>Le Nios II SBT for Eclipse crée deux dossiers : 
SE et SE_bsp.</p>

<p>Dans notre cas, nous travaillerons dans le dossier SE.</p>
<p>En se servant du Design IP déjà préparé (fichier sof) on programme la carte FPGA avec 
Quartus II Programmer (figure 2)</p>

![image](https://user-images.githubusercontent.com/93779828/141336352-daf637c2-f203-4213-8586-8a1e6f90195c.png)

<h3>III. Programmation</h3>
<p>Dans ce TP, on aura besoin de deux Sémaphores (MUTEX1 et MUTEX2) pour protéger 
la voie 1 et 2 pour qu’un véhicule traverse à la fois, et deux Sémaphores (FEU1 et FEU2) 
pour chaque voie.</p>
<p>MUTEX1 et MUTIX2 seront initialisé à 1, puisqu’au départ les voies sont vides.
FEU1 sera initialisé à 1 (Vert) et FEU2 à 0 (Rouge) puisqu’ils ne peuvent pas être rouge 
ou vert à la même fois.</p>
<p>D’abord, on déclare les bibliothèques qu’on aurait besoin :</p>

![image](https://user-images.githubusercontent.com/93779828/141344038-8d2ed85b-fb65-4b27-97c8-3e3af018ece6.png)


<p>Avec la commande OS_STK on déclare nos tâches, C-à-d : on attribue une tache à chaque véhicule et une tache pour le changement des feux.</p>

![image](https://user-images.githubusercontent.com/93779828/141344281-da488d76-b0be-4f8e-becb-3dacd5b6c1ff.png)

<p>On déclare les sémaphores comme pointeurs avec OS_EVENT:</p>

![image](https://user-images.githubusercontent.com/93779828/141344506-c4669422-044d-4bcc-8dc5-16caffb444b8.png)

<p>On définit les priorités. Ici on donne la priorité à la tache changement dans un premier part (la plus prioritaire est celui qu’elle a la valeur le moins).</p>

![image](https://user-images.githubusercontent.com/93779828/141344652-0317a3ca-d0fc-48ca-bd16-f6aa44ee0ad8.png)

<p>Et on déclare une variable entière de 8 bits qu’on va expliquer son rôle ultérieurement:</p>

![image](https://user-images.githubusercontent.com/93779828/141345231-99c41038-4e8e-4f87-85c0-c76385424521.png)

<p>Maintenant on passe au processus de chaque voie.</p>
<p>❖ Pour la voie 2, il y a le véhicule 1, 2 et 3.</p>
<p>Voici le programme du véhicule 1 :</p>

![image](https://user-images.githubusercontent.com/93779828/141345416-c64d1b03-bcf4-4e9d-9de3-f649edb0041d.png)

<p>Avec OSSemPend(FEU2,0,&erreur) le véhicule 1 essayera de prendre la sémaphore 
FEU2 de la voie 2.</p>
<p>Pour suivre ce qui se passe à l’exécution, on affiche dans la console « véhicule 1 prendre 
P(FEU2) » en utilisant printf("vehicule 1 prendre P(FEU2)\n")<p>

<p>Pour s’assurer qu’il n’y a pas des véhicules traversant la voie, le véhicule 1 doit prendre 
d’abord la sémaphore MUTEX 2 de la voie 2. Et c’est avec cette commande qu’on a pu 
résoudre ce problème OSSemPend(MUX2,0,&erreur).</p>
<p>On ajout ce condition après chaque prise d’un sémaphore pour en cas d’erreur on l’affiche 
dans la console:</p>

![image](https://user-images.githubusercontent.com/93779828/141345699-98225822-e17e-4e3c-bd3b-5318799bc6ef.png)

<p>Supposant que le véhicule a pu prendre les sémaphores FEU2 et MUTEX2 alors on 
ajoute une ligne pour afficher « Vehicule 1 sur voie 2 » sur la console:</p>

![image](https://user-images.githubusercontent.com/93779828/141345875-e22c0b3b-ead5-47f4-a7be-89a484f5e768.png)

<p>On affiche sur l’afficheur 7 segments le numéro de la voiture, le temps qu’il traverse la 
voie, ici dans notre cas on met 5 secondes:</p>

![image](https://user-images.githubusercontent.com/93779828/141346002-12d675ff-39aa-4aab-8479-e4fd1ff19bb8.png)

<p>Après que le véhicule a traversée la voie, il rend le sémaphore MUTEX2 avec la 
commande ci-dessous:</p>

![image](https://user-images.githubusercontent.com/93779828/141346153-581e0c97-3d46-44b3-aebe-18e8fd623bd9.png)

<p>Et on affiche dans la console:</p>

![image](https://user-images.githubusercontent.com/93779828/141346270-0a24aeee-e263-4b49-8b75-60a1cd6bb84a.png)

<p>puis elle rend le sémaphore FEU2 avec la commande:</p>

![image](https://user-images.githubusercontent.com/93779828/141346418-6660279e-4995-4cd8-bfe7-818164258dde.png)

<p>Finalement, on tue la tâche avec la commande ci-dessous après que le véhicule a 
traversée la voie:</p>

![image](https://user-images.githubusercontent.com/93779828/141346524-e527d7a8-2134-42ac-a628-717b00fa1a72.png)

<p>On programme le même code pour tous les véhicules en modifiant juste le paramètre 
d’affichage et les sémaphores des feux pour chaque voie.</p>

<h4>Pour Véhicule 2 :<h4>

![image](https://user-images.githubusercontent.com/93779828/141346720-8b1d89be-0ecb-4858-8bde-78168f8a7127.png)

<h4>Pour Véhicule 3 :<h4>
  
![image](https://user-images.githubusercontent.com/93779828/141346828-5c6e9213-bf9f-439f-b426-43175aec0581.png)

<p>❖ Pour la voie 1, il y a le véhicule 4 et 5.</p>
 
<h4>Pour Véhicule 4 :<h4>
  
![image](https://user-images.githubusercontent.com/93779828/141347203-fd7fe1e4-6b24-4c50-934f-cdf89f2a3863.png)

<h4>Pour Véhicule 5 :<h4>
  
![image](https://user-images.githubusercontent.com/93779828/141347321-30f8bec8-9a94-4c2d-8c51-9afd11336c44.png)

<p>Maintenant après qu’on a terminé de programmer les véhicules on passe à la 
programmation au niveau des feux.</p>
<p>Donc on aura besoin d’une seule tâche qu’on va appeler « changement » pour contrôler 
les feux des deux voies</p>

<p>Pour le programme de la tâche changement:</p>

![image](https://user-images.githubusercontent.com/93779828/141347645-ddf935aa-ded5-4666-a855-4ef80b7c8b07.png)

<p>Comme on a supposé auparavant que le feu 2 de la voie 2 est en rouge et feu 1 de la 
voie 1 est vert, on affichera dans un premier part sur l’afficheur 7 segments F1 (cad : feu 
1 en verts) pendant 7 secondes et dans la console également avec ces commandes ci-dessous</p>
  
![image](https://user-images.githubusercontent.com/93779828/141347847-d95f3aa0-ba5f-4915-9b2c-1c5959134163.png)

<p>Après les 7 secondes la tâche changement essayera de prendre premièrement la 
sémaphore FEU 1 pour que le feu de la voie 1 soit en rouge, puis rendre la sémaphore 
FEU 2 et donc feu en vert dans la voie 2, en affichant sur les 7 segments et la console le 
changement des feux:</p>
  
![image](https://user-images.githubusercontent.com/93779828/141347982-54a663e1-e66c-4e6e-99de-1b32c6cddb1f.png)

<h4>Remarque</p>
<p>Cette tâche, on ne la tue pas comme on a fait pour les autres tâches, parce que les feux 
fonctionnent infiniment et les véhicules attendent juste leurs tours pour traverser 
Carrefour</p>
  
<h4>La fonction main</h4>
<p>Finalement on écrit notre programme de la fonction main pour créer nos tâches et lancer 
l’OS.<p>
  
![image](https://user-images.githubusercontent.com/93779828/141348548-7a1c86c1-fe40-4f21-b388-85cc737160c9.png)

<p>Ici, on initialise les sémaphores, et on crée et attribue à chaque tâche qu’on a déclaré 
auparavant une priorité selon notre cahier de charge. Ensuite on lance notre programme 
OS avec la commande OSStart().<p>

<h3>IV. Conception</h3>
<p>Pour compiler le code on se rend dans Project > Build All.
Après que le code successivement compilé, on se rend sur Run > Run As pour l’exécuter 
sur notre cible FPGA.</p>
<p>Voici les informations aperçues sur la console:</p>
  
![image](https://user-images.githubusercontent.com/93779828/141349100-fca67428-686c-49ca-98f1-11a1a8e20ccd.png)
  
<h4>Sur notre carte FPGA:</h4>
<p>Pour faire un peu d’animation (figure 2) on remplace le délai m par ce code, qui permet 
au lieu de juste mettre la tâche en non-activable, on allume une LED chaque 0.5 
seconde:</p>
  
![](https://media.tenor.com/images/a0a693dec7660a6df1d2937b057a7b71/tenor.gif)
![myfile](https://media.tenor.com/images/a0a693dec7660a6df1d2937b057a7b71/tenor.gif)
