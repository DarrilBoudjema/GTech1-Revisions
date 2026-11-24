# C++ — Héritage, classes abstraites et interfaces

> **Fiche de révision approfondie** — C++17 et versions ultérieures  
> **Prérequis :** savoir déclarer une classe, écrire un constructeur et utiliser `public` / `private`.  
> **Objectif :** comprendre, écrire et expliquer l'héritage, une classe abstraite et une interface en C++.

## Sommaire

1. [L'héritage](#1-lhéritage)
2. [Le polymorphisme et les méthodes virtuelles](#2-le-polymorphisme-et-les-méthodes-virtuelles)
3. [Les classes abstraites](#3-les-classes-abstraites)
4. [Les interfaces en C++](#4-les-interfaces-en-c)
5. [Les différences essentielles](#5-les-différences-essentielles)
6. [Un exemple complet : système de notifications](#6-un-exemple-complet--système-de-notifications)
7. [Les erreurs fréquentes](#7-les-erreurs-fréquentes)

---

## 1. L'héritage

### 1.1 Définition

L'**héritage** permet à une classe **dérivée** (ou *fille*) de réutiliser et de spécialiser les caractéristiques d'une classe **de base** (ou *mère*).

On utilise généralement l'héritage lorsque l'on peut dire : **« un B est un A »** (*relation is-a*).

- Un `Chien` **est un** `Animal` : héritage adapté.
- Un `Moteur` **fait partie d'une** `Voiture` : préférer la composition, pas l'héritage.

### 1.2 Syntaxe

```cpp
#include <iostream>
#include <string>

class Animal {
public:
    explicit Animal(std::string nom) : nom_(std::move(nom)) {}

    void dormir() const {
        std::cout << nom_ << " dort.\n";
    }

protected:
    std::string nom_;
};

class Chien : public Animal {
public:
    explicit Chien(std::string nom) : Animal(std::move(nom)) {}

    void aboyer() const {
        std::cout << nom_ << " aboie.\n";
    }
};

int main() {
    Chien rex("Rex");
    rex.dormir(); // méthode héritée
    rex.aboyer(); // méthode propre à Chien
}
```

**À retenir :** `class Chien : public Animal` signifie que `Chien` hérite **publiquement** de `Animal`. Le constructeur de `Chien` initialise explicitement la partie `Animal`.

> L'exemple utilise `std::move` : en code réel, inclure aussi `<utility>`.

### 1.3 `public`, `protected`, `private`

Ces mots désignent les droits d'accès **aux membres**, mais aussi un mode d'héritage.

| Visibilité d'un membre dans la base | Depuis la classe dérivée | Depuis `main()` |
|---|---|---|
| `public` | Accessible | Accessible |
| `protected` | Accessible | Inaccessible |
| `private` | Inaccessible directement | Inaccessible |

Les membres `private` **font toujours partie de l'objet dérivé**, mais la classe dérivée ne peut pas les lire directement : elle passe par une méthode accessible de la base.

**Effet du mode d'héritage :**

| Héritage | Membres `public` de la base deviennent | Membres `protected` deviennent |
|---|---|---|
| `public` | `public` | `protected` |
| `protected` | `protected` | `protected` |
| `private` | `private` | `private` |

Les membres `private` de la base restent directement inaccessibles dans tous les cas.

En POO classique, **l'héritage `public` est le plus fréquent**, car il exprime une vraie relation « est un ».

### 1.4 Constructeurs et destructeurs

Lorsqu'on crée un objet dérivé :

1. La base est construite en premier.
2. Puis la classe dérivée est construite.
3. Lors de la destruction, c'est l'inverse : dérivée, puis base.

```cpp
#include <iostream>

class Base {
public:
    Base()  { std::cout << "Base construite\n"; }
    virtual ~Base() { std::cout << "Base detruite\n"; }
};

class Fille : public Base {
public:
    Fille()  { std::cout << "Fille construite\n"; }
    ~Fille() override { std::cout << "Fille detruite\n"; }
};

int main() {
    Fille objet;
}
```

Affichage :

```text
Base construite
Fille construite
Fille detruite
Base detruite
```

**Pourquoi `virtual ~Base()` ?** Si un objet dérivé est détruit par un pointeur sur la base (`Base*`), le destructeur virtuel garantit la bonne destruction de l'objet complet. C'est essentiel pour une base polymorphe.

### 1.5 Conversion vers la classe mère

Une référence ou un pointeur vers la base peut désigner un objet dérivé :

```cpp
Chien rex("Rex");
Animal& animal = rex; // conversion implicite vers la base
animal.dormir();
```

Attention : **l'interface visible dépend du type de la référence**. `animal.aboyer()` ne compile pas, car `aboyer()` n'est pas déclaré dans `Animal`.

---

## 2. Le polymorphisme et les méthodes virtuelles

### 2.1 Pourquoi `virtual` ?

Le **polymorphisme dynamique** permet d'appeler une méthode adaptée au **type réel de l'objet**, même via une référence ou un pointeur vers une classe de base.

```cpp
#include <iostream>

class Animal {
public:
    virtual ~Animal() = default;

    virtual void parler() const {
        std::cout << "Son inconnu\n";
    }
};

class Chien : public Animal {
public:
    void parler() const override {
        std::cout << "Ouaf !\n";
    }
};

class Chat : public Animal {
public:
    void parler() const override {
        std::cout << "Miaou !\n";
    }
};

void faireParler(const Animal& animal) {
    animal.parler();
}

int main() {
    Chien chien;
    Chat chat;
    faireParler(chien); // Ouaf !
    faireParler(chat);  // Miaou !
}
```

- `virtual` : rend une méthode redéfinissable avec **liaison dynamique**.
- `override` : demande au compilateur de vérifier qu'on redéfinit bien une méthode virtuelle héritée. **À utiliser systématiquement** lorsqu'on redéfinit.
- `final` : interdit une nouvelle redéfinition (`void parler() const final`) ou, appliqué à une classe, interdit d'en dériver.

### 2.2 Attention à la signature

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual void afficher() const {}
};

class Fille : public Base {
public:
    // Erreur de compilation volontaire : manque const
    // void afficher() override {}

    void afficher() const override {} // correct
};
```

La signature doit correspondre, notamment pour `const` et les paramètres. `override` détecte immédiatement les erreurs.

### 2.3 Éviter le *slicing* (découpage d'objet)

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual int valeur() const { return 1; }
};

class Fille : public Base {
public:
    int valeur() const override { return 2; }
};

Fille fille;
Base copie = fille; // découpe la partie spécifique de Fille
// copie.valeur() renvoie 1 ; il s'agit d'un objet Base indépendant

Base& reference = fille;
// reference.valeur() renvoie 2 : le polymorphisme fonctionne
```

Pour travailler polymorphiquement, utiliser plutôt des **références**, des **pointeurs** ou des **smart pointers** (`std::unique_ptr<Base>`).

---

## 3. Les classes abstraites

### 3.1 Définition

Une **classe abstraite** est une classe contenant **au moins une fonction virtuelle pure**, déclarée avec `= 0`.

On **ne peut pas créer directement** d'objet d'une classe abstraite. Elle sert de contrat et éventuellement de **base commune contenant du code et de l'état**.

### 3.2 Exemple : formes géométriques

```cpp
#include <cmath>
#include <iostream>

class Forme {
public:
    virtual ~Forme() = default;

    virtual double aire() const = 0; // méthode virtuelle pure

    void afficherAire() const {     // méthode déjà implémentée
        std::cout << "Aire = " << aire() << '\n';
    }
};

class Rectangle : public Forme {
public:
    Rectangle(double largeur, double hauteur)
        : largeur_(largeur), hauteur_(hauteur) {}

    double aire() const override {
        return largeur_ * hauteur_;
    }

private:
    double largeur_;
    double hauteur_;
};

int main() {
    // Forme f; // interdit : classe abstraite
    Rectangle r(4.0, 3.0);
    r.afficherAire(); // Aire = 12
}
```

### 3.3 Important : méthodes virtuelles pures et héritage

Si une classe dérivée **n'implémente pas toutes les méthodes virtuelles pures héritées**, elle reste elle aussi abstraite.

```cpp
class Forme3D : public Forme {
public:
    virtual double volume() const = 0;
    // aire() n'est pas implémentée : Forme3D reste abstraite
};
```

Une classe abstraite peut contenir :

- des attributs et un constructeur ;
- des méthodes normales (avec implémentation) ;
- des méthodes `virtual` avec implémentation ;
- des méthodes virtuelles pures `= 0` ;
- un destructeur (généralement `virtual`).

**Nuance :** même une fonction virtuelle pure peut posséder une définition en dehors de la classe, si on en a besoin. Elle reste pure et rend la classe abstraite.

---

## 4. Les interfaces en C++

### 4.1 Définition

Contrairement à Java ou C#, **C++ n'a pas de mot-clé `interface`**.

Par convention, une **interface** est une classe abstraite qui définit principalement ou exclusivement un **contrat** : des fonctions virtuelles pures, **sans état d'instance** ni logique métier commune.

Il s'agit donc d'une **convention de conception**, et non d'une catégorie distincte imposée par le langage.

### 4.2 Exemple : `IExportable`

```cpp
#include <iostream>
#include <string>

class IExportable {
public:
    virtual ~IExportable() = default;
    virtual std::string exporter() const = 0;
};

class Facture : public IExportable {
public:
    std::string exporter() const override {
        return "Facture;numero=42";
    }
};

class Rapport : public IExportable {
public:
    std::string exporter() const override {
        return "Rapport;resultat=OK";
    }
};

void sauvegarder(const IExportable& objet) {
    std::cout << objet.exporter() << '\n';
}

int main() {
    Facture facture;
    Rapport rapport;
    sauvegarder(facture);
    sauvegarder(rapport);
}
```

Avantage : la fonction `sauvegarder` dépend du **contrat `IExportable`**, pas des types concrets `Facture` et `Rapport`.

### 4.3 Une classe peut implémenter plusieurs interfaces

```cpp
#include <iostream>
#include <string>

class IImprimable {
public:
    virtual ~IImprimable() = default;
    virtual void imprimer() const = 0;
};

class IIdentifiable {
public:
    virtual ~IIdentifiable() = default;
    virtual int id() const = 0;
};

class Document : public IImprimable, public IIdentifiable {
public:
    void imprimer() const override {
        std::cout << "Impression du document\n";
    }

    int id() const override { return 123; }
};
```

C'est de l'**héritage multiple**. Il est particulièrement utile pour combiner plusieurs interfaces indépendantes. En revanche, l'héritage multiple de classes avec état peut créer des ambiguïtés et demande plus de prudence.

### 4.4 Interface ou classe abstraite ?

Une interface convient particulièrement lorsqu'on veut dire **« cet objet sait faire telle opération »** : exportable, imprimable, sérialisable, etc.

Une classe abstraite avec état et comportements communs est utile lorsque plusieurs classes appartiennent à la **même famille** et peuvent partager du code : `Forme`, `Animal`, `Vehicule`, etc.

---

## 5. Les différences essentielles

| Critère | Héritage | Classe abstraite | Interface (convention C++) |
|---|---|---|---|
| Nature | Mécanisme entre classes | Classe avec au moins une méthode virtuelle pure | Classe abstraite conçue comme contrat |
| Instanciable directement ? | Cela dépend de la classe | **Non** | **Non** |
| Contient des attributs ? | Possible | Oui | Généralement aucun état d'instance |
| Contient des méthodes définies ? | Oui | Oui | Généralement non, hormis le destructeur |
| Objectif principal | Réutiliser et spécialiser, exprimer « est un » | Partager une base + imposer des opérations | Imposer des capacités sans dépendre de l'implémentation |
| Mot-clé central | `: public Base` | `virtual methode() = 0` | Même syntaxe que la classe abstraite |

**Phrase clé :** l'héritage est un **mécanisme**, une classe abstraite est un **type de classe**, une interface est un **rôle de conception**.

---

## 6. Un exemple complet : système de notifications

Cet exemple combine **une classe abstraite** avec un état commun, **une interface** et **plusieurs classes dérivées**. Il illustre aussi les smart pointers.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>
#include <vector>

// Interface : capacité à envoyer une notification
class IEnvoyable {
public:
    virtual ~IEnvoyable() = default;
    virtual void envoyer() const = 0;
};

// Classe abstraite : base commune aux messages
class Message : public IEnvoyable {
public:
    explicit Message(std::string texte) : texte_(std::move(texte)) {}
    ~Message() override = default;

    virtual std::string canal() const = 0;

    void afficherResume() const {
        std::cout << "[" << canal() << "] " << texte_ << '\n';
    }

protected:
    std::string texte_;
};

class Email : public Message {
public:
    explicit Email(std::string texte) : Message(std::move(texte)) {}

    std::string canal() const override { return "EMAIL"; }

    void envoyer() const override {
        std::cout << "Envoi email : " << texte_ << '\n';
    }
};

class SMS : public Message {
public:
    explicit SMS(std::string texte) : Message(std::move(texte)) {}

    std::string canal() const override { return "SMS"; }

    void envoyer() const override {
        std::cout << "Envoi SMS : " << texte_ << '\n';
    }
};

int main() {
    std::vector<std::unique_ptr<Message>> messages;
    messages.push_back(std::make_unique<Email>("Bonjour !"));
    messages.push_back(std::make_unique<SMS>("Rendez-vous a 15h"));

    for (const auto& message : messages) {
        message->afficherResume();
        message->envoyer();
    }
}
```

Compilation :

```bash
g++ -std=c++17 -Wall -Wextra -pedantic main.cpp -o app
./app
```

Sortie attendue :

```text
[EMAIL] Bonjour !
Envoi email : Bonjour !
[SMS] Rendez-vous a 15h
Envoi SMS : Rendez-vous a 15h
```

**Ce que cet exemple démontre :**

1. `IEnvoyable` est une **interface** : elle exige `envoyer()`.
2. `Message` est **abstraite** : elle contient `texte_`, `afficherResume()` et la méthode pure `canal()` ; elle hérite aussi de `envoyer()` non implémentée.
3. `Email` et `SMS` sont des classes **concrètes** : elles implémentent toutes les méthodes virtuelles pures.
4. Le `std::vector<std::unique_ptr<Message>>` stocke différents types dérivés **sans slicing** et gère automatiquement la mémoire.
5. Les appels à `canal()` et `envoyer()` utilisent le **polymorphisme dynamique**.

---

## 7. Les erreurs fréquentes

### Erreur A — oublier `public`

```cpp
class A {};
class B : A {}; // héritage private par défaut si B est une class
```

Écrire `class B : public A` pour exprimer une relation de sous-type publique.

### Erreur B — oublier `virtual` dans la base

```cpp
class A {
public:
    void parler() const {} // non virtuelle
};
```

Une méthode redéclarée dans la fille masque celle de la base, mais les appels via `A&` ou `A*` ne bénéficient pas du polymorphisme dynamique.

### Erreur C — oublier le destructeur virtuel

```cpp
class Base {
public:
    virtual void f() = 0;
    // ~Base() non virtuel : risque si delete via Base*
};
```

Pour une classe destinée à une utilisation polymorphe, préférer `virtual ~Base() = default;`.

### Erreur D — confondre classe abstraite et interface

Toute interface au sens usuel est une classe abstraite, mais **toute classe abstraite n'est pas une interface** : elle peut posséder des données et du comportement partagé.

### Erreur E — oublier une méthode virtuelle pure

```cpp
class ITest {
public:
    virtual ~ITest() = default;
    virtual void a() = 0;
    virtual void b() = 0;
};

class Test : public ITest {
public:
    void a() override {}
    // b() manque : Test reste abstraite
};
```

### Erreur F — passer un objet polymorphe par valeur

```cpp
// Préférer ceci :
void traiter(const IExportable& objet);
// Plutôt qu'une classe de base concrète passée par valeur,
// qui peut provoquer du slicing.
```

---
