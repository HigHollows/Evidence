# Refonte de l'interface (workspace professionnel)

Suivi de la refonte complete de l'UI demandee pour rapprocher Evidence d'un
logiciel professionnel d'analyse (philosophie IDA Pro / Ghidra / x64dbg — pas
une copie de leur interface), livree par etapes.

## Etape 1 — Shell workspace (fait)

- Sidebar verticale permanente supprimee. Nouveau shell base sur
  **AvalonDock** (`Dirkster.AvalonDock`, MIT, dependance UI pure) avec
  panneaux dockables, redimensionnables, masquables :
  - **Explorateur** (gauche) : arborescence de l'analyse (Resume, Preuves ->
    Comportement/Reseau/Code/Fichiers, IOC, Timeline, MITRE, Historique),
    remplace l'ancienne `ListBox` de navigation.
  - **Inspecteur** (droite) : panneau contextuel — affiche les details de la
    ligne selectionnee (Preuves, imports notables, correspondances
    YARA/Sigma, IOC/Reseau) avec un bouton "Ouvrir la source" quand
    pertinent.
  - **Console** (bas) : onglets Journal (log applicatif horodate) et
    Evenements (Timeline), masquable.
  - **Onglets de documents** : "Accueil" (page d'accueil simple, analyses
    recentes) + un onglet d'analyse dynamique renomme au nom du fichier
    charge.
- Menu restructure : Fichier / Analyse / Vue / Recherche / Outils / Fenetre /
  Aide. Vue permet de reafficher/masquer Explorateur/Inspecteur/Console et de
  naviguer vers chaque vue. Fenetre > "Disposition par defaut" restaure les
  tailles de panneaux.
- Raccourcis : Ctrl+O (ouvrir), Ctrl+N (nouvelle analyse), F5 (actualiser).
- Style : police a chasse fixe pour les donnees techniques (hashes, offsets,
  chaines), boutons/onglets/DataGrid densifies, palette sombre existante
  conservee et etendue (pas de nouvelle charte, juste plus dense).
- Toute la logique metier existante (chargement de fichier, providers, YARA/
  Sigma, correlation, rapports, quarantaine...) est **inchangee** : seule la
  facon dont les vues sont presentees/naviguees a change.
- Verifie : build Release + 121/121 tests passent, lancement sans exception,
  et verification structurelle via UI Automation (arborescence des elements,
  pas de capture d'ecran — voir contrainte de confidentialite du projet) :
  panneaux presents avec des dimensions non nulles, page d'accueil active au
  demarrage, navigation reelle depuis l'explorateur bascule correctement vers
  le document d'analyse.

## Etape 2 — MITRE en arbre + timeline visuelle (fait)

- **MITRE ATT&CK** : la liste de cartes est remplacee par un `TreeView`
  (technique -> preuves, section 13 du cahier des charges). Selectionner une
  technique ou une preuve l'affiche dans l'Inspecteur.
- **Timeline** : ajout d'une vraie frise horizontale au-dessus du tableau
  chronologique (un point par evenement, positionne dans l'ordre, avec un
  slider de zoom qui espace/resserre les points et un `ScrollViewer`
  horizontal pour le deplacement). Cliquer un point ou une ligne du tableau
  affiche le detail dans l'Inspecteur. Le tableau chronologique existant est
  conserve en dessous (donnees exactes, triables) — la frise est une vue
  complementaire, pas un remplacement.
- **Vue Reseau** (ajoute apres coup, meme etape) : disposition a deux
  volets plutot que les trois colonnes de la maquette (Processus /
  Connexions / Details). A gauche, un arbre **fichier analyse -> type
  d'indicateur (Domaine/URL/IP) -> valeur** filtre le tableau a droite ;
  l'Inspecteur (dock droit) affiche le detail au clic. Le noeud racine est le
  fichier reellement analyse (fait observe), pas un "processus" invente :
  fabriquer une colonne "Processus" avec une arborescence d'evenements
  reseau donnerait l'illusion d'une capacite de telemetrie comportementale
  que ce projet n'a pas dans cette phase (uniquement des IOC extraits de
  chaines statiques, deja marques "POTENTIEL, pas OBSERVE"). Ce compromis
  garde l'esprit de la maquette (navigation par arbre + details) sans rien
  affirmer de faux.

## Etapes suivantes (pas encore commencees)

- Theme AvalonDock plus pousse (couleurs des onglets/poignees de
  redimensionnement alignees pixel-pres sur la palette du projet).
- Sauvegarde/restauration de la disposition des panneaux (serialisation
  AvalonDock) et dispositions nommees (Analyse / Reverse Engineering /
  Forensic / Reseau / Rapport).
- Vraie prise en charge multi-onglets (plusieurs fichiers analyses en
  parallele) — actuellement un seul onglet d'analyse actif a la fois,
  renomme a chaque nouveau fichier charge ; l'etat interne (`_currentFile`
  et champs associes) reste un etat unique partage, pas encore par-onglet.
- Recherche globale (Ctrl+Shift+F).

## Limite assumee

Aucune vue de desassemblage x86/x64 n'a ete ajoutee : ce projet n'embarque pas
de desassembleur (voir philosophie section 15 — ne jamais fabriquer une
capacite qui n'existe pas reellement). La vue "Code" reste l'analyse statique
PE deja existante (headers, sections, imports/exports, chaines, YARA/Sigma),
pas un veritable desassemblage instruction par instruction.
