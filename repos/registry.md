# Repo-Registry

Liste der vom Supervisor verwalteten Repositories. Trage hier jedes Repo ein,
das das Agenten-System betreuen soll.

> Hinweis: Damit ein Sub-Agent auf ein Repo zugreifen kann, muss es im
> **Session-Scope** liegen. Private Repos müssen vor der Bearbeitung zur Session
> hinzugefügt werden (Repo-Auswahl der Web-Session).

| Repo | Sprache | Sichtbarkeit | Zuständige Agenten | Notizen |
|------|---------|--------------|--------------------|---------|
| `Luccas1997/First_Website` | HTML | public | frontend-agent, review-agent | Dieses Repo (Supervisor + Beispiel-Webseite) |
| `Luccas1997/Back2Normal` | JavaScript | private | frontend-agent, backend-agent, review-agent | JS-Projekt |
| `Luccas1997/LCA_Unternehmen` | Python | private | backend-agent, review-agent | Python-Projekt |
| `Luccas1997/Netzwerk_Stuttgart` | Python | private | backend-agent, review-agent | Python-Projekt |
| `Luccas1997/Vorlesung` | – | private | nach Sichtung | Inhalt noch unklar |
| `Luccas1997/PhD` | – | private | nach Sichtung | Inhalt noch unklar |
| `Luccas1997/Finanzen` | – | private | backend-agent, review-agent | Inhalt noch unklar |

## Vorlage für neue Einträge

```
| owner/repo | Sprache | public/private | <agenten> | <kurze Beschreibung> |
```
