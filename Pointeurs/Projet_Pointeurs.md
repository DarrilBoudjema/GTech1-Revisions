# Projet · Gestionnaire de tournoi en C++

Construis une application console pour inscrire des joueurs, sélectionner un participant et attribuer des points. Ton programme devra retrouver le joueur sélectionné même après de nouvelles inscriptions.

## Préparer le projet dans Visual Studio 2026

Crée un projet C++ de type **Application console** et nomme-le `Tournoi`. Dans l’Explorateur de solutions, ouvre le fichier `.cpp` qui contient `main`. Pour démarrer le projet guidé, remplace son contenu par le programme de départ ci-dessous. Lance-le avec **Ctrl + F5**.

## 1. Construire le moteur du tournoi

Voici le programme de départ. Les retours provisoires te permettent de le lancer avant de compléter les fonctions.

```cpp
#include <iostream>
#include <string>
#include <vector>

struct Joueur {
    int id;
    std::string nom;
    int score;
};

Joueur* trouverParId(std::vector<Joueur>& joueurs, int id) {
    // Parcours les joueurs par référence et renvoie l’adresse de celui dont l’identifiant correspond.
    return nullptr;
}

Joueur* trouverDernier(std::vector<Joueur>& joueurs) {
    // Renvoie l'adresse du premier joueur au score minimal.
    return nullptr;
}

bool attribuerBonus(Joueur* joueur, int points) {
    // Si la cible existe et que le bonus est positif ou nul,
    // modifie le score et renvoie true.
    return false;
}

int main() {
    std::vector<Joueur> joueurs{
        {1, "Alice", 12},
        {2, "Bruno", 7},
        {3, "Chloe", 7}
    };

    Joueur* dernier = trouverDernier(joueurs);
    if (attribuerBonus(dernier, 3)) {
        std::cout << dernier->nom << " recoit 3 points.\n";
    }

    for (const Joueur& joueur : joueurs) {
        std::cout << joueur.nom << " : " << joueur.score << '\n';
    }
}
```

1. **Retrouve un joueur.** Complète `trouverParId`. Vérifie que l’identifiant 2 donne accès à Bruno et que 99 renvoie `nullptr`.
2. **Trouve le score minimal.** Dans `trouverDernier`, traite d’abord le vecteur vide. Sinon, initialise un pointeur candidat avec `&joueurs[0]`. Parcours les éléments par référence. Remplace l’adresse du candidat quand tu trouves un score strictement inférieur. Le premier joueur garde ainsi la priorité en cas d’égalité.
3. **Attribue le bonus.** Dans `attribuerBonus`, renvoie `false` si la cible est nulle ou si les points sont négatifs. Sinon, ajoute les points au champ `score` du joueur et renvoie `true`.
4. **Vérifie le résultat.** Les scores doivent être **Alice 12, Bruno 10, Chloe 7**. Si tu appelles de nouveau `trouverDernier`, tu dois obtenir Chloe.

## 2. Ajouter un menu interactif

### Conserver la sélection quand le vecteur grandit

Un `std::vector` range ses éléments dans une zone de mémoire contiguë. Quand une inscription dépasse sa capacité, il déplace les éléments dans une zone plus grande. Les pointeurs vers les anciens emplacements deviennent invalides.

Suppose que tu sélectionnes Bruno et que tu inscrives ensuite plusieurs joueurs. Pour pouvoir retrouver Bruno, conserve son **identifiant** entre les commandes. Au moment du bonus, récupère son adresse actuelle.

```cpp
int idSelectionne = 2;

// Le menu peut inscrire d'autres joueurs ici.

Joueur* cible = trouverParId(joueurs, idSelectionne);
if (attribuerBonus(cible, 5)) {
    std::cout << cible->nom << " : " << cible->score << '\n';
}
```

Le vecteur gère la mémoire de ses joueurs. Utilise le pointeur obtenu pour l’action en cours. Le projet ne nécessite ni `new` ni `delete`.

### Commandes à réaliser

Remplace le scénario fixe dans `main` par une boucle qui lit un choix et exécute la commande correspondante. Garde les trois joueurs initiaux. Initialise `idSelectionne` à 0 pour représenter une sélection vide et `prochainId` à 4 pour la prochaine inscription.

| Choix | Action | Comportement attendu |
|---:|---|---|
| 1 | Afficher | Écrire l’identifiant, le nom et le score de chaque joueur. Si le tournoi est vide, le signaler. |
| 2 | Inscrire | Lire un nom, ajouter un joueur à 0 point et incrémenter `prochainId`. |
| 3 | Sélectionner | Lire un identifiant et le rechercher. Si le joueur existe, mémoriser son identifiant. Sinon, afficher « Joueur introuvable » et remettre la sélection à 0. |
| 4 | Donner un bonus | Retrouver le joueur sélectionné. Si aucun joueur n’est sélectionné, le signaler. Sinon, lire les points et appeler `attribuerBonus`. |
| 5 | Aider le dernier | Appeler `trouverDernier`, puis offrir 3 points à ce joueur. Afficher son nom. Signaler un tournoi vide. |
| 0 | Quitter | Sortir de la boucle. |

**Indice pour les inscriptions.** Si tu lis le numéro de commande avec `std::cin >> choix`, utilise ensuite `std::getline(std::cin >> std::ws, nom)` pour saisir un nom complet. Cela consomme le retour à la ligne laissé par la saisie du numéro et conserve les espaces entre les mots du nom.

### Scénarios de vérification

Repars des scores initiaux pour chaque scénario, sauf celui du tournoi vide.

| Manipulation | Résultat attendu |
|---|---|
| Commencer avec un vecteur vide et choisir « Aider le dernier ». | Le programme signale que le tournoi est vide et revient au menu. |
| Sélectionner Bruno et lui donner 5 points. | Son score passe de 7 à 12. |
| Sélectionner Bruno, saisir ensuite l’identifiant 99, puis demander un bonus. | Le jeu signale l’identifiant inconnu puis l’absence de sélection. Bruno conserve ses 7 points. |
| Sélectionner Alice, inscrire plusieurs joueurs, puis donner 2 points. | Alice passe à 14. Les nouveaux joueurs restent à 0. |
| Choisir « Aider le dernier » deux fois. | Bruno reçoit le premier bonus et Chloe le second. Tous deux finissent à 10. |
| Sélectionner Alice et saisir un bonus de −4. | Le bonus est refusé. Alice conserve ses 12 points. |

**Question de révision.** Pourquoi garder `idSelectionne` entre deux commandes, puis créer `Joueur* cible` au moment d’attribuer le bonus ? Explique ta réponse en reliant l’identité du joueur, son emplacement dans le vecteur et l’effet possible d’une inscription.
