# Analyse de pathologies sur radios thoraciques

Projet d'initiation à la recherche — Master 2 Ingénierie Mathématique et Data Science, Université de Haute-Alsace (Mulhouse).

**Auteurs** : Lounis Mesrour, Amina Menacer, Khadija Darid
**Encadrement** : Pr Hammoudi, Adnan Mustafic (ENSISA)

## Idée du projet

Approche **centrée sur les données** : les modèles et leurs hyperparamètres sont figés, on ne modifie que les données.

```
D  → modèle → score de référence
D' = T(D) → même modèle → score ?
```

On mesure ainsi l'effet de chaque transformation des données (normalisation, contraste, augmentation, quantité de données, extracteurs de primitives) sur la détection de 14 pathologies thoraciques.

## Jeu de données

**NIH ChestX-ray14** : 112 120 radiographies thoraciques de face, 30 805 patients, 14 pathologies en **multi-label** (une radio peut en avoir 0, 1 ou plusieurs).
Source : [alkzar90/NIH-Chest-X-ray-dataset](https://huggingface.co/datasets/alkzar90/NIH-Chest-X-ray-dataset) (Hugging Face), données du NIH Clinical Center.

> Wang X. et al., *ChestX-ray8: Hospital-scale Chest X-ray Database and Benchmarks on Weakly-Supervised Classification and Localization of Common Thorax Diseases*, CVPR 2017.

Images réduites en 128×128, niveaux de gris.

## Protocole

- **Split par patient** (aucun patient partagé entre entraînement, validation et test) : test = liste officielle NIH (25 596 images), validation = 10 % des patients restants, entraînement = sous-échantillon tiré par patient.
- **Modèles classiques** : Gaussian Naive Bayes, KNN (k = 5), Random Forest (200 arbres).
- **Modèles profonds** : petit CNN, ResNet18 (pré-entraîné ImageNet), ViT-tiny (pré-entraîné ImageNet) ; 10 époques, 3 graines.
- **Métriques** : AUC et précision moyenne (AP), par pathologie et en moyenne macro.
- **Comparaison** : ΔAUC par rapport à la référence, intervalle de confiance à 95 % par bootstrap apparié par patient.

## Étapes

| Notebook | Contenu |
|---|---|
| `notebooks/etape1_baseline_nih.ipynb` | Préparation des données, split, baselines sur les données brutes |
| `notebooks/etape2_transformations_nih.ipynb` | Transformations D → D' et comparaison statistique à la référence |

### Transformations étudiées (étape 2)

| Famille | Variantes |
|---|---|
| Normalisation des intensités | ImageNet, statistiques du train, par image |
| Contraste | CLAHE (local), égalisation d'histogramme (global) |
| Augmentation | rotation ±10°, translation ±8 %, zoom 0,9–1,1, luminosité/contraste ±10 % (sans miroir) |
| Quantité de données | 5 000, 10 000, 20 000, 40 000 images d'entraînement |
| Primitives (modèles classiques) | HOG, LBP, Gabor, LBP + HOG |

## Organisation du dépôt

```
notebooks/   notebooks Colab
results/     résultats (CSV) et figures
splits/      découpages train / validation / test figés
```

## Exécution

1. Ouvrir le notebook dans Google Colab.
2. Ajouter le secret `GITHUB_TOKEN` (token GitHub *fine-grained*, droit *Contents : Read and write* sur ce dépôt).
3. *Exécution → Tout exécuter*. Les runs déjà présents dans `results/` sont sautés.

## Limites

- Étiquettes extraites automatiquement des comptes rendus : bruit d'étiquettes.
- Réduction 1024 → 128 pixels : perte de détails pour les petites lésions (Nodule, Mass).
- Hernia est très rare (0,2 %) : son AUC est instable.
