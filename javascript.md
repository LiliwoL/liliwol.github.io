http://www.logique.jussieu.fr/~nderugy/doc/ProgWeb/javascript.pdf

## Objets

https://developer.mozilla.org/fr/docs/Learn/JavaScript/Objects/Basics

### Déclaration d'un nouvel objet vide
```js
var personne = {};
```

Avec la console:

```console
personne

> [object Object]
```
***

### Déclaration d'un objet avec des données

```js
var personne = {
  nom: ['Jean', 'Martin'],
  age: 32,
  sexe: 'masculin',
  interets: ['musique', 'skier'],
  bio: function() {
    alert(this.nom[0] + ' ' + this.nom[1] + ' a ' + this.age + ' ans. Il aime ' + this.interets[0] + ' et ' + this.interets[1] + '.');
  },
  salutation: function() {
    alert('Bonjour ! Je suis ' + this.nom[0] + '.');
  }
};
```

Accès aux données de l'objet:
```console
personne.nom
personne.nom[0]
personne.age
personne.interets[1]
personne.bio()
personne.salutation()
```

