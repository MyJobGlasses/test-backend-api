# test-backend-api

# Exercice

Ton objectif est de monter une API qui se base sur une API publique qui enrichit/filtre ses données

## 0. Démarre un boilerplate en ruby pour construire une API

L'exercice que tu vas être amené à faire te demandera de construire un serveur, et de savoir faire les choses suivantes:
- Démarrer une application Rails en mode API ferait l'affaire.
  - Tu peux utiliser ces flags : `--skip-sprockets --skip-action-cable --skip-active-record --skip-action-mailer --skip-yarn --skip-javascript --skip-turbolinks --api`
- Faire des requêtes sur des URLs publiques (il n'y aura pas d'authentification ni de bidouille particulière à faire, juste savoir utiliser un client HTTP)
- Pas besoin de base de données, de mails, d'asset pipelining, de gérer des vues, ni de quoi que ce soit qui nécessite de l'infra particulière.
- Documenter
  - Au niveau du code, au moins les arguments + retours des méthodes publiques des services (par exemple, si tu es en Ruby, tu peux t'aligner sur Yard ou ruby-doc)
  - Au niveau du Readme,
    - Comment lancer le serveur (en Rails ce serait juste `rails server development`)
    - Comment lancer une console intéractive qui charge ton code (en Rails ce serait juste `rails console development`)
- Utiliser des noms de variables appropriés
- Nous n'attendons pas des tests, mais rien ne t'empêche de fonctionner en TDD si ça te semble plus rapide.
- Nous n'attendons pas de gestion particulière des erreurs (IO, etc) qui n'est pas demandée explicitement dans l'énoncé
- Il n'y aura pas de piège dans l'exercice et a priori pas de cas de bord (données pas propres, etc). Si jamais tu tombes sur un corner case tu pourras nous le signaler, mais si ça marche ailleurs (par exemple en prenant une donnée différente) alors c'est all good.

**Consignes supplémentaires révélées avec réception de l'exercice**

- On veut que tout ce que tu rendras disponible via l'API soit aussi facilement disponible via la console intéractive. Le scénario utilisateur : En tant que dev, je veux pouvoir facilement reproduire ce que voit un utilisateur de l'API sans environnement HTTP
  - par exemple, en Rails la console charge par défaut tout le code du dossier "/app" et on peut directement accéder aux classes
- Décris brièvement comment on peut "lancer" chaque exercice via la dite console

## 1. Implémente un service qui récupère des personnages par IDs

- Ecrit un service qui va chercher une liste de personnages dans https://rickandmortyapi.com/
- Travaille avec leur API REST ou graphQL
- Le service doit prendre en entrée un ID ou un tableau d'ID : 2 modes de fonctionnement du service (qui vont te servir par la suite)
  - (1) Soit on lui passe un ID de perso => il récupère juste ce perso
  - (2) Soit on lui passe une liste d'IDs de persos => Il recupère ces personnages là
- Pas besoin de retravailler l'output. Ton service peut fournir directement le payload correspondant au pro renvoyé par l'API (ou une liste dans le cas de (2))

## 2. Implémente un système de cache sur la récupération des personnages

Mettre en place du cache sur la récupération des personnages. On te propose d'utiliser un système de cache adapté à un environnement de dev / qui ne nécessite rien de trop contraignant. Par exemple un système de cache /lib basé sur des fichiers temporaire (le default en environnement de dev sur Rails) ou sur la mémoire, voire même une implémentation maison à l'arrache (la qualité du système de cache en lui-même ne fait pas l'objet de l'exercice, il faut juste qu'il marche sur quelques appels services/api)

Considérons le scénario suivant, par exemple on appelle 4 fois le service en console, voici ce qu'il devrait se passer
- Appel du service avec un pro A en argument -> cache miss
- Appel du service avec les pros [B,C] en argument -> cache miss
- Appel du service avec les pros [A,B,C] -> cache hit (idéalement aucun appel HTTP)
- Appel du service avec les pros [A,B,C,D,E] -> cache miss sur D,E (idéalement un seul appel avec les pros D,E)

Pas besoin de se préoccuper de la taille du cache/expiration du cache. Mettre ce que bon te semble / laisser la config par défaut.

## 3. Implémente un service qui est capable de créer un graphe "à la linkedin" des connexions de 1er et 2è niveau entre les personnages

- Le service prend en entrée un ID de personnage P, et est capable de donner la liste des personnages
  - connectés directement (1st). Un personnage P1 est dans (1st) <=> P1 a été présent dans au moins un épisode avec P
  - connectés indirectement (2nd). Un personnage P2 est dans (2nd) <=> P2 a été au moins dans un épisode avec un perso P1 de (1st) mais n'a jamais été en contact avec P

- Au sein de chaque liste, donner un score de proximité
  - (1st) --> le nombre d'épisodes ou P et P1 ont été vus ensemble
  - (2nd) --> on incrémente le score autant de fois que P2 a été dans un épisode avec un pro P1 pour tous les personnages de P1

Un exemple écrit totalement au hasard

> Rick
>   - Episode 1
>   - Episode 2
>   - Episode 3
>   - Episode 4
>
> Morty
>   - Episode 1
>   - Episode 2
>   - Episode 3
>   - Episode 4
>   - Episode 5
>
> Donald
>   - Episode 3
>   - Episode 4
>   - Episode 5
>   - Episode 6
>
> Catwoman
>   - Episode 5
>   - Episode 6
>
> Liliputien
>   - Episode 6
>
> Si on prend Rick
>
> Connection de Premier niveau :
>   - Morty avec un score de 4 (episode 1 à 4 en commun)
>   - Donald avec score de 2 (épisode 3 & 4 en commun)
> 
> Connection de second niveau
>   - Catwoman avec un score de 3 (a vu Morty dans l'épisode 5, a vu Donald dans 5 & 6)
>   - Liliputien avec un score de 1 (a vu Donald dans l'épisode 6)

On ne te demande pas d'implémenter du cache dans cette question.

L'output pour un personnage donné devrait ressembler à ça

```js
{
  "1st_level": [
    { "name": "Morty", "score": 4},
    { "name": "Donald", "score": 2}
  ],
  "2nd_level": [
    { "name": "Catwoman", "score": 3},
    { "name": "Lilipution", "score": 1}
  ],
},
```

## 4. Implémente un API permettant de récupérer des informations sur les personnages

Implémente un endpoint API sur `/api/character/:id` qui doit renvoyer ces information enrichies (en se basant sur les exemples fournis dans les autres questions).

```js

{
  // Données de base via question 1
  "character": {
    "id": 1,
    "name": "Rick",
    "status": "Alive",
    "species": "Human",
    "type": "",
    "gender": "Male",
    "origin": {
      "name": "Earth",
      "url": "https://rickandmortyapi.com/api/location/1"
    },
    "location": {
      "name": "Earth",
      "url": "https://rickandmortyapi.com/api/location/20"
    },
    "image": "https://rickandmortyapi.com/api/character/avatar/1.jpeg",
    "episode": [
      "https://rickandmortyapi.com/api/episode/1",
      "https://rickandmortyapi.com/api/episode/2",
      // ...
    ],
    "url": "https://rickandmortyapi.com/api/character/1",
    "created": "2017-11-04T18:48:46.250Z"
  }

  // Donnée enrichie via question sur le graphe de connexion 3
  "connections": {
    "1st_level": [
      { "name": "Morty", "score": 4},
      { "name": "Donald", "score": 2}
    ],
    "2nd_level": [
      { "name": "Catwoman", "score": 3},
      { "name": "Lilipution", "score": 1}
    ],
  },

  // Données enrichies via la dernière question de l'exercice
  // On te propose dans un premier temps de sérialiser la clé likely_murderers avec une liste vide
  "likely_murderers": [],
}
```

## 5. Masque les personnage qui sont censés être "cachés"

La CIA (Character Identity Anonymization) nous demande d'obfusquer les données des personnages qui sont actuellement en **prison** (à toi de te débrouiller pour touver les détails, tu as des outils pour chercher sur https://rickandmortyapi.com/documentation/#introduction).

Par exemple, en supposant que Rick, Morty et Catwoman dans l'exemple donné 5 sont en prison, on voudrait avoir :

```js
{
  # Données enrichies questions précédentes
  "likely_murderers": [],
  "connections": {
    "1st_level": [
      { "name": "****", "score": 4},
      { "name": "Donald", "score": 2}
    ],
    "2nd_level": [
      { "name": "****", "score": 3},
      { "name": "Liliputien", "score": 1}
    ],
  },

  # Données de base
  "id": 1,
  "name": "****",
  "status": "Alive",
  "species": "****",
  "type": "****",
  "gender": "****",
  "origin": {
    "name": "****",
    "url": "****"
  },
  "location": {
    "name": "****",
    "url": "****"
  },
  "image": "****",
  "episode": [
    "****",
    "****",
    // ...
  ],
  "url": "****",
  "created": "****"
}
```

## 6. R&D : Implémente un service/algo capable de donner des suspects potentiels sur le meurtre d'un personnage et de les trier au mieux

On s'attelle à fournir enfin une vrai sérialisation des `likely_murderers`

Pour un personnage mort donné, on émet l'hypothèse que ce personnage a systématiquement été tué par un autre, et on cherche à savoir quel sont les personnage qui sont susceptibles d'avoir tué ce personnage.

Pas de consigne particulière sur cet exercice : à toi de réfléchir en mode R&D à une logique simple. Ce service
- prend en argument un personnage et renvoie la liste des meutriers potentiels avec un score pour chaque (ou le score est une échelle arbitraire qui dépendra de ton modèle).
- renvoie
  - si le personnage n'est pas mort : une liste vide
  - si le personnage est mort : une liste de suspects identifiés par leur nom et leur score (plus le score est élevé, plus cela veut dire que le perso est susceptible d'avoir tué le premier personnage).


```js
[
  { "name": "Morty", "score": 4},
  { "name": "Donald", "score": 4},
]
```

