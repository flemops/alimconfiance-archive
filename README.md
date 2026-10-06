# alimconfiance-archive

Archive hebdomadaire, versionnée et vérifiable du jeu de données [Alim'confiance](https://www.data.gouv.fr/fr/datasets/resultats-des-controles-officiels-sanitaires-dispositif-dinformation-alimconfiance/) (résultats des contrôles officiels sanitaires, ministère de l'Agriculture).

## Le problème

La source officielle (export CSV publié par la DGAL sur [dgal.opendatasoft.com](https://dgal.opendatasoft.com/explore/dataset/export_alimconfiance/)) n'expose que les **12 derniers mois glissants** de contrôles. Une inspection qui sort de la fenêtre disparaît de l'export. Sans copie régulière, l'historique se perd semaine après semaine.

## Recherche d'antériorité (périmètre borné)

Le 16/08/2026, avant de lancer ce dépôt, j'ai cherché une archive existante dans **trois endroits seulement** : les 18 réutilisations listées sur data.gouv.fr, les dépôts publics GitHub et la Wayback Machine. Je n'y ai trouvé aucune archive historique de ce jeu de données. Cela ne prouve pas qu'il n'en existe aucune ailleurs ; la recherche n'a pas été refaite depuis.

## Pipeline

```
export CSV officiel ──► téléchargement (curl, 3 essais) ──► garde-fous ──► gzip -9 ──► commit
                                                             │
                                  taille ≥ 10 Mo, en-tête attendu, une seule capture par semaine ISO
```

Le workflow [`snapshot.yml`](.github/workflows/snapshot.yml) tourne chaque lundi à 06:00 UTC (déclenchement manuel : onglet Actions → snapshot → Run workflow). Chaque capture est un fichier `snapshots/alimconfiance_AAAA-MM-JJ.csv.gz` (~10 Mo compressé pour ~44 Mo bruts).

## Garde-fous

| Risque | Garde-fou dans le workflow |
| --- | --- |
| Page d'erreur ou fichier tronqué archivé à la place des données | Échec si le fichier fait moins de 10 000 000 octets |
| Format changé côté source | Échec si l'en-tête ne contient pas `Synthese_eval_sanit` |
| Réponse compressée par le serveur | `curl --compressed` (voir le commit « curl --compressed : le serveur répond en gzip ») |
| Panne réseau passagère | `--retry 3`, `--max-time 600` |
| Doublons si le workflow est relancé | Idempotence : si une capture existe déjà pour la semaine ISO courante, le job s'arrête sans rien écrire |

En cas d'échec, rien n'est committé : l'historique ne contient jamais de capture que les garde-fous ont refusée.

## Vérifier une capture

Chaque capture est un commit horodaté (`git log --format='%h %ad %s' --date=short -- snapshots`). Pour contrôler la plus récente :

```bash
gzip -t snapshots/alimconfiance_2026-10-05.csv.gz              # archive intègre
gzip -dc snapshots/alimconfiance_2026-10-05.csv.gz | head -1   # en-tête (contient Synthese_eval_sanit)
gzip -dc snapshots/alimconfiance_2026-10-05.csv.gz | wc -l     # lignes : en-tête + inspections
```

Mesures relevées le 06/10/2026 sur la capture du 05/10/2026 : 44 420 721 octets bruts, 71 810 lignes (en-tête compris). Sur la première capture (16/08/2026) : 45 082 788 octets, 72 689 lignes, soit 72 688 inspections. La baisse entre les deux est cohérente avec une fenêtre glissante de 12 mois dont les plus anciennes inspections sortent.

L'historique d'exécution du workflow est public : onglet [Actions](../../actions/workflows/snapshot.yml).

## Décisions et compromis

- **Snapshot complet plutôt que différentiel** : ~10 Mo par semaine, soit une centaine de Mo par trimestre. Plus lourd, mais chaque capture est lisible seule, sans reconstruire une chaîne de deltas.
- **Git comme stockage** : pas de service à exploiter, l'historique des commits sert de journal d'audit. Limite : le dépôt grossit de ~10 Mo par semaine (à surveiller ; Git LFS ou un stockage objet seraient le palier suivant).
- **Pas de transformation des données** : la capture est le CSV brut officiel, pour qu'elle reste comparable à la source.
- **Shell + GitHub Actions seulement** : aucune dépendance applicative. Ce dépôt ne contient pas de code Python ; l'analyse des données archivées reste à faire.

## Limites

- La fréquence hebdomadaire laisse passer toute modification de la source survenue puis annulée entre deux lundis.
- Les captures n'ont pas de signature cryptographique : l'horodatage vient de l'historique Git et des journaux GitHub Actions.
- La source étant une publication officielle, aucune vérification indépendante de son contenu n'est faite ici.
- Une archive locale redondante tourne aussi sur un poste personnel (tâche planifiée hebdomadaire) ; elle n'est pas publique.
