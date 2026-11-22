# Tutoriel U-Net, comparaison de cinq architectures

Segmentation de lésions sur des échographies mammaires (jeu *Breast Ultrasound Images*, Kaggle). Le notebook complète le U-Net du tutoriel fourni, visualise les cartes de caractéristiques de chaque couche, puis génère et compare quatre variantes.

## Les cinq architectures

| Modèle | Modification par rapport à A | Paramètres | GMACs |
|---|---|---|---|
| A. U-Net (référence) | Modèle du tutoriel complété | 31 030 593 | 12,02 |
| B. Sans connexions de saut | Suppression des concaténations | 27 897 153 | 9,61 |
| C. U-Net léger | 16 filtres de base au lieu de 64 | 1 940 817 | 0,75 |
| D. BatchNorm + Dropout | Normalisation par lots et dropout de 0,1 | 31 048 257 | 12,02 |
| E. Attention U-Net | Porte d'attention sur chaque connexion de saut | 31 380 197 | 12,29 |

Une seule fonction, `build_unet`, produit les cinq modèles. Les nombres de paramètres calculés à la main coïncident avec ceux de Keras.

## Protocole

647 images (classes `benign` et `malignant`, la classe `normal` est écartée), redimensionnées en 128×128 en niveaux de gris. Découpage 90 % / 10 % avec graine fixe : 582 images d'entraînement, 65 de validation. Entropie croisée binaire, Adam, 40 époques, même graine pour tous les modèles. Pas d'augmentation de données ni d'arrêt anticipé.

## Résultats (validation, 65 images, un entraînement par modèle)

| Modèle | IoU | Dice | IoU moyen par image | Durée (min) |
|---|---|---|---|---|
| A | **0,5281** | **0,6912** | 0,5600 | 6,6 |
| B | 0,4567 | 0,6270 | 0,4735 | 4,1 |
| C | 0,4728 | 0,6420 | 0,5562 | 2,2 |
| D | 0,5061 | 0,6720 | **0,5627** | 7,2 |
| E | 0,5150 | 0,6799 | 0,5309 | 6,5 |

Principaux constats :

- Retirer les connexions de saut coûte 7,1 points d'IoU pour environ 10 % de paramètres économisés.
- Le modèle léger, 16 fois plus petit, perd 5,5 points d'IoU global mais presque rien en IoU moyen par image.
- BatchNorm + Dropout rend la validation instable (perte d'environ 30 000 à la première époque).
- L'attention n'apporte pas de gain mesurable.

Ces conclusions reposent sur un seul entraînement par modèle, évalué à la dernière époque et sur le même ensemble que celui du suivi. Seuls les écarts d'environ deux points ou plus sont interprétables.

