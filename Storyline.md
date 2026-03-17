# How to Run Your Own AI — Storyline

## Überblick

**Titel:** "How to Run Your Own AI: Real-Life Experiences Operating a Self-Hosted Large Language Model for SeaTable Automations"
**Konferenz:** CS3 2026, University of Oslo
**Datum:** 19. März 2026, 12:10 Uhr
**Dauer:** 15 Minuten
**Session:** AI and Storage

**Roter Faden:** Wir sind mit dem Ideal gestartet: alles self-hosted, volle Datensouveränität. Für Automationen funktioniert das. Dann wollten wir mehr — "Chat with your data" — und mussten lernen, wo Self-Hosting heute an seine Grenzen stößt. Die Antwort ist kein Entweder-Oder, sondern eine Architektur, die beides ermöglicht.

**Kernaussage:** "We thought the hard part was running the AI. Turns out, the hard part is making it useful."

---

## Akt 1: "Running your own AI" (2-3 Min)

### Entmystifizierung: Ein AI-Server ist kein Hexenwerk

- Ein AI-Server ist wie jeder andere Server — nur mit einer fetten GPU
- Unser Setup: Hetzner GPU-Server, fixe monatliche Kosten, planbar
- Software-Stack: **vLLM** als Serving-Framework, bewährter Standard
- Modell: **Gemma 3 12B** von Google — warum dieses Modell?
  - Multimodal: kann Text UND Bilder verarbeiten (war ein Muss für OCR-Aufgaben)
  - 12B Parameter: der Kompromiss — kleiner geht nicht (zu wenig Fähigkeiten), größer geht nicht (passt nicht auf unsere GPU)
- **Warum Self-Hosting?** Ein Wort: **Data Sovereignty**. Unsere Kunden sind Universitäten, Forschungseinrichtungen, öffentlicher Sektor. Daten dürfen das Netzwerk nicht verlassen.

### Kernbotschaft Akt 1

> Einen eigenen AI-Server aufzusetzen ist machbar. Die Modellauswahl ist ein Kompromiss zwischen Fähigkeiten und Hardware-Grenzen.

---

## Akt 2: "Simple tasks, simple setup" (2 Min)

### Der erste Use Case: AI-Automationen in SeaTable

- SeaTable 6.0 hat vier AI-Automationsfunktionen: **Summarize, OCR, Extract, Classify**
- Plus eine **Custom Function** für individuelle Prompts
- Wie es funktioniert: Eine Zeile als Input → AI verarbeitet → Ergebnis zurück in die Zeile
- Latenz ist tolerierbar: eine Zeile dauert ein paar Sekunden — der User wartet nicht aktiv, die Automation läuft im Hintergrund
- Aber: Token-Durchsatz bestimmt, wie viele Automationen **parallel** laufen können — das wird bei vielen Nutzern relevant

### Kernbotschaft Akt 2

> Für definierte Aufgaben auf einzelnen Zeilen funktioniert Self-Hosting gut. Die Performance ist akzeptabel, die Daten bleiben auf dem eigenen Server.

---

## Akt 3: "The lessons nobody tells you" (2-3 Min)

### Operative Wahrheiten aus dem Betrieb

**Prompting ist die halbe Miete:**
- Das Modell muss *exakt* das gewünschte Ergebnis liefern — nicht mehr, nicht weniger
- Beispiel: Du willst eine Kategorie als Ergebnis. Das Modell liefert aber "Die Kategorie ist: XYZ" statt nur "XYZ"
- Die Sprache des Anwenders muss erhalten bleiben: deutsch rein → deutsch raus. Klingt trivial, ist es nicht bei einem englisch trainierten Modell

**Monitoring ist Pflicht:**
- Tokenverbrauch, Antwortzeiten, Fehlerraten — ohne Monitoring fliegt man blind
- Wann ist das Modell überlastet? Wann produziert es Unsinn?

**Die Modellauswahl-Falle:**
- Bildverarbeitung war ein Muss → hat die Auswahl stark eingeschränkt
- Kleines Modell (z.B. 4B): kann die Aufgaben nicht zuverlässig lösen
- Großes Modell (z.B. 27B): läuft nicht performant auf unserer Hardware
- 12B ist der Sweetspot — aber nur für definierte Aufgaben

### Kernbotschaft Akt 3

> Die Hardware aufzusetzen ist der einfache Teil. Die echten Herausforderungen sind: präzises Prompting, sorgfältige Modellauswahl und konsequentes Monitoring.

---

## Akt 4: "Then we wanted more" (2 Min)

### Der Twist: Von Automation zu Conversation

- Automation = **spezifische Aufgabe** auf einer Zeile. Input definiert, Output definiert.
- Aber was, wenn der User **fragen** will? Frei formuliert, offen, explorativ?
- → **"Chat with your data"**: ein AI-Chatbot als SeaTable-Plugin, der die Daten in den Bases versteht
- Das erfordert etwas fundamental anderes:
  - Nicht mehr eine Zeile, sondern **ganze Tabellen, Verknüpfungen, Strukturen**
  - Nicht mehr eine definierte Aufgabe, sondern **allgemeine Analyse**
  - Viel mehr Tokens pro Interaktion
- **Das Problem:** Das Modell weiß nichts über die Daten des Users. Woher auch?

### Warum nicht RAG?

- RAG (Retrieval Augmented Generation) funktioniert gut für **statische Wissensbasis** — Dokumente, Handbücher
- Aber SeaTable-Daten sind **strukturiert und ändern sich ständig**
- Bei jeder Datenänderung neu embedden? Nicht praktikabel
- Wir brauchen **Live-Zugriff** auf die Datenstruktur

### Kernbotschaft Akt 4

> Der Schritt von "verarbeite eine Zeile" zu "chatte mit deinen Daten" verändert die Anforderungen fundamental. Statische Ansätze wie RAG reichen nicht — wir brauchen Live-Zugriff.

---

## Akt 5: "MCP changes everything" (2-3 Min)

### MCP: Die Brücke zwischen AI und Daten

- **MCP (Model Context Protocol):** Ein Standard, der es dem AI-Modell ermöglicht, aktiv Tools aufzurufen
- Wir haben einen **SeaTable MCP Server** entwickelt
- Was der MCP Server kann: Tabellenstruktur lesen, Daten abfragen, Verknüpfungen auflösen — alles live
- **Ohne MCP:** Das Modell rät, halluziniert, oder sagt "das weiß ich nicht"
- **Mit MCP:** Das Modell fragt aktiv nach: "Welche Tabellen gibt es? Was steht in Spalte X? Zeig mir die verknüpften Datensätze."
- **Verfügbare MCP-Tools** in 7 Kategorien: read (list_tables, list_rows, get_row, find_rows, search_rows, get_schema), write (add_row, append_rows, update_rows), delete, link, sql (query_sql), upload, options

### Warum MCP der Schlüssel ist

- MCP macht den Unterschied zwischen "nettes Spielzeug" und "echtes Werkzeug"
- Das Modell wird vom passiven Textgenerator zum **aktiven Agenten**, der mit den Daten interagiert
- Ohne MCP wäre der SeaTable AI Chatbot nicht möglich gewesen

### Unsere Zwei-Stufen-Architektur für Token-Optimierung (technisches Detail für Entwickler)

Ein konkretes Implementierungsdetail, das zeigt, wie wir die API-Kosten in den Griff bekommen:

- **Stage 1 — Tool-Selektion (günstig):** Das Modell bekommt nur die Tool-Kategorien-Namen, nicht die vollen Schemas. Es entscheidet: "Ich brauche `read` und `sql`." Nur die letzten 3 User-Nachrichten, max 256 Output-Tokens.
- **Stage 2 — Execution (voll):** Das Modell bekommt nur die selektierten Tools mit vollem Schema. Volle Chat-History (max 10 Nachrichten). Loop bis zu 10 Iterationen.
- **Ergebnis:** ~40-60% weniger Tokens pro Interaktion, weil in Stage 1 die teuren Tool-Schemas nicht mitgeschickt werden.
- Zusätzlich: **Prompt Caching** (Anthropic explizit via `cache_control`, OpenAI automatisch bei identischem Prefix), **Message Truncation** (alte Tool-Results werden zu `[truncated]` komprimiert), **dynamische max_tokens** (512 für Write-Only, 4096 sonst)

> Dieses Detail eignet sich hervorragend als Slide — es ist konkret, technisch, und das Publikum kann es sofort in eigenen Projekten anwenden.

### Kernbotschaft Akt 5

> MCP ermöglicht die sinnvolle Interaktion zwischen AI und strukturierten Daten. Es ist der Schlüssel, der "Chat with your data" erst möglich macht.

---

## Akt 6: "The reality check" (2-3 Min)

### Self-Hosting stößt an seine Grenzen

**Das Capability-Problem:**
- MCP erfordert zuverlässiges **Tool-Calling**: das Modell muss erkennen, wann es ein Tool braucht, den Call korrekt formatieren, das Ergebnis interpretieren, und entscheiden, ob es einen weiteren Call braucht
- Konkretes Beispiel: Unser AI-Chat-Plugin nutzt eine Zwei-Stufen-Architektur. Schon in **Stage 1** muss das Modell ein sauberes JSON-Array zurückgeben (`["read", "sql"]`). Gemma 3 12B bettet das regelmäßig in Fließtext ein statt reines JSON zu liefern — Stage 1 scheitert bereits.
- In **Stage 2** kommt Multi-Step Tool-Calling: Tabellenstruktur holen → richtige Tabelle identifizieren → Daten abfragen → analysieren. Gemma 3 12B: falsche Tool-Auswahl, fehlerhafte Parameter, Abbruch mitten in der Kette.
- Für MCP-Chat braucht man **Reasoning-fähige Modelle** — und die sind heute Frontier-Modelle (Claude Sonnet, GPT-4o)

**Das Speed-Problem:**
- Eine User-Frage im Chat erfordert typischerweise 3-5 Tool-Calls
- Jeder Call = Modell-Inferenz → Tool ausführen → Ergebnis verarbeiten → nächster Call
- Gemma 3 12B auf unserer GPU: X Tokens/Sekunde → **Gesamtzeit pro Chat-Antwort: inakzeptabel für interaktive Nutzung**

**Die Konsequenz: "Bring Your Own Model"**
- Für den AI Chatbot: der User bringt seinen eigenen API-Key mit
- Aktuell unterstützt: **Claude** (Haiku, Sonnet) und **OpenAI** (o4-mini, o4)
- Der User zahlt seine Tokens selbst — transparenter und einfacher für alle
- Neue Herausforderung: **Key-Sicherheit** — API-Keys werden verschlüsselt auf dem Server gespeichert

### Unser Caching-Ansatz (drei Ebenen)

- **MCP-Tool-Caching:** Die verfügbaren Tools und ihre Schemas werden beim ersten Connect gecached. Warm-up-Call im Hintergrund beim Plugin-Laden, damit die erste Nachricht sofort funktioniert.
- **Schema-Caching:** Tabellenstrukturen werden gecached und nur für die tatsächlich benötigten MCP-Calls abgefragt — nicht alles auf einmal
- **Chat-History-Persistence:** Chatverlauf wird in localStorage gespeichert (pro Base-UUID). Alte Tool-Results werden zu `[truncated]` komprimiert, um Speicher zu sparen (~2.000 Tokens Ersparnis pro altem Roundtrip). Auto-Trim bei >4MB.
- **Prompt Caching:** Beide unterstützten Provider bieten Prompt Caching — Anthropic explizit über `cache_control`-Marker, OpenAI automatisch bei identischem Prompt-Prefix. Statische Teile (System-Prompt, Tool-Schemas) werden so nur einmal tokenisiert. Gecachte Tokens kosten bei beiden Anbietern deutlich weniger.
- Caching reduziert die Anzahl der API-Calls und damit Kosten und Latenz

### Kernbotschaft Akt 6

> Gemma 3 12B scheitert am Chatbot an zwei Fronten: die Reasoning-Fähigkeit reicht nicht für zuverlässiges MCP-Tool-Calling, und der Token-Durchsatz ist zu gering für interaktive Nutzung. Die pragmatische Lösung: Bring Your Own Model — Self-Hosting für Automationen, Frontier-Modelle für den Chat.

---

## Akt 7: Wrap-up und Takeaways (1 Min)

### Die Reise in einem Bild

```
AI-Automationen                          AI-Chatbot
(Summarize, OCR, Extract, Classify)      ("Chat with your data")
─────────────────────────────────────────────────────────────────
Eine Zeile                          →    Ganze Datenbanken
Definierte Aufgabe                  →    Offene Fragen
Self-hosted Gemma 3                 →    Frontier-Modelle (BYOM)
Performance tolerierbar             →    Performance kritisch
Einfaches Prompting                 →    MCP + Reasoning + Two-Stage + Caching
```

### Die drei Takeaways

1. **Self-Hosting funktioniert** — für definierte AI-Aufgaben ist ein eigener LLM-Server machbar und sinnvoll
2. **MCP ist der Game-Changer** — es macht den Unterschied zwischen AI, die rät, und AI, die tatsächlich mit deinen Daten arbeitet
3. **Die Zukunft ist hybrid** — Self-Hosting wo es reicht, Frontier-Modelle wo es nötig ist. Die Architektur muss beides können.

### Punchline

> **From "process a row" to "chat with your data" — and why the answer is not either/or.**

---

## Anmerkungen zur Umsetzung

### Zeitplanung (15 Minuten)

| Akt | Thema | Dauer |
|-----|-------|-------|
| 1 | Running your own AI — Setup & Entmystifizierung | 2-3 Min |
| 2 | Simple tasks — Automationen | 2 Min |
| 3 | Lessons learned — Prompting, Monitoring, Modellauswahl | 2-3 Min |
| 4 | The twist — Von Automation zu Chat, warum nicht RAG | 2 Min |
| 5 | MCP changes everything | 2-3 Min |
| 6 | Reality check — Grenzen von Self-Hosting, BYOM | 2-3 Min |
| 7 | Wrap-up & Takeaways | 1 Min |

**Achtung:** Das sind 13-17 Minuten — am oberen Rand des Zeitfensters. Akte 2 und 3 lassen sich am ehesten kürzen, falls nötig.

> **TODO:** Akte 2 und 3 können hier stark gestrafft werden, weil der Status-Update-Vortrag am Vortag die AI-Automationen bereits vorgestellt hat. Idee: Akt 2 und 3 in den Status-Update-Vortrag verlagern (dort bei der "AI-Powered Automations"-Folie), sodass dieser Talk direkt mit dem Setup (Akt 1) starten und dann schneller zum Twist (Akt 4: Chat + MCP) kommen kann. Das spart 3-4 Minuten und gibt mehr Raum für die stärksten Teile (Akte 4-6).

> **TODO (Zahlen):** Bei Akt 6 steht noch "X Tokens/Sekunde" als Platzhalter. Konkrete Durchsatz-Zahlen vom Gemma 3 Setup einfügen. Beispiel: "Unser Gemma 3 schafft Y Tokens/Sekunde — für eine Chat-Antwort mit 3 MCP-Calls bedeutet das Z Sekunden Wartezeit." Echte Zahlen sind für das technische CS3-Publikum extrem wertvoll.

### Verbindung zum ersten Talk

Der Status-Update-Vortrag am Vortag (18. März) teased diesen Talk an. Die dortige "No code. No cloud. No data leaves your server."-Folie bezieht sich auf Automationen. Dieser Talk erzählt dann die vollständige Geschichte — inklusive der Stellen, wo Self-Hosting an Grenzen stößt.

### Was das Publikum mitnimmt

- **Sysadmins/IT:** Konkretes Setup (Hetzner, vLLM, Gemma 3), operative Learnings, Monitoring-Ansatz
- **Entscheider:** Die Hybrid-Architektur als pragmatische Antwort auf die Self-Hosting-vs-Cloud-Frage
- **Entwickler:** MCP als Architekturmuster, Zwei-Stufen-Architektur für Token-Optimierung, Caching-Strategie, BYOM-Key-Handling
