# bars-angular

Front-end du site des bars d'étages développé par le Binet Réseau (le back-end étant Nadrieril/bars-django).

## Installation
```
npm install
bower install
```

## Configuration
L'URL du back-end peut être modifiée dans le fichier app/app.js, via les variables APIURL.url et OFFURL.url.

## Utilisation
```
grunt serve
```

## Organisation
Tout ce qui est intéressant se trouve dans le dossier app/. index.html n'est pas particulièrement utile et est quasi-entièrement généré par grunt ; assets/ contient les images et le CSS.
common/ contient le layout général ; common/bars.js contient le controleur de la page listant tous les bars ; common/main.js est la porte d'entrée dans un bar : il contient le chargement de toutes les données qui sont ensuites mises en cache, le controleur de la page d'accueil d'un bar.

### components/
* components/ contient tout ce qui est intéressant.
* admin/ contient la partie administration d'un bar ;
* auth/auth.js contient la gestion de la connexion et du token ;
* magicbar/ contient le code concernant la MagicBar™ ;
* meal/ contient la partie bouffe à plusieurs ;
* root/ contient la partie root (l'administration globale de tous les bars et objets globaux) ;
* settings/ contient la page Préférences d'un utilisateur ;
* et surtout API/ contient tout le reste (liste des aliments, des utilisateurs...).

### components/API/
Chaque dossier dans API/ correspond à un objet dans l'api. Le mieux pour avoir des infos à jour est de consulter l'API : http://bars.nadrieril.fr/api/.

#### Fonctionnement des aliments
On achète des ItemDetails qui ont chacun un (ou des) code-barre(s). On vend des SellItem, dans une certaine unité. Les StockItem, liés à un ItemDetails, stockent la quantité restante dans le bar.
On achète des pots de 830g de Nutella, des pots de 1kg de Nutella et des pots de 5kg de Nutella. On a donc 3 ItemDetails : “Pot de 830g de Nutella”, “Pot de 1kg de Nutella” et “Pot de 5kg de Nutella”. On a aussi, dans notre bar, les 3 StockItem associés.
On ne veut pas vendre le Nutella par pot mais par gramme ; on crée donc un SellItem “Nutella” avec l’unité (unit_name) g (gramme), qui regroupe les StockItem correspondants dans un bar. Si dans le bar on a un un pot de chaque, quand on achète 30g de SellItem “Nutella” on va répartir ces 30g entre chaque pot. Mais il faut savoir à quoi correspondent n grammes par rapport à “Pot de 830g de Nutella”.
La propriété sell_to_buy de StockItem indique la relation entre “n grammes de SellItem Nutella” et “m pots de 830g de Nutella”. On a simplement m = n * sell_to_buy. Une unité de SellItem correspond à sell_to_buy unités de ItemDetails. Si on change l’unité de vente (SellItem.unit_name), on change les facteurs sell_to_buy.

Mais, magie, le serveur cache ces calculs. Tout les échanges se font en unité de SellItem. StockItem.qty est en unité de SellItem, SellItem.fuzzy_qty également, dans les transactions, toutes les quantités sont en unité de SellItem, quand on fait un achat ou un inventaire (que ça concerne un StockItem ou un SellItem) on envoie la quantité en unité de SellItem.
