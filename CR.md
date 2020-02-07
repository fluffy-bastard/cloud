# Compte Rendu - Cloud
## IRC 4 - Groupe 8

![enter image description here](https://raw.githubusercontent.com/fluffy-bastard/cloud/master/84330640_1162120863994310_8195569217412530176_n.png)

### I. Configuration du switch Cisco
Communication en série avec le switch, possibilité de se connecter en ssh en configurant les line vty
Création des vlan ->

    vlan 10
    name stockage
    vlan 20
    name management
    ...
	
configuration des interfaces du switch
pour les liens trunk (ex: lien ESXi ou pfSense) ->

    switchport mode trunk (possibilité de selectionner les vlans autorisé avec la commande "allowed vlan")

pour les liens connecté a une machine d'un réseau virtuel (ex: ordinateur portable) ->
	
    switchport mode access
    switchport access vlan 10 (changer le nombre en fonction du vlan)
### II. Installation ESX et VM
Booter sur la clé USB
Installer l'hyperviseur
Une fois l'installation terminée, renseigner la configuration réseau de l'ESX
Le reste de la configuration se fait via l'interface web de l'ESX

#### Installation Windows Server 2012:
Specs:
- 1 CPU
- 4gb RAM
- 40 go
- 3 cartes réseaux (réseau Prod, Management et Stockage)

Upload du contenu de la clé USB dossier "materials" dans le datastore de l'ESX => Les ISOs sont maintenant à disposition pour chaque VM de l'ESX

Installation de l'OS dans la VM
Installation des vmware tools
Reboot

##### Ajout des rôles sur le serveur: 

     "Server Manager" > installation du rôle AD + DNS

Création du "vmware.cpe"

La politique de sécurité CPE bloque les DNS autres que ceux de l'école, il faut donc renseigner le DNS CPE en tant que forwarder:

    "Server Manager" > DNS > DC > Forwarders > ajout DNS CPE (134.214.49.16 et 134.214.49.19)

#### Installation pfSense
Cette VM fait office de routeur et de pare-feu. Elle partage le WAN CPE avec nos équipements dans les différents VLAN. Une règle NAT permet de translater les adresses des réseaux privés ESX vers Internet.
Specs:
- 1 CPU
- 512 Mo RAM
- 5 go
- 2 cartes réseaux:
		LAN "vm0" en trunk -> bridge
		WAN carte "cpe" -> NAT Network
		Promiscuity mode "allow VM"

Création de 3 sous interfaces VLAN (Management - Stockage - Prod) pour le port LAN (mode trunk)

#### Installation FreeNAS et configuration iSCSI:
Specs:
- 4 CPU
- 8go RAM
- Disque 250 go
- Disque 350 go
- 2 cartes réseaux: 
		- 1 carte management 
		- 1 carte stockage

##### Installation du logiciel
Ajout d'un disque dans le "pool"
Découpage en 3 zvol (ZFS) de 35, 20 et 140 GB

##### Activation du service "iSCSI"
Sharing > iSCSI
Utilisation du "Wizard" > choix du LUN
Création d'un "portail" pour autoriser les connexions
Discovery auth method: none
IP d'écoute : 0.0.0.0 (toutes interfaces)

##### Sur les ESX:
Configure > Storage Adapters > Add Software Adapter type iSCSI
Dans "Dynamic discovery" ajouter l'adresse IP du serveur FreeNAS (172.16.0.20:3260 pour nous)
puis faire un rescan "storage device"
Réaliser cette opération sur les 2 ESX

Il faut ensuite ajouter les LUN dans 
ESX > Storage "new datastore" > VMFS > Sélectionner le LUN

On peut ensuite migrer les disques des VM en faisant 
"migrate" > "Change storage only" et sélectionner le LUN concerné


 #### Installation vCenter

![enter image description here](https://raw.githubusercontent.com/fluffy-bastard/cloud/master/84306333_2589054144642802_4635453637299011584_n.png)

Version du vcenter : 6.7 (dernière version disponible actuellement)

Montage de l'ISO à partir d'une machine virtuelle hebergée par un des deux ESXi
Lancement du déployement de vCenter par l'interface graphique
Remplir les options du wizard de l'étape 1 (voir image)
Remplir les options du wizard de l'étape 2 (voir image)

#####  Création cluster dans vCenter
Pour commencer il faut créer un centre de données : click droit sur le vCenter -> nouveau centre de données
Pour créer le cluster : click droit sur le centre de donnée -> nouveau cluster -> choisir les options du cluster a activer (DRS,HA,vSAN)
Ensuite pour ajouter les 2 ESXi a la grappe (cluster): click droit sur le cluster -> ajouter des hotes -> selectionner les 2 serveurs

#####  Création des réseau virtuels
Sur les ESXi : choisir configurer -> Commutateur virtuels -> Ajouter une mise en réseau
Créer autant de mise en réseau qu'il y a de VLAN et d'adaptateur VMKernel pour pouvoir connecter les ESXi aux différents réseaux (voir schéma réseau)

#####  Configuration DRS
Prérequis : le DRS doit etre activé sur le cluster
Selectionner le cluster -> configurer -> vSphere DRS -> modifier -> choisir la sensibilité, activer le monitoring du processeur
Il est également possible de créer des taches planifiés DRS dans le menu Planifier DRS
Le DRS fonctionne bien avec une configuration rapide à mettre en place

#####  Update Manager
Dans le menu de vCenter -> acceder aux raccourcis -> choisir Update Manager
Aller dans mise a jour -> recherche et télécharger les mise à jour pour le système

![enter image description here](https://raw.githubusercontent.com/fluffy-bastard/cloud/master/85165995_2476795059207050_2873305371240300544_n.png)









