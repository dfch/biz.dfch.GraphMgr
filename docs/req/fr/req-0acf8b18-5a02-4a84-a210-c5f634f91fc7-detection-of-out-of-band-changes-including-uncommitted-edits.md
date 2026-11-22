---
id: 0acf8b18-5a02-4a84-a210-c5f634f91fc7
type: req
lang: fr
translation-of: en
status: draft
version: 1.0.0
---

# Détection des modifications apportées en dehors de l'API, y compris les modifications non validées

LORSQU'un fichier suivi est créé, modifié ou supprimé en dehors de l'API, y compris lorsque la modification n'est ni indexée (staged) ni validée (committed), GraphMgr doit détecter ce changement et mettre à jour les index dérivés en conséquence.

## Description

La détection doit fonctionner pour l'ensemble des scénarios : modification non validée, commit local, pull ou merge depuis un dépôt distant, et réécriture de l'historique. L'utilisateur a explicitement rejeté tout mécanisme qui ne détecterait que les modifications indexées ou validées ; l'état Git ne doit jouer aucun rôle dans la décision de mettre à jour l'index de requête.

## Caractéristiques

1. Aptitude fonctionnelle
2. Fiabilité

## Niveau

DOIT

## Priorité

10

## Étiquettes

- change-detection

## Source

Première discussion de conception les 10/11.10.2026 (first-conversation.md, section 4) ; DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Artefacts liés

### Décisions

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Objectifs

- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
