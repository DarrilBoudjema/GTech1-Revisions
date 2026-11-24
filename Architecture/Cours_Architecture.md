# Cours · L’architecture d’un logiciel C++ et les normes de codage

Apprends à reconnaître les responsabilités d’un composant, à lire le sens de ses dépendances et à définir des conventions de codage cohérentes. Les exemples utilisent le C++20. Ils se lisent indépendamment, en connaissant les fonctions et les classes.

## 1. Distinguer responsabilité, interface et implémentation

L’**architecture** décrit comment les grandes parties d’un logiciel collaborent. Elle fixe leurs responsabilités, les interfaces qu’elles exposent et le sens des dépendances entre elles.

| Notion | Question à poser | Exemple |
|---|---|---|
| Responsabilité | Quel comportement cette partie prend-elle en charge ? | Calculer une remise. |
| Interface | Que peuvent demander les autres parties ? | Calculer un montant à partir d’un prix et d’un taux. |
| Contrat | Quelles conditions et garanties accompagnent l’appel ? | Le taux est compris entre 0 et 100. Le résultat est exprimé en centimes. |
| Implémentation | Comment le résultat est-il obtenu ? | Une formule avec une règle d’arrondi définie. |

La **cohésion** mesure à quel point les éléments d’un composant servent une même responsabilité. Le **couplage** décrit la dépendance d’un composant envers un autre. Une règle de calcul qui reçoit un montant et un taux est plus facile à réutiliser qu’une règle qui dépend aussi du format d’un fichier ou de l’état d’une fenêtre.

**À vérifier.** Une règle de calcul lit elle-même le taux dans un fichier. Combien de raisons de modifier ce code peux-tu identifier ? Une évolution du format du fichier et une évolution de la règle touchent toutes deux cet emplacement. Séparer la lecture et le calcul donne à chaque partie une responsabilité indépendante.

## 2. Lire le sens des dépendances

Si un composant A utilise un type ou une fonction de B, A dépend de B. Le choix de cette direction détermine quelles modifications peuvent se propager et quelles parties peuvent être réutilisées seules.

| Partie | Responsabilité fréquente | Exemple |
|---|---|---|
| Présentation | Interpréter une entrée ou présenter un résultat. | Convertir une saisie en données exploitables. |
| Application | Coordonner les étapes d’une opération. | Charger les données, appliquer une règle et déclencher l’enregistrement. |
| Domaine | Porter les règles et les invariants métier. | Refuser une quantité inférieure à zéro. |
| Infrastructure | Communiquer avec des systèmes externes. | Lire un fichier ou interroger une base de données. |

Ces responsabilités peuvent cohabiter dans une même bibliothèque. La séparation devient particulièrement utile lorsqu’elles changent à des rythmes différents ou demandent des tests différents.

### Exercice · Distinguer ordre d’exécution et dépendance

Une opération appelle une fonction de sauvegarde. Quelle partie doit connaître le format enregistré ?

Si l’application appelle directement un type de stockage concret, elle dépend de ce type. Si elle appelle une interface située du côté applicatif, elle dépend de cette interface et l’adaptateur de stockage l’implémente. À l’exécution, l’opération appelle toujours le stockage. Dans le code, l’implémentation technique dépend du contrat applicatif. Ce choix illustre l’**inversion de dépendance**. Il sert notamment lorsque plusieurs moyens de sauvegarde ou des tests isolés sont nécessaires.

## 3. Relier l’architecture au modèle de compilation

Tu peux déclarer une fonction dans un en-tête et la définir dans un fichier source.

```cpp
// remise.hpp
#ifndef TARIFICATION_REMISE_HPP
#define TARIFICATION_REMISE_HPP

namespace tarification {

[[nodiscard]] bool taux_valide(int pourcentage);

}  // namespace tarification

#endif
```

```cpp
// remise.cpp
#include "remise.hpp"

namespace tarification {

bool taux_valide(int pourcentage) {
    return pourcentage >= 0 && pourcentage <= 100;
}

}  // namespace tarification
```

| Écriture | Comment la lire |
|---|---|
| `bool taux_valide(int pourcentage);` | Je déclare la fonction et sa signature. |
| `#include "remise.hpp"` | Le préprocesseur rend les déclarations de l’en-tête disponibles dans ce fichier. |
| `bool taux_valide(int pourcentage) { ... }` | Je définis le comportement de la fonction. |
| `namespace tarification` | Je place la fonction dans un espace de noms. |

Le fichier source et le contenu obtenu après prétraitement forment une **unité de traduction**. Le compilateur traite chaque unité séparément. L’**éditeur de liens** assemble ensuite les fichiers objets et les bibliothèques nécessaires au programme.

Une **cible de construction** regroupe des sources et leurs exigences pour produire, par exemple, une bibliothèque ou un exécutable. Elle permet d’exprimer les dépendances entre les composants. Un **en-tête public** expose une interface accessible aux consommateurs. Un en-tête interne sert à partager des détails entre plusieurs fichiers d’implémentation. Certains éléments, comme les définitions de templates, sont couramment placés dans les en-têtes pour être visibles lors de leur instanciation.

**À vérifier.** Un appel à `taux_valide` compile, mais la construction échoue avec un symbole non défini. La déclaration a permis au compilateur de vérifier l’appel. Vérifie maintenant que la définition a la même signature et que son fichier objet ou sa bibliothèque participe à l’édition de liens.

## 4. Concevoir un contrat et protéger les invariants

Un **invariant** est une propriété que les opérations d’un objet préservent. Par exemple, un taux de remise valide reste compris entre 0 et 100. Si tout le code peut modifier directement la valeur, chaque appelant doit se souvenir de cette règle. En contrôlant la création et les modifications du taux, le type peut garantir lui-même cet invariant.

| Élément du contrat | Exemple |
|---|---|
| Précondition | Un paramètre représente un montant positif exprimé en centimes. |
| Garantie après l’appel | Un refus laisse l’objet dans son état précédent. |
| Effet observable | Une fonction écrit un message, met à jour un objet ou enregistre une donnée. |
| Durée de vie | Une vue retournée reste valide tant que les données qu’elle désigne existent et que leur stockage reste stable. |

Une signature précise rend une partie du contrat visible. `const T&` indique un accès en lecture à travers la référence. Un `enum class` peut nommer plusieurs résultats attendus, comme `accepte`, `invalide` et `deja_present`. Documente ce que la signature ne suffit pas à exprimer, comme l’unité d’un montant ou la validité d’une vue retournée.

## 5. Définir des normes de codage

**C++20** désigne une édition du standard C++. Une **charte de codage** fixe les choix communs à une équipe, par exemple le nommage, le formatage, la gestion des ressources et les conventions d’erreur. Voici un exemple de charte.

| Sujet | Convention | 
|---|---|---|
| Types | `PascalCase` | `TauxRemise` |
| Fonctions et variables | `snake_case` | `taux_valide`, `prix_centimes` |
| Membres privés | `snake_case` avec suffixe `_` | `pourcentage_` |
| Énumérations | `enum class` et valeurs en `snake_case` | `Resultat::taux_invalide` |
| Fichiers et espaces de noms | Minuscules en `snake_case` | `taux_remise.hpp` |
| Mise en forme | Quatre espaces, accolades ouvrantes sur la même ligne et cible de 100 colonnes | `if (taux_valide(taux)) {` |
| Inclusions | En-têtes autonomes, garde unique et en-tête correspondant inclus en premier dans son `.cpp` | `#include "taux_remise.hpp"` |

Initialise tes variables dès leur déclaration et limite leur portée. Garde les accolades autour des blocs de contrôle. Évite `using namespace` au niveau global, surtout dans les en-têtes. Réserve les commentaires aux contrats, aux raisons d’un choix et aux cas qui demandent une explication.

### Lire la propriété et la durée de vie dans les types

| Besoin | Type courant | Ce que ça implique |
|---|---|---|
| Conserver du texte | `std::string` | La chaîne gère son stockage. |
| Lire du texte pendant un appel | `std::string_view` | Les caractères empruntés doivent rester valides pendant leur utilisation. |
| Accéder à un objet obligatoire | `const T&` ou `T&` | L’appel emprunte un objet. La forme avec `const` limite l’accès à la lecture. |
| Accéder à un objet facultatif | `T*` avec `nullptr` documenté | Le pointeur peut désigner un objet ou représenter son absence. |
| Gérer la propriété exclusive d’un objet alloué | `std::unique_ptr<T>` | Le pointeur intelligent gère la destruction de l’objet. |
| Exprimer l’absence d’une valeur | `std::optional<T>` | Le résultat contient une valeur ou représente son absence. |

Le **RAII** lie une ressource à la durée de vie d’un objet. Un `std::ifstream` ferme par exemple son fichier à sa destruction, y compris lorsqu’une exception provoque la sortie de sa portée. Utiliser des membres qui gèrent leurs ressources permet souvent d’appliquer la **règle du zéro**. La classe laisse alors le compilateur générer les opérations spéciales dont elle a besoin, comme la destruction et, si ses membres le permettent, la copie ou le déplacement.

### Exprimer les erreurs et vérifier les conventions

Un résultat comme `Resultat::taux_invalide` représente un refus prévu par le contrat. Si le projet utilise des exceptions, définis celles qui peuvent franchir une interface et intercepte-les là où le programme sait réagir. Le formatage est vérifiable avec `clang-format`, certaines règles de code avec `clang-tidy` et les contrats observables avec des tests. Les avertissements du compilateur complètent ces vérifications, par exemple avec `-Wall -Wextra -Wpedantic` sous GCC ou Clang, ou `/W4` sous MSVC.

## 6. Vérifier ta compréhension

1. **Dépendances.** Une classe métier inclut le type d’une fenêtre pour afficher une erreur. Quelle information pourrait-elle transmettre à la place ?  
   **Réponse.** Elle peut fournir un résultat métier. La présentation choisit ensuite le message et la façon de l’afficher.
2. **Compilation.** Un en-tête contient la définition d’une fonction ordinaire non inline et il est inclus par deux fichiers source. Que risques-tu ?  
   **Réponse.** Chaque unité de traduction contient une définition. Le programme enfreint la règle de définition unique et l’éditeur de liens peut signaler des définitions multiples.
3. **Durée de vie.** Une fonction crée une chaîne locale et retourne une `std::string_view` vers elle. Quand la vue devient-elle invalide ?  
   **Réponse.** À la destruction de la chaîne locale, quand la fonction se termine. Renvoyer une `std::string` permet de transmettre le texte en tant que valeur possédée.
