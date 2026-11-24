# Les opérateurs binaires en C++

**Niveau débutant** · Prérequis : variables, conditions et fonctions.

Un opérateur est **binaire** s’il prend deux opérandes. Dans `a + b`, les opérandes sont `a` et `b`. Les opérateurs **bit à bit**, comme `a & b`, travaillent sur les bits d’entiers. La plupart prennent deux opérandes, tandis que `~x` est unaire.

## 1. Les principales familles

| Famille | Opérateurs courants | Exemple |
| --- | --- | --- |
| Arithmétique | `+`, `-`, `*`, `/`, `%` | `7 / 2` vaut `3` entre entiers. `7 % 2` vaut `1`. |
| Comparaison | `==`, `!=`, `<`, `<=`, `>`, `>=` | `7 >= 2` vaut `true`. |
| Logique | `&&`, `\|\|` | `x >= 0 && x < 10` vérifie les deux bornes. |
| Affectation | `=`, `+=`, `-=`, etc. | `x += 3` ajoute 3 à `x`. |
| Bit à bit | `&`, `\|`, `^` | `6 & 3` vaut `2`. |
| Décalage de bits | `<<`, `>>` | `6u >> 1` vaut `3u`. |

Les opérateurs logiques intégrés utilisent le **court-circuit**. Dans `n != 0 && total / n > 2`, la division est évaluée uniquement si `n != 0`. Avec `||`, la partie droite est évaluée uniquement si la partie gauche est fausse. Fais aussi attention à `=` (affecter) et `==` (comparer).

### Du code au résultat

- Avec `7 / 2`, les deux opérandes sont des entiers. Le quotient est tronqué vers zéro. Avec `7.0 / 2`, le calcul se fait en virgule flottante et donne `3.5`.
- Le reste permet de décomposer un quotient entier. Par exemple, `17 / 5` donne 3 et `17 % 5` donne 2, car 17 = 3 × 5 + 2.
- Dans `6 && 3`, les deux entiers non nuls sont convertis en `true`. Le résultat est donc `true`. Dans `6 & 3`, le calcul se fait sur les bits et le résultat est l’entier `2`.
- Les priorités déterminent comment les opérations sont regroupées. `2 + 3 * 4` vaut 14, alors que `(2 + 3) * 4` vaut 20.

## 2. Lire un entier en binaire

Un entier écrit en base 2 utilise les chiffres 0 et 1. De droite à gauche, les positions sont numérotées à partir de 0 et pèsent 1, 2, 4, 8, etc.

### Décomposer `0b0110u`

| Partie | Signification |
| --- | --- |
| `0b` | Préfixe qui indique la base 2. Cette notation est disponible depuis C++14. |
| `0110` | Chiffres binaires. De gauche à droite, leurs poids sont 8, 4, 2 et 1. Le zéro de tête aligne les bits dans les exemples et laisse la valeur inchangée. |
| `u` | Suffixe pour *unsigned* (« non signé »). Pour cette valeur, le type du littéral est `unsigned int`. |

Le calcul donne `0 × 8 + 1 × 4 + 1 × 2 + 0 × 1 = 6`. Les écritures `0b0110u` et `6u` désignent donc la même valeur et ont le même type. Le zéro placé après `0b` ne fixe pas la taille du type.

| A | B | ET `&` | OU `\|` | OU exclusif `^` |
| - | - | - | - | - |
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 |

Avec `a = 0b0110u` (6) et `b = 0b0011u` (3), applique la table à chaque colonne de bits.

| Expression | Derniers 4 bits | Valeur décimale |
| --- | --- | --- |
| `a & b` | `0010` | 2 |
| `a \| b` | `0111` | 7 |
| `a ^ b` | `0101` | 5 |

Par exemple, pour calculer `0110 & 0011`, pars de la droite. À la position 0, `0 & 1` donne 0. À la position 1, `1 & 1` donne 1. Aux positions 2 et 3, au moins un opérande vaut 0. En relisant de gauche à droite, on obtient donc `0010`. Seul le bit qui pèse 2 est actif.

Le complément `~` inverse tous les bits du type sur lequel l’opération est effectuée. Ainsi, `~0b0110u` se termine en `1001` et tous les bits plus à gauche valent 1. Pour conserver uniquement les quatre derniers bits, écris `(~0b0110u) & 0b1111u`, ce qui donne 9.

## 3. Décaler les bits

Sur un entier non signé, `x << n` décale les bits vers la gauche et ajoute des zéros à droite. Les bits qui débordent sont perdus. `x >> n` décale les bits vers la droite et ajoute des zéros à gauche.

- `0b0011u << 2` donne `0b1100u`, soit 12.
- `0b1101u >> 2` donne `0b0011u`, soit 3.
- Pour un `unsigned int`, choisis un décalage compris entre 0 inclus et `std::numeric_limits<unsigned int>::digits` exclu (en-tête `<limits>`). Un décalage négatif ou trop grand entraîne un comportement indéfini.

Un décalage à gauche de deux positions multiplie par 4 tant qu’aucun bit significatif ne déborde. Un décalage à droite de deux positions sur un entier non signé correspond à une division entière par 4. Avec 13, les deux bits de droite disparaissent et le résultat vaut 3.

Dans `std::cout << x`, le symbole `<<` sert à afficher `x`. Son rôle dépend du type des opérandes.

## 4. Modifier et tester des bits avec un masque

Un **masque** indique les positions à manipuler. Par exemple, `1u << 2` produit `0b0100u` et sélectionne le bit de position 2.

| Action | Expression avec `x` et `masque` de type `unsigned int` |
| --- | --- |
| Mettre les bits sélectionnés à 1 | `x \|= masque` |
| Mettre les bits sélectionnés à 0 | `x &= ~masque` |
| Inverser les bits sélectionnés | `x ^= masque` |
| Tester si au moins un bit sélectionné vaut 1 | `(x & masque) != 0u` |
| Tester si tous les bits sélectionnés valent 1 | `(x & masque) == masque` |

Garde les parenthèses dans `(x & masque) != 0u`. Sans elles, `x & masque != 0u` se lit `x & (masque != 0u)`.

### Construire un masque de plusieurs bits

Pour sélectionner les positions 0 et 2, combine `(1u << 0) | (1u << 2)`. Le masque vaut `0101`. Si `x` vaut `0011`, alors `x & masque` vaut `0001`. Le test « au moins un » réussit, puisque `0001` est différent de zéro. Le test « tous » échoue, puisque `0001` diffère de `0101`.

Le OU exclusif permet de basculer un état. Deux applications du même masque rétablissent la valeur initiale. Par exemple, `0011 ^ 0101` donne `0110`, puis `0110 ^ 0101` redonne `0011`.

### Extraire plusieurs bits d’un entier

Supposons qu’un entier contienne une valeur sur les positions 2, 3 et 4. Avec `unsigned int mot = 0b10110100u;`, on peut lire cette valeur en deux étapes.

1. `mot >> 2` donne `0b00101101u`. Les positions 2 à 4 sont maintenant aux positions 0 à 2.
2. `(mot >> 2) & 0b111u` conserve ces trois positions et donne `0b101u`, soit 5.

### Exemple complet avec des permissions

```cpp
#include <bitset>
#include <iostream>

int main() {
    constexpr unsigned int lecture = 1u << 0;
    constexpr unsigned int ecriture = 1u << 1;
    constexpr unsigned int execution = 1u << 2;

    unsigned int droits = lecture | ecriture; // 011
    droits |= execution;                     // 111
    droits &= ~ecriture;                     // 101

    std::cout << std::bitset<3>(droits) << '\n';
    std::cout << std::boolalpha << ((droits & lecture) != 0u) << '\n';
}
```

Enregistre le bloc dans `main.cpp`, puis compile-le avec `g++ -std=c++14 -Wall -Wextra main.cpp -o demo`. En lançant `./demo`, tu obtiens `101`, puis `true`.

## 5. Exercices

1. Calcule `0b1010u & 0b1100u`, `0b1010u | 0b1100u` et `0b1010u ^ 0b1100u`.
2. À partir de `unsigned int x = 0b0101u;`, active le bit 1 puis désactive le bit 2. Donne la valeur finale en binaire.
3. Écris une fonction `est_impair` qui prend un `unsigned int` et teste son bit de poids faible.

4. Avec `x = 0b1010u` et `masque = 0b0110u`, calcule `x & masque`. Quels tests réussissent parmi « au moins un » et « tous » ?
5. Avec `mot = 0b11011010u`, extrais les deux bits qui commencent à la position 3, puis donne leur valeur décimale.
