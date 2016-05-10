---
layout: post
title: "Select2 : Ajax et ajout de données"
fullview: false
comments: true
categories: [tech]
tags: [jquery, select2, javascript]
---

Cet article porte sur l'utilisation de la librairie JQuery dénommée [Select2](https://select2.github.io/). Elle permet de personnaliser les `<select>` afin d'avoir des systèmes de tag, de recherche, d'utilisation de données distantes, etc. Durant l'utilisation de Select2 (version 4.0.0) j'ai rencontré quelques difficultés pour ajouter une nouvelle valeur parmi mes données distantes. Je vais donc vous présenter la solution que j'ai trouvé.

Cet article se base sur la [réponse de Wooki](http://stackoverflow.com/a/30847908/5724830) provenant de Stackoverflow.

# Chargement de données distantes
Avant toute chose, il s'agit de comprendre comment remonter des données distantes et les traiter. La [documentation](https://select2.github.io/examples.html#data-ajax) donne un exemple simple. On va reprendre le morceau de code et l'adapter à notre contexte. On souhaite récupérer la liste des dépôts GitHub en utilisant leur API. La requête sera sous la forme `https://api.github.com/search/repositories?q=bootstrap`. La réponse renverra notamment des `items`.

La documentation de Select2 indique qu'il faut regarder le code source de la page pour avoir les fonctions omises. Je les ajoute personnellement dans l'exemple pour la compréhension. Je retire également quelques éléments qui ne me serviront pas.

```javascript
$('.select2-simple').select2({
  placeholder: "Entrez le nom d'un repo GitHub",
  minimumInputLength: 1,
  ajax: {
    url: "https://api.github.com/search/repositories",
    dataType: 'json',
    delay: 250,
    method: 'GET',
    data: function (params) {
      return {
        q: params.term, // les termes de la requête
      };
    },
    processResults: function (data, params) {
      // le résultat est formaté via la librairie pour un
      // affichage correct côté utilisateur.
      return {
        results: data.items,
      };
    },
  },
  templateResult: function(data) {
    // Le template de présentation du résultat de la
    // recherche. Il s'agit de la liste des éléments
    // correspondant à la recherche. Ici on veut le titre des posts.
    if (data.loading) return data.text;

    return data.full_name;
  },
  templateSelection: function(data) {
    // Affichage de l'item sélectionné.
    // Sinon affichage du placeholder (data.text).
    if (data.full_name) return data.full_name + '(' + data.git_url + ')';

    return data.text;
  },
});
```
J'ai ajouté quelques commentaires pour la compréhension mais globalement voici ce que fait ce bout de code :

  1. Paramétrer la requête AJAX JQuery (URl, délai, type de données, .).
  2. Récupérer les termes de la recherche provenant de l'élément select2.
  3. Mise en forme du résultat avec `processResults` qui est une fonction propre à la librairie.
  4. Formatage du résultat avec `templateResult` et de la sélection avec `templateSelection`.

# Ajouter des valeurs de façon manuelle
Vous venez de voir comment récupérer des données distantes avec select2 et les afficher. Désormais on souhaite pouvoir ajouter une valeur qui n'existe pas dans la source des données récupérées.

L'idée est donc de réussir à traiter la nouvelle information au sein d'un ensemble préexistant.

## Profiter de la fonctionnalité des tags
Afin de pouvoir ajouter de nouvelles données, on va utiliser le système de tags de Select2. Pour réaliser cela il faut tout d'abord paramétrer la propriété indiquant l'utilisation des tags : `tags: true`.

Ensuite on va profiter de la fonction `createTag` pour retourner un nouvel objet décrivant la nouvelle donnée avec trois propriétés. Ici `newOption` permettra de déterminer s'il s'agit d'une nouvelle entrée parmi l'ensemble existant et `text` correspond aux termes saisis dans le select.

```
createTag: function (params) {
  return {
    id: params.term,
    text: params.term,
    newOption: true
  }
}
```

Pour terminer, il faut gérer l'affichage. Ici on fait quelque chose de très simple : on ajoute dans la fonction `templateResult` une condition avec un peu de formatage.

```
if (data.newOption) return data.text += "(NEW)";
```

Vous venez de voir un exemple simple vous permettant d'ajouter une nouvelle information à votre ensemble de données distantes, libre à vous ensuite de traiter comme vous le souhaitez cette information. Le code peut-être consulté et testé sur [JSFiddle](https://jsfiddle.net/theredfish/oeLcdhjo/3).
