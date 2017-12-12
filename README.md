# eedomus script : Nuki smartlock

![Nuki Logo](./dist/img/nikya_nukismartlock.png "Logo Nuki smartlock by Nikya")

* Plugin version : 1.0
* Origine : [GitHub/Nikya/nuki_smartlock](https://github.com/Nikya/eedomusScript_nuki_smartlock "Origine sur GitHub")
* Nuki Bridge HTTP-API : 1.6 ([API documentation](https://nuki.io/fr/api/))

## Description
***Nikya eedomus Script Nuki Smartlock*** est un plugin pour la box domotique eedomus, qui permet de piloter et connaitre l'�tat d'une serrure intelligent _Nuki_.

Ce plugin est compos� d'un script PHP et d'une d�claration pour 3 p�riph�riques :
- Commande d'ouverture/fermeture
- �tat de la serrure
- Indicateur de batterie faible

Son avantage principal est de mettre � jour l'�tat de la serrure, seulement si n�cessaire, en utilisant la fonctionnalit� _callback_ de l'API Nuki. (au lieu de cr�er des _polling_ c�t� eedomus)

**Nota** : Suite � des contraintes techniques impos�es par la box :
* Ce plugin ne sait g�rer qu'une seule serrure � la fois pour le moment (si besoin, le script PHP peut �tre dupliqu� pour en g�rer d'autres)
* Suite � un changement d'�tat de la serrure, la mise � jour cot� eedomus prend au minimum 30 secondes.

## Pr�requis

Une serrure Nuki Smartlock et son bridge (Mat�riel ou logiciel)

## Installation via store

Depuis le portail _eedomus_, cliquez sur
- `Configuration`
- `Ajouter ou supprimer un p�riph�rique`
- `Store eedomus`
- puis s�lectionner _Nuki Smartlock_

Des informations seront demand�es pour la cr�ation du plugin. Puis noter les **codeAPI** des p�riph�riques cr�s.

## Installation manuelle

1. T�l�charger le projet sur GitHub : [GitHub/Nikya/nuki_smartlock](https://github.com/Nikya/eedomusScript_nuki_smartlock "Origine sur GitHub")
1. Uploader le fichier `dist/nukismartlock.php` sur la box ([Doc eedomus script](http://doc.eedomus.com/view/Scripts#Script_HTTP_sur_la_box_eedomus))
2. Cr�er manuellement les 3 p�riph�riques et noter leur **codeAPI**

### Param�trage

Informations � prendre en note, car � r�utiliser ult�rieurement.

1. **Discovery** : Appeler l'URL suivante pour connaitre l'IP et le port local de votre Bridge
	* URL : https://factory.nuki.io/discover/bridges.
	* R�sultat : Une **IP** et un **port**
2. **Get Token (auth)** : S'authentifier sur le brige, avec l'IP et le port obtenu pr�c�demment, en appelant l'URL suivante et en confirmant par un **appui sur le bouton physique du bridge**.
 	* URL : http://192.168.1.50:8080/auth
 	* R�sutat : Un token
3. **Setup script** : Configurer le script eedomus, avec les informations obtenues, en appelant la _fonction setup_ (Voir ci-apr�s)
5. **Register script** : Configurer le script eedomus, avec les informations obtenues, en appelant la _fonction register_  (Voir ci-apr�s)

### Les fonctions du script

Executer le script eedomus en pr�cisant une `function`.

* Format : https://[ip_box_eedomus]/script/?exec=nukismartlock.php&function=
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=toto

#### Fonction _setup_

Configurer ce script.

* params
	- function : `setup`
	- nukihost_port : IP et Port du bridge Nuki au format `ip:port`
	- token : Token d'identification
* R�sultat
	- (XML) Un listing des �quipements trouv�s sur le bridge cibl� (noter le **Nuki ID**)
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=setup&nukihost_port=192.168.1.50:8080&token=909090

#### Fonction _register_

Abonner la box eedomus en tant que _Callback_ souhaitant �tre inform� des changements d'�tat de la serrure.

* params
	- function : `register`
	- eedomushost : IP de votre eedomus qu'appelera le bridge Nuki (Na pas mettre localhost !)
	- nukiid : Id du Nuki (Voir _fonction list_)
	- periph_id_state : Code API eedomus du p�riph�rique qui contiendra l'information _ETAT_ de la serrure
	- periph_id_batterycritical : Code API eedomus du p�riph�rique qui contiendra l'information _Batterie faible_ de la serrure
* R�sultat
	- (XML) Une confirmation ou non du succ�s de la fonction
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=register&eedomushost=192.168.1.60&nukiid=111&periph_id_state=222&periph_id_batterycritical=333

#### Fonction _list_

Lister les �quipements connus par le bridge Nuki cibl�

* params :
	- function : `list`
* R�sultat
	- (XML) Listing
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=list

#### Fonction _callback list_

Lister les callback enregistr�s par le brige Nuki.

* params :
	- function : `callback_list`
* R�sultat
	- (XML) Listing des �quipements
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=callback_list

#### Fonction _callback remove_

Supprimer un callback enregistr� sur le Bridge Nuki.  
(Utile pour pallier � l'�ventuel erreur _too many callbacks registered_)

* params :
	- function : `callback_remove`
	- id : Id du callback � supprimer (obtenue avec la _fonction callback list_)
* R�sultat
	- (XML) Listing des �quipements
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=callback_remove&id=222

#### Fonction _incomingcall_

Fonction coeur de ce script, c'est cette fonction qu'appellera le bridge Nuki � chaque changement d'�tat d'une serrure.  
Elle lis les informations re�ues et met � jour les p�riph�riques concern�s avec les nouvelles valeurs.  
Inutile de l'appeler manuellement, mais un appel permet de savoir si l'ensemble est correctement configur� et op�rationnel.

* params :
	- function : `incomingcall`
* R�sultat
	- (XML) R�sultat des valeurs lues
* Exemple : https://192.168.1.60/script/?exec=nukismartlock.php&function=incomingcall

### Valeurs possibles

#### Pour _periph value batterycritical_

* 0 : Batterie non faible
* 100 : Batterie faible

#### Pour _periph value state_

ID  | Name
----|-----------------------
0   | uncalibrated
1   | locked
2   | unlocking
3   | unlocked
4   | locking
5   | unlatched
6   | unlocked (lock �n� go)
7   | unlatching
254 | motor blocked
255 | undefined
