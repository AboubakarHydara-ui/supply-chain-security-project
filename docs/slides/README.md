# Slides

Les PowerPoint sont **générés depuis le code**, pas versionnés : les binaires `.pptx` sont
régénérables et **ignorés par git** (voir `.gitignore`). On garde les générateurs.

Deux decks distincts :

| Générateur | Produit | Usage |
|---|---|---|
| `generate_slides.py` | `intro-supply-chain-security.pptx` (12 slides) | **Intro projet** — présenter la consigne aux étudiants (Jour 1) |
| `generate_cours_slides.py` | `cours-supply-chain-security.pptx` (12 slides, **notes présentateur**) | **Cours de 30 min** — concepts + démonstration des outils |

## Régénérer

```bash
pip install python-pptx
cd docs/slides
python generate_slides.py         # deck d'intro projet
python generate_cours_slides.py   # deck de cours (30 min) avec notes
```

## Contenu — deck d'INTRO projet (`generate_slides.py`)

1. Titre · 2. Le problème · 3. Attaques réelles (SolarWinds, Codecov, dependency confusion, XZ)
· 4. La chaîne vérifiable · 5. Les 4 briques · 6. SLSA · 7. Démo cible attaque/défense
· 8. Planning 3 jours · 9. Évaluation · 10. Les 5 labs · 11. Prérequis & code fourni
· 12. À vous de jouer.

## Contenu — deck de COURS 30 min (`generate_cours_slides.py`)

1. Titre · 2. Plan (avec minutage) · 3. Le problème & attaques · 4. « on scanne » → « on vérifie »
· 5. **SBOM / Syft** · 6. **Scan / Grype** · 7. **Signature / cosign & Sigstore** (Fulcio, Rekor,
keyless) · 8. Attestations & provenance · 9. SLSA (niveaux) · 10. **Admission / Kyverno** ·
11. Chaîne complète + panorama outils · 12. À retenir / Q&R.

> **Notes du présentateur** : chaque slide du deck de cours contient des notes (repères de
> timing pour tenir 30 min + points à dire + idées de démo live). Visibles dans PowerPoint via
> *Affichage → Mode Présentateur*, ou sous la diapo en mode Normal.

> Pour modifier un deck : éditez le générateur correspondant et relancez. Le style (palette,
> mises en page) est centralisé en haut de chaque script.
