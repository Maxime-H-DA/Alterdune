# Alterdune

Console role-playing game developed in C++ as part of an Object-Oriented Programming project.

The goal was to build a real turn-based combat system backed by a clean architecture (inheritance, abstract classes, polymorphism), rather than a script with `if` statements everywhere.

## Architecture

Player and Monster both inherit from Entity, which holds the base attributes (name, HP, attack, defense) and declares `attack()` as pure virtual. Neither Entity nor Monster can be instantiated directly.

Monster is then split into 3 categories, NormalMonster, MiniBoss and Boss, which inherit from Monster and each override `attack()` and `getMaxActions()` with their own behavior (2, 3 or 4 available actions, different damage ranges).

Monsters are stored in a `vector<Monster*>`. When the GameManager calls `enemy->attack(player)`, the vtable decides which version gets executed: there is no `switch` on the category in the combat logic. The same `attack()` call therefore produces radically different results depending on the entity's actual type.

Each damage value is drawn from a range using a Mersenne Twister generator (`mt19937`) rather than `rand()`, for better statistical quality, with a single seed shared by all entities (`static`). The roll happens on every attack, so two fights against the same enemy never play out exactly the same way.

## Combat and Progression

Combat is built around 4 actions: FIGHT (damage), ACT (a catalog of 8 text-based actions, 2 of which have a negative effect), ITEM (healing) and MERCY (spare the enemy).

Each monster has a `mercyGauge` bounded between 0 and its `mercyGoal`, and MERCY is only available once the gauge is full. The player earns different amounts of XP depending on the enemy defeated (+2 / +3 / +5) and levels up automatically.

Difficulty increases over time: MiniBosses only appear after 3 fights, and Bosses only from 7 victories onward. The game ends with one of 3 possible endings (Pacifist, Neutral, Genocide) depending on the player's style.

## Data

Monsters and items are loaded from CSV files (`monsters.csv`, `items.csv`) using `ifstream` and `stringstream`, with malformed lines handled through `try/catch`. The provided file contains 45 unique monsters (26 normal, 13 mini-bosses, 6 bosses).

Each fight is recorded in a log (`history.txt`) through a lightweight `BestiaryEntry` structure, rather than by juggling several parallel lists.

## Tests

4 unit tests run automatically at startup (`runUnitTests`) before the main menu: damage taken, the Mercy system, level up, and resetting the player to a clean state.

## Tools Used

C++, STL (`vector`, `map`), `<random>` (Mersenne Twister), `ifstream` / `stringstream`, OOP (inheritance, abstract classes, polymorphism)
