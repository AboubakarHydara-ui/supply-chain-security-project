# Threat model - Chaine d'approvisionnement logicielle

- **Groupe :** Aboubakar Hydara
- **Date :** 16 juillet 2026
- **Repo :** https://github.com/AboubakarHydara-ui/supply-chain-security-project

## 1. Actif a proteger

L'actif principal est l'image conteneur executee dans Kubernetes. La propriete recherchee est que l'image en production corresponde exactement a l'artefact construit par la chaine de build attendue, depuis le depot GitHub du projet, et non a une image remplacee ou construite ailleurs.

Les proprietes visees sont :

- **integrite** : l'image n'a pas ete modifiee apres construction ;
- **authenticite** : l'image provient bien du workflow GitHub Actions autorise ;
- **tracabilite** : une provenance SLSA indique le commit et le workflow ;
- **controle d'admission** : Kubernetes refuse les images qui ne prouvent pas ces proprietes.

## 2. Surface et acteurs de menace

Les acteurs et surfaces consideres sont :

- attaquant qui pousse une image non signee dans un registry ;
- developpeur presse qui deploie un tag mutable comme `latest` ;
- substitution d'image dans le registry apres signature ;
- registry externe ou typosquatting ;
- dependance vulnerable ou compromise ;
- pipeline CI modifie ou compromis ;
- compte GitHub ou token compromis.

## 3. Table menaces -> controles -> couverture

| # | Menace | Vecteur | Controle mis en place | Couverture | Residuel |
|---|---|---|---|---|---|
| T1 | Image non autorisee | Deploiement direct dans Kubernetes | Kyverno `verifyImages` en `Enforce` | Forte | Depend de la disponibilite GHCR/Rekor |
| T2 | Image modifiee apres build | Substitution dans le registry | Signature cosign liee au digest | Forte | Si le build initial est compromis, la signature peut signer un mauvais artefact |
| T3 | Origine inconnue | Image construite hors workflow | Signature keyless OIDC avec subject GitHub Actions exact | Forte | Workflow modifiable par mainteneur autorise |
| T4 | Absence de provenance | Artefact sans trace du build | Attestation SLSA exigee par Kyverno | Forte | Provenance simple, pas generateur SLSA L3 isole |
| T5 | Tag mutable | `:latest` ou tag deplace | Politique `disallow-latest-tag` + deploiement par digest | Forte | Aucun si toutes les images sont par digest |
| T6 | Registry pirate | Docker Hub ou registry externe | Politique `allowed-registries` | Forte | GHCR reste un tiers de confiance |
| T7 | Dependances vulnerables | Paquets Python ou systeme | SBOM Syft + scan Grype `--fail-on critical --only-fixed` | Moyenne | 0-day, CVE non corrigees, severites High tolerees |
| T8 | Mauvaise gestion des secrets | Token GitHub ou cle commite | `.gitignore`, pas de `cosign.key` commite, signature keyless en CI | Moyenne | Rotation et moindre privilege a formaliser |
| T9 | CI compromise | Modification du workflow | Signature keyless permet d'identifier le workflow exact | Moyenne | Necessite protection de branche et revue obligatoire pour durcir |

## 4. Couverture observee en demo

Les controles suivants ont ete verifies en pratique :

- image CI signee et attestee acceptee par Kyverno ;
- image non signee refusee ;
- image signee manuellement refusee apres passage en mode keyless CI ;
- image Docker Hub refusee par la politique registry ;
- tag `latest` refuse ;
- provenance SLSA verifiable avec `cosign verify-attestation` ;
- GitHub Actions execute le build, le scan, la signature et les attestations.

## 5. Hors perimetre et risques residuels

Ce projet ne couvre pas completement :

- la compromission du poste developpeur ;
- la compromission du compte GitHub ;
- les vulnerabilites 0-day ;
- l'isolation forte du build au niveau SLSA L3 ;
- le durcissement complet RBAC du cluster ;
- la rotation et gouvernance complete des tokens.

La gestion des secrets reste un point d'attention operationnel. Les tokens doivent etre limites aux scopes necessaires, stockes hors depot Git, et soumis a une rotation reguliere.

## 6. Niveau SLSA vise vs atteint

| Exigence | Vise | Atteint | Justification |
|---|---|---|---|
| Provenance disponible | Oui | Oui | Attestation SLSA attachee a l'image CI |
| Build heberge | Oui | Oui | GitHub Actions construit et pousse l'image |
| Signature keyless | Oui | Oui | OIDC GitHub Actions + Fulcio/Rekor via cosign |
| Verification admission | Oui | Oui | Kyverno exige signature keyless et provenance |
| SLSA L2 | Oui | Approche L2 | Build heberge + provenance signee |
| SLSA L3 | Non | Non | Build non isole de maniere infalsifiable, workflow modifiable |

## 7. Ameliorations possibles

Pour aller plus loin :

- activer la protection de branche `main` ;
- imposer une pull request et une revue avant modification du workflow ;
- utiliser le generateur officiel `slsa-framework/slsa-github-generator` ;
- ajouter une politique Kyverno qui verifie aussi une attestation de scan ;
- rendre le package GHCR public ou gerer proprement les secrets de pull Kyverno ;
- comparer cosign/Sigstore avec Notation/Notary v2.
