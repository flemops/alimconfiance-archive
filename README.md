# alimconfiance-archive

[![snapshot](https://github.com/flemops/alimconfiance-archive/actions/workflows/snapshot.yml/badge.svg)](https://github.com/flemops/alimconfiance-archive/actions/workflows/snapshot.yml)

Archive hebdomadaire, versionnée dans Git, des résultats des contrôles officiels sanitaires publiés par le dispositif **Alim'confiance** (ministère de l'Agriculture). Projet personnel, en production depuis le 16/08/2026.

## Problème

La publication officielle est une photographie glissante : l'État ne met en ligne que les résultats récents (constat initial du 16/08/2026 : 12 derniers mois glissants ; la baisse du nombre de lignes entre le premier et le dernier snapshot est cohérente avec cette fenêtre). Un résultat qui sort de la fenêtre disparaît de la source. Sans copie régulière, l'historique n'est plus reconstituable.

## Source

Export CSV du jeu de données officiel « Résultats des contrôles officiels sanitaires – Alim'confiance » ([data.gouv.fr](https://www.data.gouv.fr/fr/datasets/resultats-des-controles-officiels-sanitaires-dispositif-dinformation-alimconfiance/)), récupéré sur le portail Opendatasoft de la DGAL (`dgal.opendatasoft.com`, jeu `export_alimconfiance`).

## Pourquoi archiver

Constat borné et daté : le 16/08/2026, une recherche sur data.gouv.fr (réutilisations), GitHub et la Wayback Machine n'a trouvé aucune archive régulière de ce jeu. Cette recherche n'était pas exhaustive ; elle justifie le projet, elle ne prouve pas que personne d'autre n'archive.

## Pipeline

[`snapshot.yml`](.github/workflows/snapshot.yml), GitHub Actions, chaque lundi 06:00 UTC (relance manuelle possible : Actions → snapshot → Run workflow) :

1. **Idempotence** : un seul snapshot par semaine ISO ; si la semaine est déjà archivée, le job s'arrête sans rien écrire.
2. **Téléchargement** du CSV complet (`curl`, 3 essais, 10 min maximum).
3. **Garde-fous** : fichier rejeté s'il pèse moins de 10 Mo ou si l'en-tête attendu (`Synthese_eval_sanit`) est absent — une page d'erreur ou un fichier tronqué ne sont jamais archivés.
4. **Compression** (`gzip -9`) et commit dans `snapshots/alimconfiance_AAAA-MM-JJ.csv.gz`, message : date et taille brute.

## Preuve

- Historique des exécutions : [onglet Actions](https://github.com/flemops/alimconfiance-archive/actions/workflows/snapshot.yml) (statut courant : badge ci-dessus).
- Un commit par snapshot : `git log --oneline -- snapshots/`.
- Dernier snapshot au 07/10/2026 : `snapshots/alimconfiance_2026-10-05.csv.gz`, 71 809 lignes de données (vérifié par décompression). Le premier, du 16/08/2026, en contenait 72 688 : le volume varie d'une semaine à l'autre, ces nombres sont des mesures datées, pas des constantes.

**Vérifier un snapshot soi-même**

```bash
gzip -t snapshots/alimconfiance_2026-10-05.csv.gz && echo "archive intègre"
gzip -dc snapshots/alimconfiance_2026-10-05.csv.gz | head -1     # en-tête : doit contenir Synthese_eval_sanit
gzip -dc snapshots/alimconfiance_2026-10-05.csv.gz | wc -l       # nombre de lignes (en-tête compris)
```

## Limites

- Granularité hebdomadaire : une modification intervenue puis annulée entre deux lundis n'est pas vue.
- Les snapshots sont des copies fidèles de l'export, sans nettoyage ni enrichissement ; leur exactitude est celle de la source.
- Aucune vérification de provenance autre que l'URL officielle et les garde-fous ci-dessus (pas de signature de la source).
- La conservation repose sur GitHub : pas d'autre copie documentée dans ce dépôt.
- Dépôt de preuve, pas un service : aucune API, aucune interface.

## Licence et attribution

| Élément | Licence |
|---|---|
| **Données** (`snapshots/`) | Données du ministère de l'Agriculture et de la Souveraineté alimentaire, publiées sous [Licence Ouverte](https://www.etalab.gouv.fr/licence-ouverte-open-licence/) (`fr-lo`, déclarée sur data.gouv.fr). Toute réutilisation doit citer la source et la date d'extraction. |
| **Code** (workflow `snapshot.yml`, ce README) | [MIT](LICENSE) |

Ce dépôt n'est ni affilié ni approuvé par le ministère.

## Mises à jour

L'action `actions/checkout` est épinglée par SHA (tag officiel `v4.3.1`, vérifié via l'API GitHub). Pour la mettre à jour : relever le SHA du nouveau tag (`gh api repos/actions/checkout/git/ref/tags/<tag>`), le remplacer dans `snapshot.yml`, puis lancer le workflow à la main avant de fusionner.
