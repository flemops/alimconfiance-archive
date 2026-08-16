# alimconfiance-archive

Archive hebdomadaire du jeu de données [Alim'confiance](https://www.data.gouv.fr/fr/datasets/resultats-des-controles-officiels-sanitaires-dispositif-dinformation-alimconfiance/) (résultats des contrôles officiels sanitaires, ministère de l'Agriculture).

**Pourquoi** : l'État n'expose que les 12 derniers mois glissants. Personne n'archive (vérifié le 16/08/2026 : 18 réutilisations data.gouv.fr, dépôts GitHub, Wayback Machine — rien). Chaque semaine non capturée est perdue définitivement.

**Comment** : le workflow [snapshot.yml](.github/workflows/snapshot.yml) tourne chaque lundi à 06:00 UTC, télécharge le CSV complet (~43 Mo, ~72 000 inspections), le valide (taille + en-tête), le compresse (~10 Mo) et le committe dans `snapshots/`. Idempotent par semaine ISO. Relance manuelle : onglet Actions → snapshot → Run workflow.

Une archive locale redondante tourne aussi sur PC (tâche planifiée « Alimconfiance Snapshot Hebdo » → `D:\CLAUDE\alimconfiance-snapshots\`, miroir OneDrive).
