# Music Recommendator

## Questions d'analyse

- Nettoyer les données : supprimer les entrées d'écoute qui ne sont pas temporellement possibles, pour permettre une meilleure analyse.
- Statistiques : réaliser des statistiques sur les données, comme l'artiste le plus écouté, l'utilisateur ayant écouté le plus de morceaux, etc.
- Recommander des utilisateurs aux artistes : inverser le parsing des utilisateurs et des artistes pour répondre à ce genre de questions : "Quels sont les utilisateurs les plus à même d'apprécier un nouvel album ?"
- Optimiser les hyperparamètres : essayer de nombreuses combinaisons de paramètres d'entrée de l'algorithme d'apprentissage, pour trouver le meilleur modèle.
- Autre algorithme : essayer un autre algorithme que ALS -> mais lequel ? Existant ? Implémenté dans Spark ?
- Recommendation temps réel : essayer un outil tel que [Oryx 2](http://oryx.io/) pour créer un système évolutif de recommendations de musiques, adaptant le modèle selon les nouvelles écoutes reçues en temps réel et personnalisé pour chaque utilisateur. -> Faisable ? Trop long ? Trop difficile ?