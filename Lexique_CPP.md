# Lexique des mots-clés importants en C++17 et versions suivantes

Les trois premières rubriques s’utilisent dès C++17. La dernière regroupe quelques ajouts de C++20 et C++23.

## Variables, constantes et types

| Mot ou expression | Sens | Exemple |
|---|---|---|
| `auto` | Le compilateur déduit le type. Avec `auto&`, on obtient une référence à l’objet d’origine. | `auto n = 42;` · `auto& joueur = equipe[0];` |
| `const` | Empêche de modifier une valeur par ce nom. Sa valeur initiale peut être calculée à l’exécution. | `const int score = lireScore();` |
| `constexpr` | Sur une variable, exige une valeur calculable à la compilation. Sur une fonction, permet son évaluation à la compilation. | `constexpr int carre(int x) { return x * x; }` |
| `static` | Une variable locale conserve sa valeur entre les appels. Un membre `static` appartient à la classe et une méthode `static` s’appelle sans objet. Au niveau d’un fichier, le nom reste propre à l’unité de compilation. | `int prochainId() { static int id = 0; return ++id; }` renvoie `1`, puis `2`, puis `3`… |
| `extern` | Permet de déclarer une variable dont la définition se trouve ailleurs. | `extern int volume;` |
| `thread_local` | Chaque thread possède sa propre instance de la variable. | `thread_local int compteur = 0;` |
| `decltype` | Récupère le type d’un nom ou d’une expression. | `int score = 0; decltype(score) record = 10;` |
| `using` | Crée notamment un alias de type. | `using Identifiant = unsigned int;` |
| `enum class` | Définit un type avec des valeurs nommées, accessibles avec `::`. | `enum class Etat { pret, actif };` puis `Etat::pret` |
| `nullptr` | Représente un pointeur qui ne désigne aucun objet. | `Joueur* cible = nullptr;` |

## Fonctions, classes et objets

| Mot ou expression | Sens | Exemple |
|---|---|---|
| `class`, `struct` | Définissent un type. Les membres sont privés par défaut avec `class` et publics avec `struct`. | `struct Point { int x, y; };` |
| `public`, `protected`, `private` | Règlent l’accès aux membres. `public` ouvre l’accès à tous, `protected` l’ouvre aussi aux classes dérivées, `private` le réserve à la classe et à ses amis. | `private: int solde = 0;` |
| `this` | Pointeur vers l’objet courant dans une fonction membre. | `this->score = score;` |
| `const` après `()` | Permet d’appeler une méthode sur un objet constant. La méthode préserve les attributs ordinaires de cet objet. | `int score() const;` |
| `inline` | Permet de placer une même définition de fonction ou, depuis C++17, de variable dans un en-tête inclus par plusieurs `.cpp`. Le compilateur garde la main sur l’optimisation des appels. | `inline int doubler(int x) { return 2 * x; }` |
| `explicit` | Sur un constructeur, empêche une conversion implicite vers le type de la classe. | Avec `explicit Duree(int);`, on écrit `Duree d{5};`. |
| `virtual` | Lors d’un appel via une référence ou un pointeur de base, la méthode exécutée dépend du type réel de l’objet. | `virtual void dessiner();` |
| `override` | Demande au compilateur de vérifier qu’une méthode redéfinit bien une méthode virtuelle héritée. | `void dessiner() override;` |
| `final` | Empêche de dériver d’une classe ou de redéfinir à nouveau une méthode virtuelle. | `class Cercle final {};` |
| `= default` | Demande au compilateur de générer une fonction spéciale, comme un constructeur. | `Point() = default;` |
| `= delete` | Interdit l’utilisation d’une fonction. Ici, cela interdit la construction par copie. | `Point(const Point&) = delete;` |
| `noexcept` | Indique qu’aucune exception ne doit sortir de la fonction. Si une exception en sort, le programme appelle `std::terminate`. | `void fermer() noexcept;` |

## Code générique, conversions et erreurs

| Mot ou expression | Sens | Exemple |
|---|---|---|
| `template`, `typename` | Définissent du code générique. Dans l’exemple, `T` représente le type sur lequel la fonction travaille. | `template<typename T> T carre(T x) { return x * x; }` |
| `if constexpr` | Choisit une branche à la compilation. Utile quand le traitement dépend d’un paramètre de `template`. | `if constexpr (std::is_integral_v<T>)` |
| `static_assert` | Vérifie une condition à la compilation. Si elle est fausse, la compilation échoue. | `static_assert(sizeof(int) >= 4);` |
| `static_cast` | Effectue explicitement une conversion autorisée par le langage. | `static_cast<double>(total) / nombre` permet une division en virgule flottante. |
| `try`, `catch`, `throw` | `throw` lance une exception. `try` délimite le code surveillé et `catch` traite une exception correspondante. | `throw std::runtime_error("Fichier introuvable");` |
| `namespace` | Regroupe des noms pour éviter les collisions. | `namespace jeu { struct Point {}; }` permet d’écrire `jeu::Point`. |

## Ajouts en C++20 et C++23

| Mot ou expression | Version | Sens |
|---|---|---|
| `concept`, `requires` | C++20 | Expriment les conditions qu’un type doit satisfaire pour être accepté par du code générique. |
| `consteval` | C++20 | Déclare une fonction dont les appels directs ordinaires doivent être évalués à la compilation. |
| `constinit` | C++20 | Garantit l’initialisation statique d’une variable globale, `static` ou `thread_local`. La variable peut ensuite être modifiée. |
| `co_await`, `co_yield`, `co_return` | C++20 | S’emploient dans les coroutines pour attendre un résultat, produire une valeur intermédiaire ou terminer la coroutine. |
| `if consteval` | C++23 | Permet à une fonction de choisir un traitement selon qu’elle est évaluée à la compilation ou à l’exécution. |
