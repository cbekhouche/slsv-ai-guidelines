# Bots

## Vue d’ensemble

Le répertoire **Bots** contient un ensemble d’**assistants interactifs, pas à pas** utilisés dans les outils SousLeSensVocables pour guider les utilisateurs à travers des opérations en plusieurs étapes, telles que :  
- Création de ressources d’ontologie  
- Construction de restrictions OWL  
- Création de mappings  
- Exploration et composition de graphes  
- Configuration d’actions orientées utilisateur (ex. : partage de données utilisateur, recherche de nœuds similaires)

Les bots sont généralement implémentés sous forme de **workflows déclaratifs** exécutés par un moteur d’UI partagé. Un bot définit le plus souvent :  
- Un objet `workflow` (avec branchement via `_OR` / `_DEFAULT`)  
- Une table `functions` indexée par les noms des étapes du workflow  
- Optionnellement des `functionTitles` pour les invites destinées à l’utilisateur  
- Un point d’entrée `start(...)` qui initialise le moteur et démarre le workflow

La plupart des bots instancient `BotEngineClass`, mais certains bots hérités s’appuient sur un global partagé `_botEngine` et/ou réutilisent des ensembles de fonctions d’autres bots (par exemple `SparqlQuery_bot.functions`).

Le moteur sous-jacent fournit des primitives UI réutilisables (sélection dans une liste, saisie de valeur, navigation dans l’historique), ce qui permet aux bots de se concentrer sur la logique métier, par exemple :  
- Opérations SPARQL  
- Actions sur les graphes de lignage  
- Accès au modèle d’ontologie  
- Persistance de la configuration de mapping  
- Parcours de partage de données utilisateur  
- Interrogation SPARQL interactive avec plusieurs modes de sortie (graphe/table/CSV)

---

## Modules

### Moteur (exécuteur de workflow « core »)

1. **BotEngineClass** (`_botEngineClass.js`) — Moteur principal d’UI : charge le template d’UI du bot, exécute les étapes du workflow, gère les branchements (`_OR`) et maintient l’historique de navigation (précédent/réinitialisation).

### Utilitaires partagés & aides de workflow

2. **CommonBotFunctions** (`_commonBotFunctions.js`) — Aides partagées : charge les modèles d’ontologie pour une source et ses imports, et produit des listes triées de vocabulaires, classes, propriétés et propriétés non-object pour la sélection dans l’UI des bots.
3. **NonObjectPropertyFilterWorklow** (`_nonObjectPropertyFilterWorklow.js`) — Workflow d’aide pour construire des fragments de filtre SPARQL sur des propriétés de type datatype / non-object en sélectionnant propriété / opérateur / valeur, puis en chaînant des conditions avec AND/OR/fin.

### Création de ressources & modélisation OWL

4. **CreateResource_bot** (`createResource_bot.js`) — Création générique de ressources (Class, Individual, DatatypeProperty) avec post-actions optionnelles (éditer ou dessiner dans le whiteboard de lignage).
5. **CreateAxiomResource_bot** (`createAxiomResource_bot.js`) — Création ciblée de ressources d’axiomes : crée des `owl:Class` (avec `subClassOf` optionnel) et des sous-propriétés, et renvoie un `params.newObject` normalisé.
6. **CreateRestriction_bot** (`createRestriction_bot.js`) — Crée des restrictions OWL : écrit les restrictions de cardinalité sous forme de nœuds anonymes (blank nodes) et délègue la création des restrictions de valeur à une boîte de dialogue d’arête de lignage.

### Onboarding de source

7. **CreateSLSVsource_bot** (`createSLSVsource_bot.js`) — Bot « OntoCreator » pour créer une nouvelle source, téléverser éventuellement un graphe (fichier/URL), ajouter des métadonnées d’ontologie (creator/description) et rediriger vers les outils (Lineage/MappingModeler).

### Exploration & navigation de graphes (Lineage / Whiteboard)

8. **GraphPaths_bot** (`graphPaths_bot.js`) — Exploration de graphes : calcule et dessine des chemins depuis/vers/entre des nœuds avec sortie en texte/CSV ou surlignage dans le graphe.
9. **NodeRelations_bot** (`nodeRelations_bot.js`) — Assistant interactif « query graph » depuis le nœud courant du whiteboard de lignage : explore les object properties, les propriétés d’annotation/datatype (avec filtrage par valeur), les restrictions (y compris restrictions inverses), l’appartenance à des conteneurs, et des utilitaires de lignage supplémentaires (ex. : dialogue de similitude, visualisations de parcours).
10. **Similars_bot** (`similars_bot.js`) — Assistant interactif pour rechercher des nœuds « similaires » soit dans la sélection du whiteboard, soit dans une source, avec correspondance exacte/floue, filtrage optionnel des résultats (par parent) et plusieurs options de sortie (graphe/table/enregistrement).

### Assistance au mapping (KGcreator)

11. **KGcreator_bot** (`KGcreator_bot.js`) — Assistant de mapping en contexte KGcreator : aide à définir des modèles de triplets de mapping (typage d’URI, `rdf:type`, prédicats, propriétés non-object, jointures) et persiste la configuration de mapping.

### Assistants KGQuery

12. **KGquery_annotations_bot** (`KGquery_annotations_bot.js`) — Assistant pour configurer une requête SPARQL de sélection (variables `SELECT`) et des filtres en réutilisant `SparqlQuery_bot.functions`, puis renvoie `filter` et `filterLabel` via le callback de validation.
13. **KGquery_filter_bot** (`KGquery_filter_bot.js`) — Constructeur de filtres interactif : sélectionne une propriété (non-object), un opérateur et une valeur (y compris plage de dates), puis construit une clause SPARQL `FILTER(...)` avec chaînage logique optionnel.
14. **KGquery_composite_graph_bot** (`KGquery_composite_graph_bot.js`) — Bot « HyperGraphMaker » : importe des graphes KGquery depuis une source et ses imports, télécharge/régénère des graphes, colore les graphes importés, les fusionne, et peut joindre des graphes en sélectionnant une classe commune.
15. **SparqlQuery_bot** (`sparqlQuery_bot.js`) — Assistant SPARQL interactif pour interroger des graphes et afficher les résultats sous forme de nouveau graphe, ajouter au graphe courant, exporter en table/CSV, ou éditer/exécuter du SPARQL brut (y compris requêtes sauvegardées et rappel de la « dernière requête »).

### Aides MappingModeler (mappings techniques / lookups / prédicats supplémentaires)

16. **MappingModeler_bot** (`mappingModeler_bot.js`) — Assistant Mapping Modeler pour appliquer des actions de mapping technique telles que l’ajout de prédicats (y compris propriétés non-object), l’ajout de `rdf:type`, l’ajout de `rdfs:subClassOf`, l’ajout de transformations et la création de datatype properties via `CreateResource_bot`.
17. **Lookups_bot** (`mappingModeler_lookups_bot.js`) — Assistant (wizard) pour créer et enregistrer des configurations de « lookup » : sélection de datasource/table/colonnes/mapping cible, écriture du lookup dans la config, puis sauvegarde du graphe de mapping visjs.

### Données utilisateur & partage

18. **ShareUserData_bot** (`shareUserData_bot.js`) — Assistant pour partager un élément UserData avec des profils et/ou des utilisateurs, lister les partages existants et retirer des profils/utilisateurs partagés ; persiste les mises à jour via patch et sauvegarde des métadonnées UserData.

### Widgets UI utilisés par les bots

19. **ManchesterSyntaxWidget** (`manchesterSyntaxWidget.js`) — Widget de saisie basé sur des jetons (tokens) qui suggère des classes/propriétés d’ontologie (depuis la source + imports + vocabulaires de base) et construit une expression de type Manchester à partir de tokens validés.

---

## Fonctionnalités

- **Moteur de workflow réutilisable** avec des primitives UI cohérentes (sélection dans une liste, invites, navigation précédent/réinitialisation, branchement via `_OR`).
- **Aides de sélection sensibles à l’ontologie** sur une source et ses imports (vocabulaires/classes/propriétés/propriétés non-object).
- **Persistance adossée à SPARQL** et manipulation du modèle via des proxys SPARQL et des services de lignage (création de ressources, restrictions, métadonnées, graphes de prédicats).
- **Intégration Graphe/Lineage** : dessiner les ressources créées, calculer et surligner des chemins, explorer relations/restrictions de nœuds, visualiser les parcours.
- **Workflows de recherche de similarité** : correspondance exacte/floue sur les nœuds, filtrage optionnel des résultats et plusieurs options d’affichage/export.
- **Workflows de partage** pour les données utilisateur : partager avec profils/utilisateurs, lister et révoquer les partages, persister les mises à jour de métadonnées.
- **Support KGQuery** : construction interactive de filtres, composition de « hypergraphes » et bot SPARQL généraliste avec plusieurs modes de sortie (graphe/table/CSV) incluant édition de requête et réutilisation de requêtes sauvegardées.
- **Support MappingModeler** : opérations guidées de mapping technique (prédicats, `rdf:type`, subclass, transforms, lookups) et création de datatype properties via le bot CreateResource partagé.

---

## Utilisation

- Choisir le module de bot approprié et appeler son point d’entrée `start(...)` (les signatures varient : certains bots instancient `BotEngineClass` localement, d’autres réutilisent un moteur partagé/global).
- Définir ou réutiliser un objet de workflow pour représenter la chaîne d’étapes et la logique de branchement (`_OR`, `_DEFAULT`).
- Implémenter les étapes du workflow dans la table `functions` du bot et fournir des titres destinés à l’utilisateur via `functionTitles` pour des invites cohérentes.
- Utiliser les utilitaires partagés (`CommonBotFunctions`) pour charger les modèles d’ontologie et lister les éléments sélectionnables de façon uniforme ; utiliser les aides spécialisées (ex. : `NonObjectPropertyFilterWorklow`) pour générer des fragments de filtre cohérents.
- Persister les résultats via les services SPARQL/Lineage/MappingModeler/UserData (insertion de triplets, mise à jour de métadonnées, sauvegarde de graphe de mapping) ; selon le bot, les sorties sont renvoyées via un validateur/callback, stockées dans `params` (ex. : `params.newObject`) et/ou produites en effets de bord UI (dialogues, rendu de graphe, configurations sauvegardées).
