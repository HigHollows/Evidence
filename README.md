<p align="center"><img src="assets/banner.png" alt="Evidence" width="480" /></p>

[![build](https://github.com/HigHollows/Evidence/actions/workflows/build.yml/badge.svg)](https://github.com/HigHollows/Evidence/actions/workflows/build.yml)

# Malware Analyzer

Outil Windows d'**investigation** de fichiers potentiellement malveillants — pas un
antivirus. Il collecte des preuves techniques depuis plusieurs sources (analyse
statique, VirusTotal, sandboxes cloud, YARA/Sigma, MITRE ATT&CK), les corrèle, puis
produit un rapport traçable jusqu'aux données brutes. Une IA optionnelle peut
expliquer le raisonnement, mais **le logiciel fonctionne intégralement sans elle**.

## Principe fondamental

Chaque affirmation est catégorisée : `OBSERVÉ`, `CORROBORÉ`, `PROBABLE`, `POSSIBLE`,
`NON OBSERVÉ`, `INCONNU`, `INCONCLUSIF`. Une capacité potentielle (une API trouvée
dans le binaire) n'est jamais confondue avec un comportement observé. L'absence de
preuve n'est jamais présentée comme une preuve d'absence. Aucun verdict n'est forcé :
un fichier peut être `INCONCLUSIF` si les preuves ne suffisent pas à conclure.

## État du projet

Développement par phases, voir [docs/ROADMAP.md](docs/ROADMAP.md).

- **Phase 1 (fait)** — application Windows, sélection de fichier (glisser-déposer ou
  dialogue), calcul de hash (SHA-256/SHA-1/MD5), détection de type par signature
  binaire (pas par extension), historique local SQLite.
- **Phase 2 (fait)** — analyse statique PE (EXE/DLL/SYS) : headers, sections et
  leur entropie, imports/exports, table d'imports "notables" catégorisée en
  **capacités potentielles** (jamais des preuves de comportement), chaînes
  ASCII/UTF-16 avec recherche, détection d'anomalies structurelles (empaquetage
  possible, sections exécutables+inscriptibles, etc.), présence et signataire
  Authenticode (sans validation de chaîne de confiance dans cette phase).
- **Phase 3 (fait)** — provider VirusTotal : clé API personnelle stockée
  chiffrée (DPAPI) localement, jamais journalisée ni transmise à l'IA ; consultation
  par hash uniquement (pas d'upload du fichier) avec consentement explicite avant
  chaque requête ; mise en cache locale (24h) pour éviter les requêtes inutiles ;
  gestion des erreurs (clé invalide, quota atteint, service injoignable) sans
  bloquer le reste de l'analyse ; fenêtre **Outils > Fournisseurs** pour
  ajouter/tester/supprimer la clé. Architecture `IExternalProvider` posée pour
  brancher les futurs providers (sandbox, réputation, IA...) de la même façon.
- **Phase 4 (fait)** — provider **Hybrid Analysis (Falcon Sandbox)** : recherche
  de rapports sandbox *publics déjà existants* pour un hash (`GET /search/hash`,
  clé personnelle) — cette phase ne soumet jamais le fichier pour une nouvelle
  exécution et ne déclenche donc aucune analyse dynamique automatique. Résultat
  affiché avec verdict, score, famille signalée et environnement, toujours
  accompagné du rappel qu'un rapport d'un autre utilisateur ne constitue pas une
  preuve pour ce dossier. Nouvel onglet **Comportement** dans l'UI.
- **Phase 5 (fait)** — extraction d'**IOC** (hash du fichier en `OBSERVÉ`,
  URL/IP/domaines/chemins/clés de registre/emails extraits des chaînes statiques
  en `POSSIBLE`), export TXT/CSV/JSON ; onglet **Réseau** présentant ces mêmes
  indicateurs réseau avec un bandeau explicite "POTENTIEL, pas OBSERVÉ" (ce ne
  sont pas des connexions capturées pendant une exécution) ; onglet **Timeline**
  retraçant les actions de *cet outil* (hash calculé, requêtes providers
  envoyées), explicitement distinguée d'une timeline d'événements d'exécution
  que cette phase ne produit pas encore.
- **Phase 6 (fait)** — moteur **YARA** (sous-ensemble maison documenté : pas
  de regex ni de modules, pour rester fiable sans dépendance native non
  vérifiable) avec règles intégrées d'exemple et règles personnalisées
  (`%LOCALAPPDATA%\MalwareAnalyzer\yara_rules\*.yar`), activables/désactivables
  et ré-analysables depuis l'onglet **Code > YARA** ; moteur **Sigma** (format
  simplifié propre au projet, pas du YAML Sigma standard) évaluant les
  observations déjà disponibles (imports notables, verdicts sandbox publics,
  réputation VirusTotal) depuis l'onglet **Code > Sigma** ; onglet **MITRE**
  associant chaque technique à des preuves précises (import notable ou règle
  YARA déclenchée) — jamais une technique attribuée sans preuve directe.
- **Phase 7 (fait)** — moteur de **corrélation** : agrège toutes les preuves
  déjà collectées (comportement sandbox, YARA, imports notables, Sigma,
  réputation VirusTotal, IOC réseau) avec une hiérarchie de poids explicite
  (comportement observé > capacité statique > réputation) et un identifiant
  traçable par preuve (`BEHAVIOR-x`, `CODE-x`, `NET-x`, `IOC-x`, `REP-x`) ;
  calcule une **couverture d'analyse** qui force `INCONCLUSIF` si trop de
  modules manquent ; détecte les **contradictions entre sources** avec leurs
  explications possibles ; propose des hypothèses de classification (Stealer,
  Backdoor/RAT, Spyware/Keylogger, Downloader) toujours accompagnées
  d'arguments POUR et CONTRE, jamais une certitude forcée. Nouvel onglet
  **Preuves** (double-clic pour remonter à la source), verdict dynamique dans
  **Résumé**.
- **Phase 8 (fait)** — assistant **IA optionnel**, compatible avec toute API
  "chat completions" au format OpenAI (endpoint/modèle/clé configurables,
  fonctionne avec un serveur local). Reçoit uniquement un contexte JSON compact
  (verdict, preuves, hypothèses, contradictions, couverture) — jamais le fichier
  complet, jamais une clé API. Prompt système strict (20 règles, section 53) :
  interdiction d'inventer une preuve/API/domaine, obligation de citer des
  identifiants réels, de signaler les limites, de fournir les arguments contre
  son propre verdict. Un garde-fou automatique signale toute citation d'un
  identifiant de preuve absent du contexte fourni. Bulle discrète en bas de
  fenêtre ; toutes les analyses techniques continuent de fonctionner sans elle.
- **Phase 9 (en cours)** — **rapports HTML/PDF/JSON** (menu Fichier), générés
  uniquement à partir des données déjà collectées : verdict en tête, détails
  progressifs, chaque preuve avec son identifiant. Le PDF est écrit directement
  (police à chasse fixe standard, pagination automatique), sans bibliothèque
  tierce non vérifiable. **Analyse d'archives ZIP** récursive (onglet
  **Fichiers**) avec protections contre les archives excessives (profondeur,
  nombre de fichiers, taille cumulée, taux de compression suspect) ; RAR/7Z
  sont détectés mais pas encore décompressés (nécessiteraient une dépendance
  tierce hors du contrôle de ce projet).

Aucun provider de réputation supplémentaire n'est encore branché : la fenêtre
l'indique explicitement (`INCONCLUSIF`) plutôt que d'afficher un faux résultat.

## Stack technique

- **.NET 8 / C# / WPF** — stabilité, intégration native Windows (Credential
  Manager/DPAPI pour les futurs secrets, associations de fichiers, menu contextuel),
  packaging `.exe` simple, écosystème mature pour l'analyse PE et le parsing binaire.
- **SQLite** (`Microsoft.Data.Sqlite`) pour l'historique local et le cache providers.
- **DPAPI** (`System.Security.Cryptography.ProtectedData`) pour chiffrer les clés API localement.
- **xUnit** pour les tests.

## Structure du dépôt

```text
src/
  MalwareAnalyzer.App/    Application WPF (UI)
  MalwareAnalyzer.Core/   Logique métier (hash, historique, modèles)
tests/
  MalwareAnalyzer.Tests/  Tests unitaires (xUnit)
docs/                     Documentation (roadmap, notes)
rules/ yara/ sigma/       Règles de détection (Phase 6)
locales/                  Chaînes UI externalisées (fr/en, section 48)
assets/                   Ressources graphiques
scripts/                  Scripts de build/dev
installer/                Installateur Windows (phase ultérieure)
```

## Compilation

Nécessite le **SDK .NET 8** (pas seulement le runtime) : https://dotnet.microsoft.com/download/dotnet/8.0

```bash
dotnet build src/MalwareAnalyzer.sln
dotnet test src/MalwareAnalyzer.sln
dotnet run --project src/MalwareAnalyzer.App
```

## Clés API et confidentialité

Aucune clé API n'est fournie avec l'application. Chaque utilisateur configure ses
propres clés (VirusTotal, sandbox, IA) dans les paramètres, à partir de la Phase 3.
Les secrets ne sont jamais écrits dans les logs, les rapports, ni envoyés à l'IA.
Rien n'est envoyé vers un service externe sans confirmation explicite.

## Coût

Objectif 0 € : uniquement des services avec offre gratuite, sous clés personnelles de
l'utilisateur. Aucune infrastructure serveur ni abonnement n'est requis pour utiliser
le logiciel.

## Licence

[MIT](LICENSE).
