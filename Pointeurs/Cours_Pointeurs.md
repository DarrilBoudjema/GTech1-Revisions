# Cours · Les pointeurs en C++

Apprends à lire une adresse, à modifier un objet à travers un pointeur et à reconnaître quand une adresse reste valide. Les exemples peuvent être explorés dans une application console C++ de Visual Studio 2026.

## 1. Lire une adresse et la suivre

```cpp
int score = 12;
int* p = &score;
*p += 3;
```

| Écriture | Comment la lire |
|---|---|
| `int* p` | Je déclare une variable qui peut contenir l’adresse d’un entier. |
| `&score` | Je prends l’adresse de la variable `score`. |
| `p = &score` | Je mémorise cette adresse dans `p`. |
| `*p` | Je suis l’adresse contenue dans `p` pour accéder à l’entier. |
| `*p += 3` | J’ajoute 3 à cet entier. La variable `score` vaut maintenant 15. |

L’étoile a deux usages. Dans une déclaration comme `int* p`, elle fait partie du type. Dans une expression comme `*p += 3`, elle donne accès à l’objet pointé. Cette opération s’appelle le **déréférencement**.

### Exercice · Deux pointeurs, deux scores

Avant de lire les résultats, prédis les valeurs finales d’`alice` et de `bruno`.

```cpp
int alice = 12;
int bruno = 7;
int* p = &alice;
int* q = p;

*p += 3;
p = &bruno;
*q = 20;
```

| Après l’instruction | Cible de `p` | Cible de `q` | `alice` | `bruno` |
|---|---|---|---:|---:|
| `int* q = p;` | alice | alice | 12 | 7 |
| `*p += 3;` | alice | alice | 15 | 7 |
| `p = &bruno;` | bruno | alice | 15 | 7 |
| `*q = 20;` | bruno | alice | 20 | 7 |

`q = p` copie une adresse. Chaque pointeur conserve ensuite sa propre valeur. Pour copier le **score** d’Alice chez Bruno à la fin de l’exemple, tu écrirais `*p = *q;`.

**À observer dans Visual Studio.** Place ces lignes dans `main`. Pose un point d’arrêt sur `*p += 3;` avec **F9**, puis lance le débogage avec **F5**. Dans la fenêtre *Espion*, ajoute `p`, `q`, `*p` et `*q`. Avance avec **F10**. À quelle instruction les adresses deviennent-elles différentes ?

## 2. Passer un pointeur à une fonction

```cpp
void ajouterPoint(int* score) {
    if (score != nullptr) {
        ++(*score);
    }
}
```

Avec `int total = 4;`, l’appel `ajouterPoint(&total);` fait passer `total` à 5. La fonction reçoit une copie de l’adresse. En suivant cette adresse, elle atteint l’entier de l’appelant.

Que se passerait-il si tu écrivais `score = nullptr;` à la fin de la fonction ? Seule la copie locale du pointeur changerait. L’entier de l’appelant garderait la valeur 5.

`nullptr` représente l’absence de cible. Dans cet exemple, `ajouterPoint(nullptr);` est un appel prévu par la fonction : le test évite de déréférencer un pointeur nul. Pour représenter une sélection vide dans ton propre code, initialise le pointeur avec `int* selection = nullptr;`.

## 3. Accéder aux champs d’un joueur

```cpp
struct Joueur {
    int id;
    std::string nom;
    int score;
};

Joueur alice{1, "Alice", 12};
Joueur* joueur = &alice;
joueur->score += 3;
```

`alice.score` accède au score à partir de l’objet. `joueur->score` accède au même champ à partir de son adresse. Tu peux lire la flèche comme un raccourci pour `(*joueur).score`.

Pour afficher un joueur, le code a seulement besoin de lire ses champs.

```cpp
void afficherJoueur(const Joueur* joueur) {
    if (joueur != nullptr) {
        std::cout << joueur->nom << " : " << joueur->score << '\n';
    }
}
```

Le type `const Joueur*` autorise la lecture de l’objet à travers ce pointeur. Si tu essaies d’écrire `joueur->score += 3;` dans cette fonction, le compilateur signale une erreur.

## 4. Renvoyer l’adresse d’un élément du tournoi

Cette fonction cherche un joueur par identifiant et renvoie son adresse si elle le trouve.

```cpp
Joueur* trouverParId(std::vector<Joueur>& joueurs, int id) {
    for (Joueur& joueur : joueurs) {
        if (joueur.id == id) {
            return &joueur;
        }
    }
    return nullptr;
}
```

Le symbole `&` intervient à trois endroits.

| Écriture | Sens |
|---|---|
| `std::vector<Joueur>& joueurs` | Le paramètre est une **référence** au vecteur de l’appelant. |
| `Joueur& joueur` | À chaque tour, ce nom désigne directement un élément du vecteur. |
| `return &joueur;` | L’expression calcule et renvoie **l’adresse** de cet élément. |

Imagine que tu enlèves le `&` dans la boucle et écrives `for (Joueur joueur : joueurs)`. Chaque tour crée alors une copie locale. Renvoyer son adresse laisserait à l’appelant un **pointeur pendant**, car la copie est détruite en quittant la fonction. Vérifier que le pointeur est différent de `nullptr` ne garantit donc pas que sa cible existe encore.
