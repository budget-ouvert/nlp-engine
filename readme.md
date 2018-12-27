Ce repo est utilisé pour réaliser deux tâches :
* formater des documents tirés de [data.gouv.fr](https://data.gouv.fr)
* permettre la comparaison de ces documents entre différentes années

Les documents générés sont exportés vers `./../server/resources`, dont le repo est accessible [ici](https://github.com/budget-ouvert/server).

L'ensemble des données utilisées pour ces deux tâches est stocké dans `./input_data`.
* Les données présentes dans `./input_data/nomenclatures_plf` ont été obtenues lors du Hackathon Datafin 2018. Elles contiennent les nomenclatures des PLFs 2012 à 2019, sans les crédits.
* les dossiers `./input_data/raw_data/{plf, plr}_{année}` contiennent des fichiers CSV tirés de [data.gouv.fr](https://data.gouv.fr).
* `./input_data/decoded_data` contient les mêmes dossiers que `./input_data/raw_data` avec un encoding différent (`ISO-8859-1` ou affilié pour `raw_data`, `UTF-8` pour `decoded_data`). Ce sont ces fichiers qui sont utilisés pour produire les fichiers finaux.

## Formatage

Le formatage est réalisé dans le notebook `./parsing_pipeline.ipynb`. L'objectif principal est de regrouper plusieurs documents sous un format facilement utilisable.

Le format choisi est celui des hiérarchies D3. Les documents générés peuvent ainsi être directement transmis au [client web](https://github.com/budget-ouvert/client) qui permet de les visualiser. En particulier, chaque noeud est organisé comme suit :
```
interface Node = {
    name: string,
    size: number,
    children: Node[],
    ...
}
```

Il est parfois utile de générer des hiérarchies plates, afin de pouvoir rechercher des noeuds par un identifiant unique.

## Comparaison des documents

Cette partie du projet projet a été développée au cours du hackathon Datafin 2018.
Un fil des ressources de l'équipe est disponible ici : [https://forum.datafin.fr/t/etude-des-projets-de-loi-de-finance-dans-le-temps/247/5](https://forum.datafin.fr/t/etude-des-projets-de-loi-de-finance-dans-le-temps/247/5)

Initialement, il s'agissant de comparer deux nomenclatures successives du Projet de Loi de Finance français. Le cadre d'application est cependant plus large, et maintenant appliqué aux recettes notamment. En effet, il s'agit, étant donnés deux graphes A et B, de former des associations de noeuds entre ces deux graphes.

La difficulté du problème vient de plusieurs points :
* plusieurs noeuds de A peuvent être associés à un même point de B, et vice-versa
* il est possible qu'un noeud de A ne soit associé à aucun noeud de B (à dessin), et vice-versa

Le notebook `./mapping_pipeline.ipynb` crée des paires de noeuds de la façon suivante :
* pour chacun des deux graphes, on construit pour chaque noeud une représentation (un embedding) de son intitulé
* pour chaque noeud du graphe A, on calcule les k noeuds les plus proches dans le graphe B
* pour chaque noeud du graphe B, on calcule les k noeuds les plus proches dans le graphe A
* on sélectionne parmi les paires ainsi générées celles qu'il faut conserver (TODO)

Des [embeddings pré-entrainés](https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.fr.vec) sont utilisés pour créer l'embedding de graphe.
