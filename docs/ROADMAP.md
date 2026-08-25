# Feuille de route

Developpement par phases (voir cahier des charges complet dans les notes du projet).
Chaque phase doit fonctionner correctement avant de passer a la suivante.

| Phase | Contenu | Statut |
|---|---|---|
| 1 | Application Windows + UI + selection fichier + hash + historique local | Fait |
| 2 | Analyse statique PE (headers, sections, imports/exports, strings, entropie) | Fait |
| 3 | VirusTotal + framework multi-provider | Fait |
| 4 | Sandboxes cloud (analyse comportementale) | Fait |
| 5 | IOC + analyse reseau + timeline | Fait |
| 6 | YARA + Sigma + MITRE ATT&CK | Fait |
| 7 | Moteur de correlation | Fait |
| 8 | Assistant IA (optionnel) | Fait |
| 9 | Rapports HTML/PDF/JSON + analyse d'archives ZIP | En cours |
| 10 | Quarantaine + historique avance + comparaison d'echantillons | A venir |
| 11 | Tests de securite et de robustesse | A venir |

## Regle de non-regression fonctionnelle

Une phase livree ne doit jamais etre cassee par une phase suivante. Les nouvelles
tables SQLite sont additives (jamais de suppression de colonnes existantes sans
migration explicite documentee).

## Principe directeur

Le logiciel collecte des preuves techniques categorisees (OBSERVE / CORROBORE /
PROBABLE / POSSIBLE / NON OBSERVE / INCONNU / INCONCLUSIF). Il ne prononce jamais
un verdict qui depasse ce que les preuves disponibles permettent d'affirmer.
