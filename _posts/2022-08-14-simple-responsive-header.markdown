---
layout: post
title:  "Un header responsive simple avec CSS grid"
date:   2022-08-14 11:38:00 +0200
categories: CSS
tags: Responsive
lang: fr
---

Ce tutoriel vous permettra de créer un header responsive simple en HTML, CSS et Javascript natif.
Le header utilise CSS grid et gère le dark et light mode.

Voir la démo sur [Code Pen](https://codepen.io/gaellan/pen/vYRVJeP).

## Le HTML

Commençons par créer un fichier `index.html` à la racine du projet.

Le contenu de `index.html` :

{% highlight html %}
<!DOCTYPE html>
<html lang="fr">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Simple Responsive Header</title>
        <link rel="stylesheet" href="assets/css/style.css" />
    </head>
    <body>
        <header>
            <a class="sr-only" href="#main-title">Accéder directement au contenu</a>
            <button id="mobile-header-button">
                <span class="bi-list"></span>
                <span class="sr-only">Ouvrir le menu</span>
            </button>
            <nav id="main-nav">
                <ul>
                    <li>
                        <a href="/" title="Accueil">
                            <h1>Nom du site</h1>
                        </a>
                    </li>
                    <li>
                        <a href="" title="Page À propos">À propos</a>
                    </li>
                    <li>
                        <a href="" title="Accueil du blog">Blog</a>
                    </li>
                </ul>
            </nav>
        </header>
        <main id="home">
            <h2 id="main-title">

            </h2>
        </main>
        <footer>

        </footer>
        <script type="application/javascript" src="assets/javascript/index.js"></script>
    </body>
</html>
{% endhighlight %}

L'idée pour nous est d'afficher le header horizontalement sur les grand écrans et verticalement au clic sur un bouton sur les mobiles et tablettes.

Nous ajoutons un lien direct vers le contenu 

{% highlight html %}
<a class="sr-only" href="#main-title">Accéder directement au contenu</a> 
{% endhighlight %}

pour permettre aux personnes utlisant un lecteur d'écran de passer la navigation si ielles le souhaitent.

## Le CSS

Pour bien organiser nos fichiers nous créons un dossier `assets` à la racine du projet puis un dossier `assets/css` qui contiendra notre style.

Nous créons ensuite un fichier `assets/css/style.css` qui fera des imports des autres fichiers et contiendra les règles applicables dans tous les cas.

Le contenu du fichier `assets/css/style.css` :

{% highlight css %}
@import "reset.css";
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.9.1/font/bootstrap-icons.css");
@import "_header.css";

.sr-only {
display: none;
}

body {
font-family: Arial, sans-serif;
}
{% endhighlight %}

Le fichier `assets/css/reset.css` permet d'annuler les règles d'affichage par défaut des navigateurs. Il est disponible [à cette adresse](http://meyerweb.com/eric/tools/css/reset/).

{% highlight css %}
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.9.1/font/bootstrap-icons.css");
{% endhighlight %}

importe la police d'icônes de Bootstrap dont la documentation est disponible [à cette adresse](https://icons.getbootstrap.com/).

Et enfin nous importons les règles d'affichage de notre header que nous avons définies dans le fichier `assets/css/_header.css`.

Le contenu du fichier `assets/css/_header.css` :

{% highlight css %}
/* Light mode */
@media (prefers-color-scheme: light) {

    body > header {
        background-color: #ffffff;
        color: #000000;
        border-bottom: 1px solid #000000;
    }

    body > header a {
        color:#000000;
    }

    body > header h1 {
        color: #595cff;
    }

    #mobile-header-button {
        color:#000000;
    }
}

/* Dark mode */
@media (prefers-color-scheme: dark) {

    body > header {
        background-color: #343434;
        color:#ffffff;
    }

    body > header a {
        color:#ffffff;
    }

    body > header h1{
        color: #6fffe9;
    }

    #mobile-header-button {
        color: #ffffff;
    }
}

/* Phones */
@media (min-width: 0px) {

    #mobile-header-button {
        background-color: transparent;
        border: none;
        font-size: 1.5rem;
    }

    body > header ul{
        text-align: center;
        display: grid;
        grid-template-rows: 1fr 1fr 1fr;
        font-size: 1.3rem;
    }

    body > header ul li {
        padding-top: 1rem;
        padding-bottom: 1rem;
    }

    #main-nav:not(.open)
    {
        display: none;
    }
}

/* Desktops */
@media (min-width: 992px) {

    #main-nav:not(.open)
    {
        display: block;
    }

    #mobile-header-button {
        display:none;
    }

    body > header h1 {
        font-size: 1.3rem;
        font-weight: bold;
    }

    body > header ul {
        text-align: left;
        display:grid;
        grid-template-columns: 0.2fr 0.1fr 0.1fr;
        grid-template-rows: none;
    }

    body > header ul li{
        padding-top: 0;
        padding-bottom: 0;
    }

    body > header h1:hover {
        text-decoration: underline;
    }
}

/* General */

body > header {
padding:1rem;
}

body > header a {
text-decoration: none;
}

body > header a {
text-decoration: none;
}

body > header a:hover {
text-decoration: underline;
}
{% endhighlight %}

Dans ce fichier nous utilisons 4 media queries différentes pour gérer les tailles d'écrans et le choix de thème.

### Le choix de thème

{% highlight css %}
@media (prefers-color-scheme: light)
{% endhighlight %}

s'appliquera si l'utilisateur a choisi le light mode comme configuration de son navigateur.

{% highlight css %}
@media (prefers-color-scheme: dark)
{% endhighlight %}

s'appliquera si l'utilisateur a choisi le dark mode comme configuration de son navigateur.

### Les tailles d'écran

{% highlight css %}
@media (min-width: 0px)
{% endhighlight %}

s'appliquera par défaut (les écrans de 0px de largeur et plus).

{% highlight css %}
@media (min-width: 992px)
{% endhighlight %}

à partir d'une largeur supérieur à 992px ce sont ces règles qui s'appliqueront.

Les règles qui ne sont pas contenues dans une mediq query s'appliquent dans tous les cas.

### Afficher la navigation

{% highlight css %}
@media (min-width: 0px) {

    #main-nav:not(.open)
    {
        display: none;
    }
}

@media (min-width: 992px) {

    #main-nav:not(.open)
    {
        display: block;
    }
}
{% endhighlight %}

Si elle n'a pas la classe `.open` la navigation ne s'affiche pas sur les petits écrans mais elle s'affiche toujours sur les grands.

{% highlight css %}
@media (min-width: 0px) {

    body > header ul{
        text-align: center;
        display: grid;
        grid-template-rows: 1fr 1fr 1fr;
        font-size: 1.3rem;
    }

}

@media (min-width: 992px) {

    body > header ul {
        text-align: left;
        display:grid;
        grid-template-columns: 0.2fr 0.1fr 0.1fr;
        grid-template-rows: none;
    }

}
{% endhighlight %}

Sur les petits écrans les liens apparaissent verticalement sur 3 lignes. Sur les grands écrans, ils apparaissent horizontalement sur 3 colonnes.

Si vous ajoutez des liens il faudra simplement ajouter des lignes et des colonnes.

## Le Javascript

Dans notre CSS nous avons précisé que sur mobile, la navigation doit avoir la classe `.open` pour être visible.

Le but de notre Javascript sera donc de lui ajouter/retirer la classe `.open` au clic sur le bouton.

Dans notre dossier `assets` créons un dossier `assets/javascript` dans lequel nous ajoutons un fichier `assets/javascript/index.js`.

Le contenu du fichier `assets/javascript/index.js` :

{% highlight javascript %}
window.addEventListener("DOMContentLoaded", (event) => {
var $mobileHeaderBtn = document.getElementById('mobile-header-button');

    $mobileHeaderBtn.addEventListener('click', function(){
        var $mainNav = document.getElementById('main-nav');
        var $classes = $mainNav.classList;

        $classes.toggle("open");
    });
});
{% endhighlight %}

Tout d'abord nous utilisons l'event :

{% highlight javascript %}
window.addEventListener("DOMContentLoaded", (event)
{% endhighlight %}

pour nous assurer que le DOM est bien chargé.

Ensuite nous ajoutons un listener sur le clic du bouton :

{% highlight javascript %}

var $mobileHeaderBtn = document.getElementById('mobile-header-button');

$mobileHeaderBtn.addEventListener('click', function(){
        
});

{% endhighlight %}

Et enfin nous récupérons la liste des classes de la `#main-nav` pour lui ajouter/retirer la classe `.open` :

{% highlight javascript %}

var $mainNav = document.getElementById('main-nav');
var $classes = $mainNav.classList;

$classes.toggle("open");

{% endhighlight %}

## Conclusion

Vous avez maintenant un header responsive en CSS grid, qui gère le dark et light mode et n'utilise que du Javascript natif.

Vous pouvez retrouver le code du header sur [le repo GitHub](https://github.com/Gaellan/SimpleResponsiveHeader).

:nerd_face: Happy coding !