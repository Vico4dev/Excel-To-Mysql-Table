# Guide d'utilisation du script d'importation Excel vers MySQL

Ce guide explique comment utiliser le script Python pour importer des données d'un fichier Excel dans une base de données MySQL. Le script offre une interface graphique conviviale pour faciliter le processus d'importation, y compris la gestion des types de données et le traitement des erreurs potentielles.

---

## Table des matières

- [Fonctionnalités du script](#fonctionnalités-du-script)
- [Prérequis](#prérequis)
- [Installation des dépendances](#installation-des-dépendances)
- [Configuration de la base de données MySQL](#configuration-de-la-base-de-données-mysql)
- [Utilisation du script](#utilisation-du-script)
- [Explication du fonctionnement du script](#explication-du-fonctionnement-du-script)
- [Gestion des erreurs et des problèmes courants](#gestion-des-erreurs-et-des-problèmes-courants)
- [Remarques importantes](#remarques-importantes)

---

## Fonctionnalités du script

- **Interface graphique conviviale** : Le script utilise Tkinter pour offrir une interface utilisateur simple.
- **Importation depuis Excel** : Permet de sélectionner un fichier Excel (`.xlsx` ou `.xls`) à importer.
- **Gestion des types de données** : Infère automatiquement les types de données et permet à l'utilisateur de les ajuster.
- **Renommage des colonnes** : Offre la possibilité de renommer les colonnes avant l'importation.
- **Prise en charge des types de données courants** :
  - Texte
  - Nombre entier
  - Nombre décimal
  - Date
  - Coordonnées GPS
  - Booléen
- **Vérification des noms de colonnes et de table** : Empêche l'utilisation de mots réservés SQL ou de noms invalides.
- **Gestion des valeurs manquantes** : Remplace les valeurs manquantes par `NULL` dans la base de données.
- **Affichage des erreurs détaillées** : Fournit des messages d'erreur clairs pour faciliter le débogage.

---

## Prérequis

- **Python 3.x** installé sur votre machine.
- **MySQL** installé et configuré sur votre machine.
- **Accès à une base de données MySQL** avec les informations suivantes :
  - Hôte : `localhost`
  - Port : `3306`
  - Nom d'utilisateur : `root`
  - Mot de passe : *(pas de mot de passe)*
  - Nom de la base de données : `ma_base_de_donnees`

---

## Installation des dépendances

Le script nécessite plusieurs bibliothèques Python. Vous pouvez les installer en exécutant la commande suivante dans votre terminal :
  - pandas : Pour lire et manipuler les fichiers Excel.
  - SQLAlchemy : Pour interagir avec la base de données MySQL.
  - openpyxl : Pour lire les fichiers Excel au format .xlsx.
  - pymysql : Pilote pour connecter SQLAlchemy à MySQL.

```bash
pip install pandas sqlalchemy openpyxl pymysql

```

## Remarques importantes
Sécurité :

 - Il est fortement recommandé de ne pas utiliser l'utilisateur root sans mot de passe pour des raisons de sécurité.
Créez un utilisateur dédié avec un mot de passe et des permissions limitées pour vos opérations d'importation.
Encodage des caractères :

 - Assurez-vous que votre base de données MySQL est configurée avec l'encodage utf8mb4 pour gérer correctement les caractères spéciaux et les emojis.
## Personnalisation du script :

Vous pouvez ajuster la fonction map_sql_type pour mapper d'autres types de données ou personnaliser les types existants en fonction de vos besoins.
Vérification des données :

Avant d'importer, vérifiez que vos données sont cohérentes et qu'elles correspondent aux types de données sélectionnés.

@2024 Vico4dev