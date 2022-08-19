---
layout: post
title:  "Une prévisualisation avec l'API fetch et PHP"
date:   2022-08-19 09:00:00 +0200
categories: JavaScript
tags: fetch
lang: fr
---

Ce tutoriel vous permettra de réaliser une prévisualisation de vos contenus en utilisant CKEditor, l'API fetch de JavaScript et du PHP natif.

Vous pouvez retrouver le code du tutoriel dans [le repo GitHub](https://github.com/Gaellan/PagePreviewer).


Nous réutilisons ici le code du Routeur Simple en PHP (le tuto d'explication est disponible [ici](https://gaellan.github.io/php/2022/08/12/routing-simple-php.html])).

## Le but du tuto

Nous voulons créer un moyen de visualiser en avance le rendu d'un contenu sans avoir besoin de recharger la page ou de rechanger de page.

![yay](/assets/gif/page-previewer.gif)

La résultat final ressemble au Gif au dessus : une editeur de texte un peu avancé avec des styles et niveaux de titre, et lorsque nous cliquons sur prévisualiser, nous voyons le contenu mis en forme à côté.

## Les routes

### Le fichier routes.txt

Le contenu de notre fichier `config/routes.txt` :

{% highlight html%}
/ HomeController:index
/page-preview PageController:preview
{% endhighlight %}

Nous avons donc deux routes : 
- une route `/` qui ne fait rien.
- une route `/page-preview` avec notre outil de prévisualisation

### Le Routeur

Notre routeur a un peu changé par rapport à celui du tutoriel. 
En effet nous allons avoir besoin de gérer les requêtes `$_POST`. Nous ajoutons donc la gestion des $_POST à nos appels de controllers.

Le contenu du fichier `services/Router.php` :

{% highlight php %}
<?php

class Router {
    
    private function parseRequest(string $request)
    {
        $route = []; 
        
        $routeData = explode("/", $request);
        
        $route["path"] = "/".$routeData[1]; 
        
        if(count($routeData) > 2) 
        {
            $route["parameter"] = $routeData[2];
        }
        else
        {
            $route["parameter"] = null;
        }
        
        return $route;
    }
    
    public function route(array $routes, string $request)
    {
        $requestData = $this->parseRequest($request);
        
        $routeFound = false;
        
        foreach($routes as $route)
        {
            $controller = $route["controller"];
            $method = $route["method"];
            $parameter = $route["parameter"];
            
            if($route["path"] === $requestData["path"])
            {
                if($route["parameter"] && $requestData["parameter"] !== null)
                {
                    $routeFound = true;
                    
                    $ctrl = new $controller();
                    $ctrl->$method($_POST, $requestData["parameter"]);
                }
                else if(!$route["parameter"] && $requestData["parameter"] === null)
                {
                    $routeFound = true;
                    
                    $ctrl = new $controller();
                    $ctrl->$method($_POST);
                }
            }
        }
        
        if(!$routeFound)
        {
            throw new Exception("Route not found", 404);
        }
    }
}

?>
{% endhighlight %}

## Les controllers

Comme nous pouvons le déduire de notre fichier `config/routes.txt` nous avons deux controllers:

- `HomeController` qui ne fait rien d'autre qu'afficher la page d'accueil
- `PageController` dont la méthode `preview` va nous afficher notre outil

### HomeController

Le code de `HomeController` n'a pas changé depuis le tuto :

{% highlight php %}
<?php

class HomeController
{
    public function index(array $post)
    {
        require "templates/home.phtml";
    }
}
{% endhighlight %}

### PageController

Le code du PageController `controllers/PageController.php`:

{% highlight php %}
<?php

class PageController extends AbstractController
{
    public function preview(array $post)
    {
        if(isset($_POST["data"])) // the form has been submitted
        {
            $title = $_POST["title"];
            $content = $_POST["content"];
            $view = $this->renderPartial("_preview", [
                "title" => $title,
                "content" => $content
            ]);
        }
        else
        {
            $this->render("page_preview", [
            
            ]);
        }
    }
}

{% endhighlight %}

Comme vous pouvez le constater, `PageController` hérite d'une autre classe (mot clé `extends`) : `AbstractController`.

#### AbstractController

Le code de l'AbstractController `controllers/AbstractController.php` :
{% highlight php %}
<?php

abstract class AbstractController
{
    protected function renderPartial(string $template, array $values)
    {
        $data = $values;
        
        require "templates/".$template.".phtml";
    }
    
    protected function render(string $template, array $values)
    {
        $data = $values;
        $page = $template;
        
        require "templates/layout.phtml";
    }
}
{% endhighlight %}

L'`AbstractController` est une sorte de modèle de controller. Il contient deux méthodes :

- `renderPartial` qui affiche un bout précis d'un template mais sans remettre tout le layout.
- `render` qui afiche une page complète avec `<!-- DOCTYPE>`, `<head>` et autres.


Retour à notre `PageController`. Il a deux rôles :

#### Afficher la page

{% highlight php %}
else
{
    $this->render("page_preview", [
            
    ]);
}
{% endhighlight %}

#### Recevoir le formulaire et générer le HTML pour la visualisation

{% highlight php %}
if(isset($_POST["data"])) // the form has been submitted
{
    $title = $_POST["title"];
    $content = $_POST["content"];
    $view = $this->renderPartial("_preview", [
        "title" => $title,
        "content" => $content
    ]);
}
{% endhighlight %}

## Les templates

Les templates de la page 404 et de la home sont les mêmes que ceux du tuto du Routeur.
Par contre nous en avons de nouveaux :

### layout.phtml

Le contenu du fichier `templates/layout.phtml` :

{% highlight php %}
<!DOCTYPE html>
<html lang="fr">
    <head>
        <meta charset="utf-8" />
        <title>Page Previewer</title>
        <link rel="stylesheet" href="assets/style.css" />
    </head>
    <body>
        <h1>Page Previewer</h1>
        <?php require "templates/".$page.".phtml"; ?>

        <script src="https://cdn.ckeditor.com/ckeditor5/35.0.1/classic/ckeditor.js"></script>
        <script type="application/javascript" src="assets/index.js"></script>
    </body>
</html>
{% endhighlight %}

Il charge notre style, le javascript de CKEditor et notre propre JavaScript.

### page_preview.phtml

Le contenu du fichier `templates/page_preview.phtml` :

{% highlight php %}
<main>
    <section>
        <h2>Le formulaire</h2>
        <form id="editPageForm" method="POST" action="/page-previewer/page-preview">
            <fieldset>
                <label for="title">Titre</label>
                <input type="text" name="title" id="title" />
            </fieldset>
            <fieldset>
                <label for="content">Contenu</label>
                <textarea name="content" id="content">

                </textarea>
            </fieldset>
            <fieldset>
                <input id="previewSubmit" type="submit" value="Prévisualiser" />
            </fieldset>
        </form>
    </section>
    <section>
        <h2>L'aperçu</h2>
        <section id="previewContent">
            
        </section>
    </section>
</main>
{% endhighlight %}

Nous avons d'un côté notre formulaire de saisie : 

{% highlight php %}

<section>
    <h2>Le formulaire</h2>
    <form id="editPageForm" method="POST" action="/page-previewer/page-preview">
        <fieldset>
            <label for="title">Titre</label>
            <input type="text" name="title" id="title" />
        </fieldset>
        <fieldset>
            <label for="content">Contenu</label>
            <textarea name="content" id="content">
            
            </textarea>
        </fieldset>
        <fieldset>
            <input id="previewSubmit" type="submit" value="Prévisualiser" />
        </fieldset>
    </form>
</section>

{% endhighlight %}

et de l'autre la zone dans laquelle nous allons prévisualiser :

{% highlight php %}

<section>
    <h2>L'aperçu</h2>
    <section id="previewContent">

    </section>
</section>

{% endhighlight %}

### _preview.phtml

Le contenu de `templates/_preview.phtml` qui génèrera le HTML de notre prévisualisation :

{% highlight php %}

<article>
    <header>
        <h1>
            <?= $data["title"]; ?>
        </h1>
    </header>
    <main>
        <?= $data["content"]; ?>
    </main>
</article>

{% endhighlight %}

## Le CSS

Ici du CSS assez simpliste, il nous sert juste à créer deux espaces côte à côte pour le formulaire et la prévisualisation.
Il nous sert également à mettre du style différent dans l'espace de prévisualisation pour bien voir la différence.

Le contenu de `assets/style.css` :

{% highlight css %}

body {
width:100vw;
height:100vh;
}

body > main {
display:grid;
height:100%;
grid-template-rows:1fr;
grid-template-columns:1fr 1fr;
}

.ck-editor__editable[role="textbox"] {
/* editing area */
min-height: 50vh;
}

#previewContent {
font-family: Arial, sans-serif;
display:flex;
}

#previewContent h1 {
font-size: 1.8rem;
}

#previewContent h2 {
font-size: 1.6rem;
}

#previewContent h3 {
font-size:1.2rem;
}

#previewContent a {
color: red;
}

#previewContent a:hover {
color:blue;
}

{% endhighlight %}

{% highlight css %}
.ck-editor__editable[role="textbox"] {
/* editing area */
min-height: 50vh;
}
{% endhighlight %}

Nous permet de nous assurer que nous aurons assez de place pour écrire dans l'editeur.

## Le JavaScript

Le contenu du fichier `assets/index.js` : 

{% highlight javascript %}

window.addEventListener("DOMContentLoaded", (event) => {
    ClassicEditor
        .create( document.querySelector( '#content' ) )
        .then( editor => {
            contentEditor = editor;
        } )
        .catch( error => {
        } );
        
       $submit = document.getElementById("previewSubmit");
       
       $submit.addEventListener('click', function(event){
           event.preventDefault();
           title = document.getElementById("title").value;
           
           formData = new FormData();
           formData.append('data', true);
           formData.append('title', title);
           formData.append('content', contentEditor.getData());
           
           const options = {
                method: 'POST',
                body: formData
            };
            
            fetch('/page-previewer/page-preview', options)
                .then(response => response.text())
                .then(data => {
                    preview = document.getElementById("previewContent");
                    preview.innerHTML = data;
                });
       });
});

{% endhighlight %}

Commençons par le début : nous mettons notre code dans

{% highlight javascript %}
window.addEventListener("DOMContentLoaded", (event) => {

}
{% endhighlight %}

pour nous assurer que tout le DOM est bien chargé avant de commencer à le manipuler.



{% highlight javascript %}

ClassicEditor
    .create( document.querySelector( '#content' ) )
    .then( editor => {
    contentEditor = editor;
    } )
    .catch( error => {

} );

{% endhighlight %}

Nous sert à initialiser l'éditeur de texte en le chargeant depuis nos textarea qui à l'id `content`.

{% highlight javascript %}

$submit = document.getElementById("previewSubmit");

{% endhighlight %}

Nous récupérons notre bouton de soumission de formulaire.

{% highlight javascript %}

$submit.addEventListener('click', function(event){

event.preventDefault();

});

{% endhighlight %}

et nous écoutons chaque clic sur le bouton pour intercepter la soumission du formulaire.

{% highlight javascript %}
title = document.getElementById("title").value;

formData = new FormData();
formData.append('data', true);
formData.append('title', title);
formData.append('content', contentEditor.getData());

{% endhighlight %}

Nous récupérons les valeurs des champs du formulaire et nous nous en servons pour créer un objet FormData qui nous permet d'envoyer les champs du formulaire à PHP.

{% highlight javascript %}

const options = {
method: 'POST',
body: formData
};

fetch('/page-previewer/page-preview', options)
    .then(response => response.text())
    .then(data => {
        preview = document.getElementById("previewContent");
        preview.innerHTML = data;
});

{% endhighlight %}

Nous utilisons l'API fetch pour envoyer notre requête "POST" :

{% highlight javascript %}

const options = {
method: 'POST',
body: formData
};

fetch('/page-previewer/page-preview', options)

{% endhighlight %}

Puis nous récupérons la réponse de PHP (il nous renvoie du HTML c'est donc du texte) :

{% highlight javascript %}

.then(response => response.text())

{% endhighlight %}

Nous injectons enfin le HTML récupéré dans notre section de prévisualisation `previewContent` :

{% highlight javascript %}

.then(data => {
preview = document.getElementById("previewContent");
preview.innerHTML = data;
}

{% endhighlight %}

N'oubliez pas de modifier la route qu'appelle fetch en fonction de la configuration de votre serveur !

:nerd_face: Happy Coding !