# Kiosque de jeu cloud pour bars et restaurants

Les clients paient depuis leur smartphone (QR code affiché sur la TV) pour jouer 20 minutes à un
jeu Xbox Cloud Gaming sur la TV du bar.

## Architecture (résumé)

- **Backend cloud** (Node.js, Express, SQLite) : pages publiques (mobile, TV, back-office), webhook
  SumUp, machine à états, timers.
- **Agent local** (Node.js sur Raspberry Pi ou mini-PC dans le bar) : se connecte en sortie au
  backend par WebSocket et pilote la box Android par ADB.
- **Écran TV** : page web plein écran sur la box, en WebSocket avec le backend.
- **Client mobile** : page web responsive en français.

## Avancement

| Étape | Contenu | Statut |
|---|---|---|
| 0 | Plan de validation manuel + risques | **En cours** : voir `docs/` |
| 1 | Architecture, arborescence, schéma SQLite, machine à états, API | À venir |
| 2 | Agent local + ADB (mode mock) | À venir |
| 3 | Backend de base + écran TV | À venir |
| 4 | Page mobile | À venir |
| 5 | SumUp | À venir |
| 6 | Timer, prolongation, fin de session | À venir |
| 7 | Back-office + alertes | À venir |
| 8 | Déploiement | À venir |

## Documents

- [`docs/00-risques-et-points-a-trancher.md`](docs/00-risques-et-points-a-trancher.md) : risques,
  incohérences du cahier des charges, frais SumUp, choix techniques, questions ouvertes.
- [`docs/etape-0-validation-manuelle.md`](docs/etape-0-validation-manuelle.md) : tests à faire à
  la main sur la H96 Max, checklist légale, gabarit de compte-rendu.
