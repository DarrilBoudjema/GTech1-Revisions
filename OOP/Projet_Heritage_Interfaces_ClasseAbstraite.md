# Projet pratique C++ — Gestion d'une médiathèque

> **Sujet de TP à réaliser en autonomie**  
> **Thèmes :** héritage, classes abstraites, interfaces, méthodes virtuelles et polymorphisme.  
> **Standard :** C++17

## 1. Objectif du projet

Tu dois développer une application **console** permettant de gérer plusieurs types de documents d'une médiathèque : des **livres**, des **DVD** et des **magazines**.

Tous les documents possèdent un titre et peuvent être affichés. Cependant, seuls les livres et les DVD sont empruntables. Le programme doit pouvoir manipuler différents documents à travers une **classe de base abstraite**, sans avoir besoin de connaître leur type exact lorsqu'il les affiche.

À la fin du projet, tu dois savoir :

- définir une classe mère et plusieurs classes filles ;
- utiliser `public`, `protected` et `private` correctement ;
- déclarer une **méthode virtuelle pure** avec `= 0` ;
- définir une **interface** ne contenant que des opérations à implémenter ;
- redéfinir des méthodes avec `override` ;
- utiliser le polymorphisme via une référence ou un pointeur vers une classe de base ;
- expliquer pourquoi un destructeur virtuel est nécessaire.

> **Important :** ce document est volontairement un **énoncé guidé, sans solution complète**. L'objectif est d'écrire toi-même les classes et les méthodes.

---

## 2. Cahier des charges

### Fonctionnalités obligatoires

1. Créer un livre avec un titre, un auteur et un nombre de pages.
2. Créer un DVD avec un titre, un réalisateur et une durée en minutes.
3. Créer un magazine avec un titre et un numéro de publication.
4. Afficher les informations propres à chaque type de document.
5. Emprunter un livre ou un DVD lorsqu'il est disponible.
6. Refuser l'emprunt d'un document déjà emprunté.
7. Retourner un livre ou un DVD emprunté.
8. Afficher tous les documents à l'aide du **polymorphisme**.
9. Empêcher l'instanciation directe de `Document` et de `IEmpruntable`.

### Règles métier

- Un document est identifié au minimum par son titre.
- Un livre et un DVD sont **disponibles** lors de leur création.
- Un magazine n'est **pas empruntable**.
- Un emprunt réussi change l'état du document.
- Un retour réussi rend le document disponible à nouveau.
- Un emprunt ou un retour impossible renvoie `false` ; une opération réussie renvoie `true`.

---

## 3. Architecture demandée

```text
                  Document (classe abstraite)
                  - titre_ : string
                  + afficher() const = 0
                  + ~Document() virtual
                             ^
               _____________|_____________
              |             |             |
            Livre          DVD         Magazine
              |             |
              |             |
       IEmpruntable      IEmpruntable
         (interface)      (interface)

Livre et DVD héritent de Document ET de IEmpruntable.
Magazine hérite uniquement de Document.
```

### Rôles des classes

| Classe | Type | Responsabilité |
|---|---|---|
| `Document` | Classe abstraite | Fournir un titre commun et imposer `afficher()` |
| `IEmpruntable` | Interface | Définir le contrat d'emprunt et de retour |
| `Livre` | Classe concrète | Afficher auteur/pages et gérer l'emprunt |
| `DVD` | Classe concrète | Afficher réalisateur/durée et gérer l'emprunt |
| `Magazine` | Classe concrète | Afficher son numéro sans possibilité d'emprunt |

**Question de compréhension :** pourquoi `IEmpruntable` ne doit-elle pas contenir de titre, d'auteur ou de réalisateur ?

---

## 4. Organisation des fichiers

```text
mediatheque/
├── include/
│   ├── Document.hpp
│   ├── IEmpruntable.hpp
│   ├── Livre.hpp
│   ├── DVD.hpp
│   └── Magazine.hpp
├── src/
│   ├── Document.cpp
│   ├── Livre.cpp
│   ├── DVD.cpp
│   ├── Magazine.cpp
│   └── main.cpp
└── README.md
```

Tu peux aussi commencer en écrivant tout dans un seul `main.cpp`, puis séparer le code une fois qu'il fonctionne.

---

## 5. Étape 1 — Créer la classe abstraite `Document`

Dans `Document.hpp`, écris une classe avec :

- un attribut `titre_` de type `std::string` ;
- un constructeur recevant le titre ;
- un accesseur `getTitre() const` ;
- une méthode **virtuelle pure** `afficher() const` ;
- un **destructeur virtuel**.

Squelette de départ :

```cpp
#pragma once
#include <string>

class Document {
public:
    explicit Document(const std::string& titre);
    virtual ~Document() = default;

    const std::string& getTitre() const;

    // TODO : déclarer afficher() comme méthode virtuelle pure

protected:
    std::string titre_;
};
```

Dans `Document.cpp`, implémente le constructeur et l'accesseur.

**Vérification :** le code `Document d("Essai");` ne doit **pas** compiler dès lors que `afficher()` est pure.

**Questions :**

1. Pourquoi `Document` est-elle abstraite ?
2. Une classe abstraite peut-elle posséder un constructeur ?
3. Pourquoi `~Document()` est-il `virtual` ?

---

## 6. Étape 2 — Créer l'interface `IEmpruntable`

Dans `IEmpruntable.hpp`, définis une interface contenant exactement ces opérations :

```cpp
#pragma once

class IEmpruntable {
public:
    virtual ~IEmpruntable() = default;

    virtual bool emprunter() = 0;
    virtual bool retourner() = 0;
    virtual bool estDisponible() const = 0;
};
```

L'interface ne doit **pas** enregistrer elle-même l'état des documents. Chaque classe concrète est responsable de son comportement.

**Questions :**

1. Pourquoi `IEmpruntable` est-elle abstraite ?
2. Peut-on écrire `IEmpruntable objet;` ?
3. En quoi une interface se distingue-t-elle ici de `Document` ?

---

## 7. Étape 3 — Créer la classe `Livre`

La classe `Livre` doit hériter **publiquement** de `Document` et de `IEmpruntable` :

```cpp
class Livre : public Document, public IEmpruntable {
    // TODO
};
```

Attributs à ajouter :

- `auteur_` : `std::string` ;
- `pages_` : `int` ;
- `disponible_` : `bool`, initialisé à `true`.

Méthodes à écrire :

- constructeur `(titre, auteur, pages)` ;
- `void afficher() const override` ;
- `bool emprunter() override` ;
- `bool retourner() override` ;
- `bool estDisponible() const override`.

**Comportement attendu :**

```text
Livre : Le Petit Prince
Auteur : Antoine de Saint-Exupéry
Pages : 96
Statut : disponible
```

Scénario à tester :

```text
Premier emprunt  -> true
Deuxième emprunt -> false
Retour           -> true
Deuxième retour  -> false
Disponible       -> true
```

**Point important :** initialise la partie `Document` dans la liste d'initialisation du constructeur `Livre`.

---

## 8. Étape 4 — Créer la classe `DVD`

La classe `DVD` hérite elle aussi de `Document` et de `IEmpruntable`.

Elle possède :

- `realisateur_` : `std::string` ;
- `dureeMinutes_` : `int` ;
- `disponible_` : `bool`.

Implémente toutes les méthodes virtuelles héritées, avec `override`.

**Affichage attendu :**

```text
DVD : Interstellar
Réalisateur : Christopher Nolan
Durée : 169 minutes
Statut : emprunté
```

**Question :** si tu oublies d'implémenter `retourner()`, pourquoi ne peux-tu plus créer de `DVD` ?

---

## 9. Étape 5 — Créer la classe `Magazine`

La classe `Magazine` hérite **uniquement** de `Document`.

Elle possède un attribut `numero_` de type `int`, un constructeur et une redéfinition de `afficher()`.

Affichage attendu :

```text
Magazine : Science & Vie
Numéro : 1280
```

**À vérifier :** `Magazine` ne possède pas de méthode `emprunter()` imposée par une interface.

**Question :** pourquoi serait-ce une erreur de faire hériter `Magazine` de `IEmpruntable` si la médiathèque interdit son emprunt ?

---

## 10. Étape 6 — Utiliser le polymorphisme

Dans `main.cpp`, crée une collection contenant **au moins** deux livres, un DVD et un magazine. Affiche-les sans appeler directement `Livre::afficher()`, `DVD::afficher()` ou `Magazine::afficher()`.

Tu peux utiliser cette structure :

```cpp
#include <iostream>
#include <memory>
#include <vector>

#include "Document.hpp"
#include "Livre.hpp"
#include "DVD.hpp"
#include "Magazine.hpp"

int main() {
    std::vector<std::unique_ptr<Document>> documents;

    // TODO : utiliser std::make_unique<Livre>(...)
    // TODO : utiliser std::make_unique<DVD>(...)
    // TODO : utiliser std::make_unique<Magazine>(...)

    for (const auto& document : documents) {
        document->afficher();
        std::cout << "\n";
    }
}
```

**À observer :** `document` est manipulé comme un `Document`, mais la version de `afficher()` exécutée dépend du **type réel** de l'objet. C'est du **polymorphisme dynamique**.

**Questions :**

1. Pourquoi utiliser `std::unique_ptr<Document>` plutôt que `std::vector<Document>` ?
2. Que se passerait-il si `afficher()` n'était pas `virtual` ?
3. Que t'apporte `override` ?

> Si tu n'as pas encore étudié `unique_ptr`, tu peux d'abord démontrer le polymorphisme avec une fonction `void afficherDocument(const Document& document)` qui appelle `document.afficher()` et lui passer successivement un livre, un DVD et un magazine.

---

## 11. Étape 7 — Manipuler les objets par leur interface

Écris cette fonction **en dehors des classes** :

```cpp
void testerEmprunt(IEmpruntable& objet) {
    // TODO : afficher la disponibilité initiale
    // TODO : emprunter deux fois
    // TODO : retourner l'objet
    // TODO : afficher la disponibilité finale
}
```

Dans `main()`, passe-lui un objet `Livre` puis un objet `DVD`.

L'idée est que `testerEmprunt` fonctionne avec **n'importe quel objet qui implémente `IEmpruntable`**, sans dépendre de son type concret.

**Défi :** explique la différence entre :

```cpp
void voir(const Document& document);
void testerEmprunt(IEmpruntable& objet);
```

La première fonction attend un **document** ; la seconde attend un objet qui respecte un **contrat d'emprunt**.

---

## 12. Tests à réaliser

Coche chaque test après l'avoir vérifié dans ton programme.

- [ ] `Document` ne peut pas être instanciée.
- [ ] `IEmpruntable` ne peut pas être instanciée.
- [ ] `Livre`, `DVD` et `Magazine` peuvent être instanciés.
- [ ] L'affichage d'un livre comporte son auteur et son nombre de pages.
- [ ] L'affichage d'un DVD comporte son réalisateur et sa durée.
- [ ] L'affichage d'un magazine comporte son numéro.
- [ ] Un livre est disponible à sa création.
- [ ] Le premier emprunt réussit et le deuxième échoue.
- [ ] Un retour réussit seulement si le document a été emprunté.
- [ ] Une collection de `Document` affiche les bons types concrets.
- [ ] `testerEmprunt(IEmpruntable&)` fonctionne avec un livre et un DVD.
- [ ] Les méthodes redéfinies portent le mot-clé `override`.
- [ ] Le projet compile sans erreur avec `-Wall -Wextra -pedantic`.


---

## 13. Bonus (facultatifs)

Ces ajouts restent dans le périmètre des trois notions principales.

**Bonus A — Un quatrième type :** crée une classe `LivreAudio` héritant de `Document` et de `IEmpruntable`, avec un narrateur et une durée.

**Bonus B — Interface supplémentaire :** crée une interface `IConsultable` avec `virtual void consulter() const = 0;`. Fais-la implémenter par `Magazine` et `Livre`, puis écris une fonction `consulterObjet(const IConsultable&)`.

**Bonus C — Polymorphisme avancé :** crée une fonction `afficherTous(const std::vector<std::unique_ptr<Document>>& documents)` qui affiche toute la collection sans tester les types concrets.

---

## 14. Barème d'auto-évaluation /20

| Critère | Points |
|---|---:|
| `Document` correctement abstraite | 3 |
| Interface `IEmpruntable` correcte | 3 |
| Héritage et implémentations de `Livre` | 3 |
| Héritage et implémentations de `DVD` | 3 |
| `Magazine` correctement conçue | 2 |
| Polymorphisme via `Document` | 3 |
| Utilisation polymorphe de `IEmpruntable` | 2 |
| Code propre, `override`, destructeurs virtuels | 1 |
| **Total** | **20** |

### Questions de soutenance

Sans regarder le cours, essaie de répondre à l'oral :

1. Quelle différence fais-tu entre une classe concrète et une classe abstraite ?
2. Pourquoi `Document::afficher()` est-elle `virtual` et pure ?
3. Pourquoi une interface C++ est-elle généralement constituée de méthodes virtuelles pures ?
4. Pourquoi `Livre` peut-il hériter de deux classes ici ?
5. Quelle différence y a-t-il entre **hériter** d'une classe et **implémenter** un contrat ?
6. Qu'apporte le polymorphisme lorsque tu ajoutes `LivreAudio` ?
7. À quoi sert le destructeur virtuel de `Document` ?

---

## 15. Critère de réussite final

**Le projet est terminé si :** tu peux créer des documents de types différents, les afficher à travers une référence ou un pointeur `Document`, emprunter et retourner les seuls documents concernés au travers de `IEmpruntable`, et expliquer **pourquoi** chaque héritage existe.

**À ne pas faire :** remplacer le polymorphisme par une grande chaîne de `if` qui teste si le document est un livre, un DVD ou un magazine. Le but est précisément de laisser les méthodes virtuelles sélectionner le bon comportement.

**Livrable :** le dossier de ton code source et, dans le `README.md`, trois courts paragraphes expliquant l'héritage, la classe abstraite et l'interface utilisés dans ton projet.
