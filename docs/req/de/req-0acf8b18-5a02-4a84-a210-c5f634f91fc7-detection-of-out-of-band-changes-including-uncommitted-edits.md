---
id: 0acf8b18-5a02-4a84-a210-c5f634f91fc7
type: req
lang: de
translation-of: en
status: draft
version: 1.0.0
---

# Erkennung von Änderungen außerhalb der API, einschließlich nicht committeter Bearbeitungen

WENN eine verfolgte Datei außerhalb der API erstellt, geändert oder gelöscht wird – auch wenn die Änderung weder gestaged noch committed ist –, muss GraphMgr diese Änderung erkennen und die abgeleiteten Indizes entsprechend aktualisieren.

## Beschreibung

Die Erkennung muss für das gesamte Spektrum an Szenarien funktionieren: nicht committete Bearbeitung, lokaler Commit, Pull oder Merge von einem Remote sowie History-Rewrite. Der Benutzer hat ausdrücklich jeden Mechanismus abgelehnt, der nur gestagte oder committete Änderungen erkennt; der Git-Status darf keine Rolle dabei spielen, ob der Abfrageindex aktualisiert werden muss.

## Merkmale

1. Funktionale Eignung
2. Zuverlässigkeit

## Stufe

MUSS

## Priorität

10

## Schlagwörter

- change-detection

## Quelle

Erste Design-Diskussion am 10./11.10.2026 (first-conversation.md, Abschnitt 4); DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Verknüpfte Artefakte

### Entscheidungen

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Ziele

- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
