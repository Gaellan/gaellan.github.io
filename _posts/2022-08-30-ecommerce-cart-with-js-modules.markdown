---
layout: post
title:  "Un panier e-commerce utilisant les modules Javascript"
date:   2022-08-30 09:00:00 +0200
categories: javascript
tags: modules
lang: fr
---

Ce tutoriel vous permettra de créer un panier pour un site e-commerce en utilisant les modules JavaScript.

![yay](/assets/gif/cart.gif)

Tout le code du tutoriel est disponible sur [Le repo Github](https://github.com/Gaellan/ECommerceCart).

## Mettre en place le HTML

le contenu de notre fichier `index.html`:

{% highlight html %}

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Page démo panier</title>
    <link rel="stylesheet" href="style.css" />
</head>
    <body class="cart-open">
        <main>

        </main>
        <aside>
            <header>
                <h2>Mon panier</h2>
                <p id="cart-total-price">Total : 59 $</p>
            </header>
            <main>
                <ul>

                </ul>
            </main>
            <footer>
                <a class="btn btn-cart-back" href="">Continuer mes achats</a>
                <a class="btn btn-cart-order" href="">Finaliser la commande</a>
            </footer>
        </aside>
        <script type="module" src="assets/javascript/main.js"></script>
    </body>
</html>

{% endhighlight %}

La seule chose inhabituelle c'est la façon d'appeler notre JavaScript. On a l'habitude de faire quelque chose comme : 

{% highlight html %}

<script type="application/javascript" src="assets/javascript/main.js"></script>

{% endhighlight %}

Mais là nous allons utiliser les modules ce qui change un peu notre syntaxe d'inclusion.

### Un web server

Pour tester en local des modules javascript il faut un serveur web. Si vous n'en avez pas à disposition voici une solution pour Google Chrome :

- Chrome : [Application Chrome Web Server](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb).


## Le Style

Pour cet exercice le SASS/CSS est purement décoratif.

## Le JavaScript

### main.js

Le point d'entrée de nos scripts est celui que l'on a chargé directement dans le HTML : `main.js`.

Voici donc le contenu du fichier `assets/javascript/main.js` :

{% highlight javascript %}

import { initCart } from "./cart/cart.js";

window.addEventListener("DOMContentLoaded", (event) => {
initCart();
});

{% endhighlight %}

Il n'a l'air de rien comme ça mais c'est la magie des modules : on isole les comportements.

Nous commençons par importer une fonction initCart depuis notre module `cart.js`.

{% highlight javascript %}

import { initCart } from "./cart/cart.js";

{% endhighlight %}

Nous appelons ensuite cette fonction dès que le DOM a fini de se charger.

{% highlight javascript %}

window.addEventListener("DOMContentLoaded", (event) => {
initCart();
});

{% endhighlight %}

### cart.js

`cart.js` est un plus gros morceau puisqu'il contient tout notre logique, nous allons donc progresser étape par étape.

Tout d'abord, `initCart()` qui était appellée dans notre `main.js` doit être déclarée et ensuite être exportée (pour que nous puissions l'importer dans `main.js`).

{% highlight javascript %}

// init the cart with either data from session storage or the demo data
function initCart() {

    if(getCart() === null) // if cart session storage doesn't exist create it
    {
        saveCart(fakeCart);
    }

    renderCart();
}

export { initCart };

{% endhighlight %}

`initCart()` fait donc appel à 4 choses différentes :

- `getCart()` va nous permettre de récupérer l'état actuel du panier
- `saveCart()` va nous permettre de sauvegarder l'état actuel du panier
- `fakeCart` sont des données de démonstration pour notre panier
- `renderCart()` nous permet d'afficher notre panier en l'injectant dans notre HTML

#### getCart()

La fonction `getCart()` va nous permettre de récupérer l'état actuel du panier que nous stockons dans le `sessionStorage` :

{% highlight javascript %}

// retrieve the cart from session storage
function getCart()
{

    return JSON.parse(sessionStorage.getItem("cart"));

}

{% endhighlight %}

Nous allons chercher dans le `sessionStorage` notre panier. Pour économiser le storage nous transformons notre avons transformé notre objet en string JSON avant de le stocker.
Nous devons donc utiliser `JSON.parse` pour le retransformer en objet.

#### saveCart()

La fonction `saveCart()` va nous permettre de sauvegarder dans le `sessionStorage` l'état actuel de notre panier.

{% highlight javascript %}

// update the cart in session storage
function saveCart($cart)
{
    sessionStorage.setItem("cart", JSON.stringify($cart));
}

{% endhighlight %}

Nous sauvegardons donc le `$cart` que nous avons reçu sous forme de string JSON (`JSON.stringify`) dans le `sessionStorage`.

#### fakeCart

`fakeCart` est un objet représentant notre panier et son contenu que nous utilisons pour faire notre démonstration. 
Dans un vrai site il serait relié, via des appels à `fetch` au panier réel de notre client-e.

{% highlight javascript %}

let fakeCart = {
    totalPrice : 59,
    itemCount : 3,
    items : [
        {
        id : 12,
        imageUrl : "https://picsum.photos/id/1025/50/50",
        price : 23,
        amount: 1,
        },
        {
        id : 23,
        imageUrl : "https://picsum.photos/id/823/50/50",
        price : 17,
        amount: 1,
        },
        {
        id : 42,
        imageUrl : "https://picsum.photos/id/742/50/50",
        price : 19,
        amount: 1,
        },
    ]
} // this cart is for demonstration purposes only, 
// it should be replaced by the real thing

{% endhighlight %}

#### renderCart()

`renderCart()` c'est la fonction qui va nous permettre d'afficher notre panier et ses produits. 
Elle va remplir notre HTML avec le HTML qui correspond aux produits du panier.

{% highlight javascript %}

// display the cart
function renderCart() {

    // retrieve the cart
    var $cart = getCart();

    // remove the ul
    var $productList = document.querySelector("body > aside > main");
    var $ulToRemove = document.querySelector("body > aside > main > ul");
    $productList.removeChild($ulToRemove);

    // create the new ul
    var $newUl = document.createElement("ul");

    // create the lis
    for(var i = 0; i < $cart.items.length; i++)
    {
        if($cart.items[i].amount > 0)
        {
            var $item = $cart.items[i];
            var $li = document.createElement("li");
            $li.appendChild(createCartItem($item)); // append them
            $newUl.appendChild($li);
        }
    }

    $productList.appendChild($newUl);

    // update cart total price
    let $totalPrice = document.getElementById("cart-total-price");
    $totalPrice.innerText = "Total : " + $cart.totalPrice + " $";

    loadListeners();
}

{% endhighlight %}

Tout d'abord nous récupérons la version la plus à jour de notre panier pour être sûr-e-s d'afficher le bon :

{% highlight javascript %}

// retrieve the cart
var $cart = getCart();

{% endhighlight %}

Petit rappel de notre HTML notre liste de produits va aller se placer dans le `<main>` de l'`<aside>`:

{% highlight html %}
<main>
    <ul>

    </ul>
</main>
{% endhighlight %}

Il faut savoir que quand on veut ajouter des éléments HTML avec du JavaScript il faut les ajouter un par un. 
Pour éviter de devoir ajouter tous nos `<li>` un par un, nous allons retirer le `<ul>` déjà présent pour le remplacer par le notre que nous aurons pré-rempli.

{% highlight javascript %}

// remove the ul
var $productList = document.querySelector("body > aside > main");
var $ulToRemove = document.querySelector("body > aside > main > ul");
$productList.removeChild($ulToRemove);

// create the new ul
var $newUl = document.createElement("ul");

{% endhighlight %}

Nous allons ensuite créer une liste de `<li>` contenant les bonnes infos sur nos produits ainsi que nos boutons d'action et l'ajouter à notre `<ul>`.
Nous n'ajoutons un `<li>` dans notre HTML que si la quantité du produit dans le panier est supérieure à 0.

{% highlight javascript %}

    // create the lis
    for(var i = 0; i < $cart.items.length; i++)
    {
        if($cart.items[i].amount > 0)
        {
            var $item = $cart.items[i];
            var $li = document.createElement("li");
            $li.appendChild(createCartItem($item)); // append them
            $newUl.appendChild($li);
        }
    }

    $productList.appendChild($newUl);

{% endhighlight %}

Comme nous avons pas mal de manipulations du DOM à faire pour créer les articles dans notre panier, nous isolons la création dans une fonction : `createCartItem()`.

##### createCartItem()

{% highlight javascript %}

// create one cart item to be injected in the html
function createCartItem($item)
{
    var $containerSection = document.createElement("section");

    // creating the figure and img
    var $figure = document.createElement("figure");
    var $img = document.createElement("img");
    $img.setAttribute("alt", "image du produit " + $item.id);
    $img.setAttribute("src", $item.imageUrl);
    $figure.appendChild($img);

    $containerSection.appendChild($figure);

    // creating the product info
    let $productInfo = document.createElement("section");
    $productInfo.classList.add("cart-product-info");
    let $productName = document.createElement("h3");
    let $productNameContent = document.createTextNode("Nom du produit " + $item.id);
    $productName.appendChild($productNameContent);
    $productInfo.appendChild($productName);

    $containerSection.appendChild($productInfo);

    // creating the product actions
    let $productActions = document.createElement("section");
    $productActions.classList.add("cart-product-actions");

    let $buttonsSection = document.createElement("section");

    let $removeButton = document.createElement("button");
    $removeButton.setAttribute("data-product-id", $item.id);
    $removeButton.classList.add("cart-btn");
    $removeButton.classList.add("cart-button-remove");
    let $minus = document.createTextNode("-");
    $removeButton.appendChild($minus);

    let $amountSpan = document.createElement("span");
    let $amountContent = document.createTextNode($item.amount);
    $amountSpan.appendChild($amountContent);

    let $addButton = document.createElement("button");
    $addButton.setAttribute("data-product-id", $item.id);
    $addButton.classList.add("cart-btn");
    $addButton.classList.add("cart-button-add");
    let $plus = document.createTextNode("+");
    $addButton.appendChild($plus);

    $buttonsSection.appendChild($removeButton);
    $buttonsSection.appendChild($amountSpan);
    $buttonsSection.appendChild($addButton);

    $productActions.appendChild($buttonsSection);

    let $productPrice = document.createElement("p");
    $productPrice.setAttribute("data-product-id", $item.id);
    $productPrice.classList.add("cart-product-price");

    let $productPriceSpan = document.createElement("span");
    let $productPriceSpanContent = document.createTextNode("" + $item.amount * $item.price);
    $productPriceSpan.appendChild($productPriceSpanContent);

    let $currencyContent = document.createTextNode("$");

    $productPrice.appendChild($productPriceSpan);
    $productPrice.appendChild($currencyContent);

    $productActions.appendChild($productPrice);

    $containerSection.appendChild($productActions);

    return $containerSection;
}

{% endhighlight %}

Comme vous pouvez le voir la fonction est longue mais en soi elle n'est pas compliquée juste répétitive.

Nous créons d'abord la `<section>` qui contiendra tout le monde puis la `<figure>` et l'`<img>` de notre produit puis nous assemblons le tout.

{% highlight javascript %}

    var $containerSection = document.createElement("section");

    // creating the figure and img
    var $figure = document.createElement("figure");
    var $img = document.createElement("img");
    $img.setAttribute("alt", "image du produit " + $item.id);
    $img.setAttribute("src", $item.imageUrl);
    $figure.appendChild($img);

    $containerSection.appendChild($figure);

{% endhighlight %}

Nous créons ensuite la `<section>` qui contiendra les informations du produit et nous l'ajoutons à la suite du reste :

{% highlight javascript %}

// creating the product info
let $productInfo = document.createElement("section");
$productInfo.classList.add("cart-product-info");
let $productName = document.createElement("h3");
let $productNameContent = document.createTextNode("Nom du produit " + $item.id);
$productName.appendChild($productNameContent);
$productInfo.appendChild($productName);

$containerSection.appendChild($productInfo);

{% endhighlight %}

Ensuite nous construisons bloc par bloc la `<section>` qui contient les boutons pour changer la quantité et l'affichage du prix du produit et nous lájoutons à la suite :

{% highlight javascript %}

    // creating the product actions
    let $productActions = document.createElement("section");
    $productActions.classList.add("cart-product-actions");

    let $buttonsSection = document.createElement("section");

    let $removeButton = document.createElement("button");
    $removeButton.setAttribute("data-product-id", $item.id);
    $removeButton.classList.add("cart-btn");
    $removeButton.classList.add("cart-button-remove");
    let $minus = document.createTextNode("-");
    $removeButton.appendChild($minus);

    let $amountSpan = document.createElement("span");
    let $amountContent = document.createTextNode($item.amount);
    $amountSpan.appendChild($amountContent);

    let $addButton = document.createElement("button");
    $addButton.setAttribute("data-product-id", $item.id);
    $addButton.classList.add("cart-btn");
    $addButton.classList.add("cart-button-add");
    let $plus = document.createTextNode("+");
    $addButton.appendChild($plus);

    $buttonsSection.appendChild($removeButton);
    $buttonsSection.appendChild($amountSpan);
    $buttonsSection.appendChild($addButton);

    $productActions.appendChild($buttonsSection);

    let $productPrice = document.createElement("p");
    $productPrice.setAttribute("data-product-id", $item.id);
    $productPrice.classList.add("cart-product-price");

    let $productPriceSpan = document.createElement("span");
    let $productPriceSpanContent = document.createTextNode("" + $item.amount * $item.price);
    $productPriceSpan.appendChild($productPriceSpanContent);

    let $currencyContent = document.createTextNode("$");

    $productPrice.appendChild($productPriceSpan);
    $productPrice.appendChild($currencyContent);

    $productActions.appendChild($productPrice);

    $containerSection.appendChild($productActions);

{% endhighlight %}

Et enfin nous renvoyons le tout pour qu'il soit inséré dans le `<li>` :

{% highlight javascript %}

return $containerSection;

{% endhighlight %}


Retour à notre fonction `renderCart()`, qui doit finir son travail.

Elle affiche le prix total du panier dans le header du panier. Puis elle appelle une fonction `loadListeners()` maintenant que notre nouveau DOM est en place.

{% highlight javascript %}

// update cart total price
let $totalPrice = document.getElementById("cart-total-price");
$totalPrice.innerText = "Total : " + $cart.totalPrice + " $";

loadListeners();

{% endhighlight %}

#### loadListeners()

`loadListeners()` est la fonction qui va nous permettre d'écouter les clics sur tout nos boutons d'ajout ou retrait de quantité d'articles dans le panier.

{% highlight javascript %}

// load the listeners for the add and remove buttons
function loadListeners()
{
    let $addButtons = document.getElementsByClassName("cart-button-add");
    let $removeButtons = document.getElementsByClassName("cart-button-remove");

    for(var i = 0; i < $addButtons.length; i++)
    {
        $addButtons[i].addEventListener("click", addItem);
        $removeButtons[i].addEventListener("click", removeItem);
    }
}

{% endhighlight %}

En utilisant leur attribut `class`, nous récupérons tous nos boutons :

{% highlight javascript %}

let $addButtons = document.getElementsByClassName("cart-button-add");
let $removeButtons = document.getElementsByClassName("cart-button-remove");

{% endhighlight %}

Nous parcourons la liste et attachons à chaque bouton d'ajout la fonction `addItem` et à chaque bouton de retrait la fonction `removeItem` :

{% highlight javascript %}

for(var i = 0; i < $addButtons.length; i++)
{
    $addButtons[i].addEventListener("click", addItem);
    $removeButtons[i].addEventListener("click", removeItem);
}

{% endhighlight %}

#### addItem()

`addItem()` est la fonction qui permet d'augmenter de 1 la quantité d'un produit dans le panier : 

{% highlight javascript %}

// update the item amount to + 1
function addItem(event)
{
    let $id = event.target.getAttribute("data-product-id");
    let $itemKey = findItem($id);
    let $cart = getCart();

    if($itemKey !== null)
    {
        $cart.items[$itemKey].amount += 1;
        saveCart($cart);
        computeCartTotal();
        renderCart();
    }
}

{% endhighlight %}

Nous récupérons l'id du produit concerné en utilisant l'attribut `data-product-id` de notre bouton.

Comment savons nous quel bouton a été cliqué ? En utilisant l'attribut `event.target` de l'évenement que nous envoie notre listener.

Nous recherchons ensuite dans les produits de notre panier l'index auquel se trouve celui que nous voulons modifier en utilisant une fonction `findItem()` que nous avons créée.

{% highlight javascript %}

// get the item key in the cart.items array
function findItem($id)
{
    let $cart = getCart();

    for(var i = 0; i < $cart.items.length; i++)
    {
        if($cart.items[i].id === parseInt($id))
        {
            return i;
        }
    }

    return null;
}

{% endhighlight %}

`findItem()` parcourt notre liste de produit et nous renvoie l'index du produit que nous cherchons si elle le trouve sinon elle renvoie `null`.

Attention : nous avons récupéré l'id depuis le DOM c'est donc une `string` mais dans notre objet c'est un nombre, pour éviter les problèmes lors de la comparaison nous transformons l'id du DOM en nombre (`parseInt()`).

Nous vérifions que `findItem` a bien trouvé le produit (en vérifiant que son retour n'est pas `null`) puis nous mettons à jour la quantité et enfin nous suavegardons notre panier.
Nous allons ensuite utiliser la fonction `computeCartTotal()` pour mettre à jour le prix total du panier et appeler `renderCart()` pour afficher notre panier modifié.

{% highlight javascript %}

if($itemKey !== null)
{
    $cart.items[$itemKey].amount += 1;
    saveCart($cart);
    computeCartTotal();
    renderCart();
}

{% endhighlight %}


#### computeCartTotal()

`computeCartTotal` met le prix total du panier à 0, puis parcourt la liste des produits puis ajoute pour chaque produit `quantité x prix` au prix total puis sauvegarde le panier.

{% highlight javascript %}

// update the total price of the cart
function computeCartTotal()
{
    let $cart = getCart();
    let $price = 0;

    for(var i = 0; i < $cart.items.length; i++)
    {
        $price += ($cart.items[i].price * $cart.items[i].amount);
    }

    $cart.totalPrice = $price;
    saveCart($cart);
}

{% endhighlight %}

#### removeItem()

`removeItem` fonctionne exactement comme `addItem` sauf que l'on fait - 1 au lieu de + 1 :

{% highlight javascript %}

// update the item amount to - 1
function removeItem(event)
{
    let $id = event.target.getAttribute("data-product-id");
    let $itemKey = findItem($id);
    let $cart = getCart();

    if($itemKey !== null)
    {
        $cart.items[$itemKey].amount -= 1;
        saveCart($cart);
        computeCartTotal();
        renderCart();
    }

}

{% endhighlight %}


Vous pourrez ensuite relier ce panier au vrai système backend de votre site (par exemple en utilisant fetch et les $_SESSION en PHP) !

:nerd_face: Happy Coding !