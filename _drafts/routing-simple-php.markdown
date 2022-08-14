---
layout: post
title:  "Routing simple en PHP natif"
date:   2022-08-12 21:22:18 +0200
categories: PHP
tags: Routing
lang: fr
---

Ce tutoriel vous permettra de mettre en place un routing simple en PHP natif.
Ce routing respecte le MVC et est Orienté Objet.

## Pourquoi faire un routing ?

Il est généralement considéré comme une mauvaise pratique de gérer les différentes pages d'un site au moyen des différents fichiers.
En effet, de nombreuses pages ont souvent le même aspect, il vaut donc mieux savoir que telle url doit servir tel template et éviter de dupliquer inutilement du code.

PHP est un language qui permet de dynamiser des pages : profitons-en !

## Tout commence par l'index

Le principe de base du routing en PHP est de rediriger toutes les requêtes vers le fichier `index.php` qui se charge ensuite de traiter les informations et afficher les bons templates.

La version la plus simple de cette idée consiste à utiliser les paramètre `$_GET` pour demander des informations à l'index.
Par exemple `https://mon-site.com/index.php?route=blog&post_id=42` est une façon de demander à l'index d'afficher l'article de blog qui a l'id 42.

Par contre cette URL n'est pas très jolie, ni très ordonnée et n'est pas très SEO friendly. C'est pourquoi nous allons la réécrire.
Elle ressemblera plutôt à ceci : `https://mon-site.com/blog/42`.

### Réécrire des URLs

La réécriture d'URL c'est le travail du serveur. Lorsque votre serveur utilise Apache, cela se passe dans un fichier `.htaccess` que vous placerez à la racine du site avec votre `index.php`.

Le contenu du fichier `.htaccess` :

{% highlight apache %}
<IfModule mod_rewrite.c>
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ /the/path/to/your/index.php?path=$1 [NC,L,QSA]
</IfModule>
{% endhighlight %}

N'oubliez pas de remplacer `/the/path/to/your/` par le chemin vers votre `index.php`.

## Le fichier de configuration des routes

Pour rendre les routes plus simples à définir et modifier nous allons les réunir dans un fichier de configuration.
Nous créons donc un dossier `config` à la racine de notre projet et nous ajoutons un fichier `routes.txt` dans ce dossier.

Le contenu du fichier `config/routes.txt` :

```
/ HomeController:index
/blog BlogController:index
/blog/* BlogController:show
```

Notre projet aura donc 3 routes possibles : 
- La page d'accueil : `/`
- La liste des articles du blog : `/blog`
- Les détails d'un article du blog : `/blog/*`

*Ici le caractère `*` signifie que l'on peut recevoir nímporte quoi après le `/`.*

Chaque route est donc représentée par une ligne avec le format suivant : `path/parameter Controller:method`.
La paramètre du path est optionnel. La route `/blog` ne prend pas de paramètre, mais la route `/blog/42` prend un paramètre (ici `42`).

## Les Controllers

En nous basant sur le fichier `/config/routes.txt` nous pouvons déduire que nous aurons besoin de 2 controllers.
Nous allons les créer dans un nouveau dossier `controllers` à la racine du projet.

Nos controllers seront assez simple, ils contiendront les méthodes indiquées pour les routes qui recevront les paramètres de la route si nécessaire et afficheront le template correspondant.

### HomeController

Le contenu du fichier `controllers/HomeController.php` :

{% highlight php %}
<?php

class HomeController
{
    public function index()
    {
        require "templates/home.phtml";
    }
}
{% endhighlight %}

### BlogController

Le contenu du fichier `controllers/BlogController.php` :

{% highlight php %}
<?php

class BlogController
{
    public function index()
    {
        require "templates/blog_index.phtml";
    }

    public function show(int $id)
    {
        require "templates/blog_show.phtml";
    }
}
{% endhighlight %}

## Les templates

Nos templates vont également être très simples. Ils vont uniquement servir à vérifier que notre routing fonctionne.
Nous allons les créer dans un nouveau dossier `templates` à la racine du projet. 

Nous n'utilisons pas de moteur de templates (comme Twig par exemple), nos fichiers de templates seront donc au format `.phtml`.

### home.phtml

Le contenu du fichier `templates/home.phtml` :

{% highlight html %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Accueil</title>
    </head>
    <body>
        <h1>This is the homepage</h1>
    </body>
</html>
{% endhighlight %}

### blog_index.phtml

Le contenu du fichier `templates/blog_index.phtml` :

{% highlight html %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Blog</title>
    </head>
    <body>
        <h1>This is the blog index</h1>
    </body>
</html>
{% endhighlight %}

### blog_show.phtml

Le contenu du fichier `templates/blog_show.phtml` :

{% highlight html %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Blog <?= $id ?></title>
    </head>
    <body>
        <h1>This is a blog article with id <?= $id ?></h1>
    </body>
</html>
{% endhighlight %}

## Le Router

Notre Router va servir à analyser la requête et appeler le bon controller si la route demandée existe.

Nous allons le créer dans un nouveau dossier `services` à la racine du projet.

Le contenu du fichier `services/Route.php` :

{% highlight php %}
<?php

class Router {

    private function parseRequest(string $request)
    {
        $route = [];

        $routeData = explode("/", $request); // we split the request using the / character

        $route["path"] = "/".$routeData[1]; // the path is what is after the first /

        if(count($routeData) > 2) // if we have more than one /
        {
            $route["parameter"] = $routeData[2]; // the parameter is after the second /
        }
        else
        {
            $route["parameter"] = null; // we don't have a parameter
        }

        return $route;
    }

    public function route(array $routes, string $request)
    {
        $requestData = $this->parseRequest($request); // we analyze the request and sort it

        $routeFound = false;

        foreach($routes as $route) // we go through the list of routes we built in the autoload
        {
            $controller = $route["controller"];
            $method = $route["method"];
            $parameter = $route["parameter"];

            if($route["path"] === $requestData["path"]) // if the path exists
            {
                if($route["parameter"] && $requestData["parameter"] !== null) // if a parameter was needed and we have one
                {
                    $routeFound = true;

                    $ctrl = new $controller();
                    $ctrl->$method($requestData["parameter"]);
                }
                else if(!$route["parameter"] && $requestData["parameter"] === null) // or a parameter was not needed and we don't have one
                {
                    $routeFound = true;

                    $ctrl = new $controller();
                    $ctrl->$method();
                }
            }
        }

        if(!$routeFound) // anything else will throw an exception telling us the route does not exist
        {
            throw new Exception("Route not found", 404);
        }
    }
}
{% endhighlight %}

Nous avons deux méthodes : 

- `private function parseRequest(string $request)` qui sert à transformer la `string $request` en un `array` mieux organisé.
- `public function route(array $routes, string $request)` qui appelle `parseRequest` puis match la requête à son Controller et sa méthode.

Si la route n'existe pas, nous levons une `Exception`.

Le paramètre `array $routes` utilisé dans `Router->route()` aura été défini au chargement de notre site dans l'autoload.

En PHP on peut appeler une classe ou une fonction depuis une `string` qui contient le nom de la classe / fonction : 

{% highlight php %}
// $controller : "HomeController" ou "BlogController"
$ctrl = new $controller();

// $method : "index" ou "show"
$ctrl->$method();
{% endhighlight %}

est donc l'équivalent par exemple de :

{% highlight php %}
$ctrl = new HomeController();
$ctrl->index();
{% endhighlight %}

ou

{% highlight php %}
$ctrl = new BlogController();
$ctrl->index();
{% endhighlight %}


## L'autoload

Notre fichier `autoload.php` que nous plaçons à la racine du projet va nous permettre de faire deux choses :

- faire des `require` des controllers et du routeur
- charger la liste des routes depuis notre fichier de configuration

Le contenu du fichier `autoload.php` : 

{% highlight php %}
<?php

require "./controllers/HomeController.php";
require "./controllers/BlogController.php";
require "./services/Router.php";

$routes = [];

// Read the routes config file
$handle = fopen("config/routes.txt", "r");

if ($handle) { // if the file exists

    while (($line = fgets($handle)) !== false) { // read it line by line

        $route = []; // each route is an array

        $routeData = explode(" ", str_replace(PHP_EOL, '', $line)); // divide the line in two strings (cut at the " ")

        $route["path"] = $routeData[0]; // the path is what was before the " "

        if(substr_count($route["path"], "/") > 1) // check if the path string has more than 1 "/"
        {
            $route["parameter"] = true; // the route expects a parameter
            $pathData = explode("/", $route["path"]); // divide the path in three strings (cut at the "/")
            $route["path"] = "/".$pathData[1]; // isolate the path without the parameters
        }
        else
        {
            $route["parameter"] = false; // the route does not expect a parameter
        }

        $controllerString = $routeData[1]; // the controller string is what was after the " ";

        $controllerData = explode(":", $controllerString); // divide the controller string in two strings (cut at the ":")

        $route["controller"] = $controllerData[0]; // the controller is what was before the ":"

        $route["method"] = $controllerData[1]; // the method is what was after the ":"

        $routes[] = $route; // add the new route to the routes array
    }

    fclose($handle); // close the file
}
{% endhighlight %}

### Ouvrir et lire un fichier

Après avoir initialisé nos controllers et notre router, nous ouvrons le fichier `config/routes.txt` et nous le parcourons ligne par ligne :

{% highlight php %}
$handle = fopen("config/routes.txt", "r"); // open the file
{% endhighlight %}

{% highlight php %}
if ($handle) { // if the file exists

    while (($line = fgets($handle)) !== false) { // read it line by line
{% endhighlight %}

### Trier les données du fichier

Nous voulons trier les données du fichier pour transformer chaque ligne en `array` avec le format suivant :

{% highlight php %}
array $route = [
    "path" => "the path (/ ou /blog ou /blog/*)",
    "parameter" => "(optionnel) si la route attend un paramètre (true ou false)",
    "controller" => "le controller ( HomeController ou BlogController)",
    "method" => "la méthode à appeler dans le controller (index ou show)"
];
{% endhighlight %}

Nous allons donc utiliser les fonctions de la librairie standard de PHP pour découper la `string $line` et la stocker dans le tableau au format qui nous intéresse.
Puis nous allons placer chacune des routes que nous avons assemblé dans un `array $routes` général qui contiendra toutes les routes possibles.

Nous utilisons `str_replace` pour nettoyer notre `string $line` et retirer les sauts à la ligne:

{% highlight php %}
str_replace(PHP_EOL, '', $line)
{% endhighlight %}

Puis nous séparons notre `string $line` en deux `string` de chaque côté du caractère espace : 

{% highlight php %}
explode(" ", $line);
{% endhighlight %}

#### Path et paramètre
Nous vérifions si notre route attend un paramètre en vérifiant le nombre de `/` dans la `string` :

{% highlight php %}
if(substr_count($route["path"], "/") > 1)
{% endhighlight %}

Si oui nous précisons que la route attend un paramètre, et nous retirons le paramètre du path :

{% highlight php %}
$route["parameter"] = true;
$pathData = explode("/", $route["path"]);
$route["path"] = "/".$pathData[1];
{% endhighlight %}

Si non, nous précisons que la route n'attend pas de paramètre :

{% highlight php %}
$route["parameter"] = false;
{% endhighlight %}

#### Controller et méthode

Nous séparons le controller de la méthode en utilisant le caractère `:` :

{% highlight php %}
$controllerData = explode(":", $controllerString);
$route["controller"] = $controllerData[0];
$route["method"] = $controllerData[1];
{% endhighlight %}

Et enfin nous ajoutons notre nouvelle `$route` à la liste : 

{% highlight php %}
$routes[] = $route;
{% endhighlight %}

### Fermer le fichier

Sans oublier de fermer le fichier une fois que nous avons terminé : 

{% highlight php %}
fclose($handle); // close the file
{% endhighlight %}

## Assembler tous les éléments

Pour finir nous allons assembler tout ça depuis notre fichier `index.php` : 

{% highlight php %}
<?php
require "autoload.php";

try {

    $router = new Router();

    if(isset($_GET['path']))
    {
        $request = "/".$_GET['path'];
    }
    else
    {
        $request = "/";
    }

    $router->route($routes, $request);
}
catch(Exception $e)
{
    if($e->getCode() === 404)
    {
        require "./templates/404.phtml";
    }
}
{% endhighlight %}

Ici nous catchons en plus notre `Exception` pour afficher une page 404 si la route n'existe pas.

Le contenu du fichier `templates/404.phtml` :

{% highlight html %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Page not found</title>
    </head>
    <body>
        <h1>404 : page not found</h1>
    </body>
</html>
{% endhighlight %}

## Conclusion

Vous avez maintenant un routeur de base près à l'emploi. Vous pouvez aller plus loin en implémentant une hiérarchie dans vos templates et en ajoutant des managers que vous chargerez dans `autoload.php`.

Vous retrouverez tout le code de ce tuto dans le [repo Github](https://github.com/Gaellan/BasicRoutingPHP).

:nerd_face: Happy Coding !

