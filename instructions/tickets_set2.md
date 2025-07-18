## Lernpfad-Generator - GET Endpunkte

Dies ist die detaillierte Schritt-für-Schritt-Anleitung zur Implementierung der Hausaufgaben-Tickets für Tag 2. Konzentriert euch auf jedes Detail, um die Konzepte der Objektorientierten Programmierung (OOP) und des robusten Datenzugriffs zu verstehen.

**Wichtig:** Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist und eure virtuelle Umgebung (`(venv)`) aktiv ist, bevor ihr beginnt!

### Epic: PLPG-API-TOPICS (Topics API)

Dieses Epic konzentriert sich weiterhin auf die Verbesserung und Erweiterung unseres `Topic & Skill Service`.

**Vorbereitung für den Tag: Feature-Branch erstellen**

Bevor wir mit der Arbeit an den Tickets beginnen, erstellen wir einen neuen Git-Branch. Das ist eine Best Practice, da es uns erlaubt, an neuen Funktionen zu arbeiten, ohne den Hauptentwicklungszweig (`main`) zu beeinflussen. Sollte etwas schiefgehen, bleibt `main` sauber.

1. **Branch erstellen und wechseln:**
    - Stellt sicher, dass ihr auf dem `main`Branch seid und dieser aktuell ist:
        
        ```
        git checkout main
        git pull origin main
        
        ```
        
    - Erstellt einen neuen Branch für die heutigen Aufgaben. Ein guter Name könnte `feature/day2-topic-skill-refactor` sein, da wir uns auf die Refaktorisierung und Robustheit des Topic & Skill Services konzentrieren.
        
        ```
        git checkout -b feature/day2-topic-skill-refactor
        
        ```
        
    - **Erklärung:**
        - `git checkout main`: Wechselt zum `main`Branch.
        - `git pull origin main`: Holt die neuesten Änderungen vom `main`Branch auf GitHub.
        - `git checkout -b feature/day2-topic-skill-refactor`: Erstellt einen neuen Branch namens `feature/day2-topic-skill-refactor` und wechselt sofort zu diesem. Ihr solltet im Terminal sehen, dass ihr auf diesem neuen Branch seid.

### Ticket: PLPG-API-8: Datenzugriff mit JsonDataManager refaktorisieren (OOP)

**Beschreibung:** Die Logik zum Lesen und Schreiben von JSON-Dateien soll in eine eigene Python-Klasse `JsonDataManager` ausgelagert werden. Dies verbessert die Code-Organisation durch Anwendung von Prinzipien der Objektorientierten Programmierung (OOP) und macht den Code wiederverwendbarer und wartbarer.

**Schritte zur Implementierung:**

1. **Neue Datei `data_manager.py` erstellen:**
    - Öffnet euren Code-Editor.
    - Erstellt im Verzeichnis `topic_skill_service` eine neue Datei namens `data_manager.py`.
    - Diese Datei wird unsere neue Klasse `JsonDataManager` enthalten.
2. **Klasse `JsonDataManager` in `data_manager.py` definieren:**
    - Fügt den folgenden Code in `data_manager.py` ein. Wir beginnen mit einer einfachen Version der `read_data`und `write_data`Methoden. Die Robustheit wird in Ticket PLPG-API-9 hinzugefügt.
    
    ```python
    # data_manager.py
    import json
    import os
    
    class JsonDataManager:
        """
        Diese Klasse ist für das Lesen und Schreiben von JSON-Daten aus/in Dateien zuständig.
        Sie kapselt (schließt ein) die Logik für den Dateizugriff,
        sodass andere Teile der Anwendung (wie app.py) sich nicht darum kümmern müssen,
        wie die Daten gespeichert oder geladen werden.
        """
    
        def __init__(self):
            # Der Konstruktor der Klasse. Hier können wir Initialisierungen vornehmen,
            # falls die Klasse interne Zustände bräuchte. Für jetzt ist er leer.
            pass
    
        def read_data(self, filepath):
            """
            Liest Daten aus einer JSON-Datei.
            Args:
                filepath (str): Der vollständige Pfad zur JSON-Datei.
            Returns:
                list or dict: Die aus der Datei gelesenen Daten (Liste oder Dictionary).
            """
            # In dieser ersten Version gehen wir davon aus, dass die Datei existiert und gültig ist.
            # Fehlerbehandlung kommt in Ticket PLPG-API-9.
            with open(filepath, 'r', encoding='utf-8') as f:
                return json.load(f)
    
        def write_data(self, filepath, data):
            """
            Schreibt Daten in eine JSON-Datei.
            Args:
                filepath (str): Der vollständige Pfad zur JSON-Datei.
                data (list or dict): Die Daten, die in die Datei geschrieben werden sollen.
            """
            # Sicherstellen, dass das Verzeichnis existiert, bevor geschrieben wird
            os.makedirs(os.path.dirname(filepath), exist_ok=True)
            with open(filepath, 'w', encoding='utf-8') as f:
                # json.dump schreibt die Python-Daten als JSON-Text in die Datei.
                # indent=4 macht die JSON-Ausgabe besser lesbar (Einrückung von 4 Leerzeichen).
                json.dump(data, f, indent=4, ensure_ascii=False)
    
    ```
    
    - **Erklärung: Warum diese Refaktorisierung notwendig ist**
        
        Bisher haben wir die Logik zum Lesen der JSON-Datei direkt in unserer `app.py` gehabt, möglicherweise sogar mehrfach, wenn wir verschiedene Dateien gelesen haben. Das funktioniert für kleine Projekte, führt aber schnell zu Problemen, wenn die Anwendung wächst:
        
        1. **Code-Duplizierung (DRY-Prinzip verletzt):** Stellt euch vor, ihr müsstet die Logik zum Lesen einer JSON-Datei an fünf verschiedenen Stellen in `app.py` oder in verschiedenen Services wiederholen. Jedes Mal, wenn sich die Art und Weise ändert, wie wir JSON-Dateien lesen (z.B. Fehlerbehandlung hinzufügen, Kodierung ändern), müssten wir diese Änderung an *allen* fünf Stellen vornehmen. Das ist fehleranfällig und ineffizient.
        2. **Mangelnde Organisation und Klarheit:** Die `app.py` würde mit API-Routing-Logik (welche URL macht was?) *und* Details zur Datenverarbeitung (wie lese ich eine Datei?) überladen. Das macht den Code schwerer zu lesen, zu verstehen und zu pflegen. Es ist schwerer zu erkennen, welche Aufgabe welcher Teil des Codes hat.
        3. **Starke Kopplung:** Unsere API-Endpunkte (`get_topics`, `get_skills`) sind direkt "bewusst" darüber, *wie* die Daten gespeichert werden (in JSON-Dateien) und *wie* sie gelesen werden (durch direkte Dateizugriffsfunktionen). Wenn wir uns später entscheiden, von JSON-Dateien auf eine Datenbank umzusteigen, müssten wir *jeden einzelnen* Endpunkt ändern, der Daten liest oder schreibt. Das ist sehr unflexibel.
        4. **Schwierigere Testbarkeit:** Es ist komplizierter, die Datenlese-/Schreiblogik isoliert von den API-Endpunkten zu testen, wenn alles miteinander vermischt ist.
    - **Wie die Klasse `JsonDataManager` diese Probleme löst (OOP-Prinzipien):**
        
        Die Einführung der Klasse `JsonDataManager` und die Anwendung von Objektorientierter Programmierung (OOP) helfen uns, diese Probleme zu lösen:
        
        - **Kapselung (Encapsulation):** Die Klasse `JsonDataManager` "kapselt" (schließt ein) alle Details der JSON-Dateibehandlung (Lesen, Schreiben, Fehlerbehandlung) in sich. `app.py` muss nicht mehr wissen, *wie* die Daten gespeichert oder geladen werden, sondern nur noch, *dass* sie es tun kann, indem sie die Methoden des `data_manager`Objekts aufruft (z.B. `data_manager.read_data()`). Dies schafft eine klare **Trennung der Verantwortlichkeiten**: `app.py` ist für das Routing zuständig, `JsonDataManager` für den Datenzugriff.
        - **Wiederverwendbarkeit (Reusability):** Die Methoden `read_data` und `write_data` sind jetzt in einer zentralen Klasse definiert. Jedes Mal, wenn wir JSON-Daten aus einer Datei lesen oder in eine Datei schreiben müssen (sei es für Topics, Skills oder später für Benutzer), können wir einfach die Methoden des `JsonDataManager`Objekts verwenden, ohne Code zu kopieren. Das spart Zeit und reduziert Fehler.
        - **Wartbarkeit und Flexibilität:** Wenn wir uns später entscheiden, die Speichertechnologie zu ändern (z.B. von JSON-Dateien zu einer SQLite-Datenbank), müssen wir nur die Implementierung der `JsonDataManager`Klasse anpassen (oder eine neue Klasse wie `DatabaseManager` erstellen, die dieselben Methoden `read_data` und `write_data` anbietet). Die API-Endpunkte in `app.py` müssen sich nicht ändern, da sie immer noch nur `data_manager.read_data()` aufrufen. Das macht zukünftige Änderungen viel einfacher und sicherer.
        - **Testbarkeit:** Wir können die `JsonDataManager`Klasse jetzt isoliert testen, um sicherzustellen, dass sie JSON-Dateien korrekt liest und schreibt, unabhängig von der Flask-Anwendung.
    - **Erklärung der Code-Details in `data_manager.py`:**
        - `import json`, `import os`: Wir importieren die notwendigen Module. `json` für die JSON-Serialisierung/-Deserialisierung und `os` für plattformunabhängige Pfadoperationen.
        - `class JsonDataManager:`: So definieren wir eine Klasse in Python. Eine Klasse ist wie ein Bauplan für Objekte.
        - `def __init__(self):`: Dies ist der "Konstruktor" der Klasse. Er wird automatisch aufgerufen, wenn ein neues `JsonDataManager`Objekt erstellt wird (z.B. `data_manager = JsonDataManager()`). `self`bezieht sich auf die aktuelle Instanz des Objekts. Für diese Klasse ist der Konstruktor derzeit leer, da keine Initialisierung von Instanzvariablen erforderlich ist.
        - `def read_data(self, filepath):`: Dies ist eine Methode (eine Funktion innerhalb einer Klasse). Sie nimmt den `filepath` entgegen und ist für das Lesen der JSON-Daten zuständig. Die `self`Referenz ist notwendig, da es sich um eine Instanzmethode handelt.
        - `def write_data(self, filepath, data):`: Dies ist eine weitere Methode zum Schreiben von Daten.
            - `os.makedirs(os.path.dirname(filepath), exist_ok=True)`: Diese Zeile ist sehr wichtig! Sie stellt sicher, dass alle notwendigen Verzeichnisse im Pfad `filepath` existieren, bevor versucht wird, die Datei zu schreiben. `exist_ok=True` verhindert einen Fehler, wenn die Verzeichnisse bereits vorhanden sind.
            - `json.dump(data, f, indent=4, ensure_ascii=False)`:
                - `json.dump` schreibt die Python-Daten (`data`) als JSON-Text in die geöffnete Datei (`f`).
                - `indent=4` macht die JSON-Ausgabe in der Datei besser lesbar, indem es eine Einrückung von 4 Leerzeichen verwendet.
                - `ensure_ascii=False` ist entscheidend, damit Python-Strings mit Nicht-ASCII-Zeichen (wie deutschen Umlauten: ä, ö, ü oder anderen Sonderzeichen) korrekt als solche in die JSON-Datei geschrieben werden und nicht in `\uXXXX`Sequenzen umgewandelt werden.
3. **`app.py` anpassen, um `JsonDataManager` zu verwenden:**
    - Öffnet eure `app.py`Datei.
    - Wir müssen die Art und Weise ändern, wie wir Daten lesen. Statt `read_json_file` direkt in `app.py` zu haben, werden wir eine Instanz von `JsonDataManager` erstellen und deren Methoden verwenden.
    
    ```python
    # app.py
    import os
    import json # Bleibt importiert, da jsonify intern JSON verwendet
    from flask import Flask, jsonify, request # request wird später für POST/PUT benötigt
    
    # Importiere unsere neue Klasse aus der data_manager.py Datei
    from data_manager import JsonDataManager
    
    app = Flask(__name__)
    
    # Definieren der Dateipfade
    DATA_DIR = os.path.join(os.path.dirname(__file__), 'data')
    TOPICS_FILE = os.path.join(DATA_DIR, 'topics.json')
    SKILLS_FILE = os.path.join(DATA_DIR, 'skills.json') # Füge diese Zeile hinzu, falls noch nicht geschehen
    
    # Erstelle eine Instanz unseres Datenmanagers.
    # Wir werden diese Instanz verwenden, um Daten zu lesen und zu schreiben.
    data_manager = JsonDataManager()
    
    # --- Bestehende Routen anpassen ---
    
    @app.route('/')
    def hello_world():
        return 'Hello from Topic & Skill Service!'
    
    @app.route('/topics', methods=['GET'])
    def get_topics():
        # Verwende die read_data-Methode unseres data_manager-Objekts
        topics = data_manager.read_data(TOPICS_FILE)
        return jsonify(topics)
    
    # Füge den Endpunkt für GET /skills hinzu (falls noch nicht geschehen)
    @app.route('/skills', methods=['GET'])
    def get_skills():
        skills = data_manager.read_data(SKILLS_FILE)
        return jsonify(skills)
    
    # Füge den Endpunkt für GET /topics/<id> hinzu (falls noch nicht geschehen)
    @app.route('/topics/<id>', methods=['GET'])
    def get_topic_by_id(id):
        topics = data_manager.read_data(TOPICS_FILE)
        topic = next((t for t in topics if t['id'] == id), None)
        if topic:
            return jsonify(topic)
        return jsonify({"error": "Topic not found"}), 404
    
    # Füge den Endpunkt für GET /skills/<id> hinzu (falls noch nicht geschehen)
    @app.route('/skills/<id>', methods=['GET'])
    def get_skill_by_id(id):
        skills = data_manager.read_data(SKILLS_FILE)
        skill = next((s for s in skills if s['id'] == id), None)
        if skill:
            return jsonify(skill)
        return jsonify({"error": "Skill not found"}), 404
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung der Änderungen in `app.py`:**
        - `from data_manager import JsonDataManager`: Diese Zeile importiert unsere neu erstellte Klasse aus der Datei `data_manager.py`. Dies ist der Standardweg, um Code aus anderen Python-Dateien (Modulen) zu verwenden.
        - `data_manager = JsonDataManager()`: Hier erstellen wir ein Objekt (eine "Instanz") unserer Klasse `JsonDataManager`. Dieses Objekt ist jetzt unser Werkzeug, um mit den JSON-Dateien zu interagieren. Wir erstellen es einmal, wenn die Anwendung startet, und können es dann in allen Routen wiederverwenden.
        - Alle Aufrufe wie `read_json_file(TOPICS_FILE)` werden nun zu `data_manager.read_data(TOPICS_FILE)`. Das macht den Code in `app.py` sauberer und konzentriert sich auf die API-Logik (Was soll passieren, wenn `/topics` aufgerufen wird?), während die Details des Dateizugriffs in `data_manager.py` versteckt sind. Dies ist ein Beispiel für **Abstraktion**: Wir abstrahieren die Komplexität des Dateizugriffs hinter einer einfachen Methode.
4. **Testen der Refaktorisierung:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - Verwendet Postman, um alle `GET`Endpunkte, die ihr bereits implementiert habt, erneut zu testen:
        - `GET http://127.0.0.1:5000/topics`
        - `GET http://127.0.0.1:5000/skills` (stellt sicher, dass `data/skills.json` existiert, auch wenn es leer ist)
        - `GET http://127.0.0.1:5000/topics/t1`
        - `GET http://127.0.0.1:5000/skills/s1` (falls s1 in `skills.json` existiert)
        - Testet auch mit IDs, die nicht existieren, um die 404-Fehler zu überprüfen.
    - Alle Endpunkte sollten weiterhin wie erwartet funktionieren.
5. **Git Commit (Zwischenstand speichern):**
    - Wenn alles funktioniert, ist es Zeit, eure Änderungen zu speichern.
    - Geht ins Terminal (stellt sicher, dass ihr im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day2-topic-skill-refactor` seid).
    - Fügt die neuen und geänderten Dateien zum Staging-Bereich hinzu:
        
        ```
        git add .
        
        ```
        
    - Erstellt einen Commit mit einer beschreibenden Nachricht:
        
        ```
        git commit -m "PLPG-API-8: Refaktorierung des Datenzugriffs mit JsonDataManager (OOP)"
        
        ```
        
    - Pusht die Änderungen zu GitHub (auf euren Feature-Branch):
        
        ```
        git push origin feature/day2-topic-skill-refactor
        
        ```
        

### Ticket: PLPG-API-9: Robuste JSON-Dateilesefunktion implementieren

**Beschreibung:** Die `read_data`-Methode in der `JsonDataManager`-Klasse soll so erweitert werden, dass sie Fehler beim Lesen von JSON-Dateien (z.B. wenn die Datei nicht existiert oder ungültiges JSON enthält) elegant behandelt. Dies erhöht die Stabilität unseres Microservices.

**Schritte zur Implementierung:**

1. **`read_data` in `data_manager.py` erweitern:**
    - Öffnet `data_manager.py`.
    - Die `read_data`Methode muss um `try-except`Blöcke erweitert werden, um `FileNotFoundError` und `json.JSONDecodeError` abzufangen.
    
    ```python
    # data_manager.py (aktualisierte read_data-Methode)
    import json
    import os
    
    class JsonDataManager:
        # ... (Konstruktor und write_data-Methode bleiben gleich)
    
        def read_data(self, filepath):
            """
            Liest Daten aus einer JSON-Datei.
            Args:
                filepath (str): Der vollständige Pfad zur JSON-Datei.
            Returns:
                list or dict: Die aus der Datei gelesenen Daten (Liste oder Dictionary).
                              Gibt eine leere Liste bei Fehlern oder fehlender Datei zurück.
            """
            if not os.path.exists(filepath):
                print(f"INFO: Datei nicht gefunden: {filepath}. Gebe leere Liste zurück.")
                return [] # Leere Liste zurückgeben, wenn die Datei nicht existiert
    
            try:
                with open(filepath, 'r', encoding='utf-8') as f:
                    return json.load(f)
            except json.JSONDecodeError:
                # Dieser Fehler tritt auf, wenn der Inhalt der Datei kein gültiges JSON ist.
                print(f"FEHLER: Ungültiges JSON in Datei: {filepath}. Gebe leere Liste zurück.")
                return []
            except Exception as e:
                # Fängt alle anderen unerwarteten Fehler beim Dateizugriff ab.
                print(f"FEHLER: Ein unerwarteter Fehler ist aufgetreten beim Lesen von {filepath}: {e}. Gebe leere Liste zurück.")
                return []
    
        # ... (write_data-Methode bleibt gleich)
    
    ```
    
    - **Erklärung:**
        - `if not os.path.exists(filepath):`: Dies ist die erste Prüfung. Wenn die Datei nicht existiert, geben wir sofort eine leere Liste zurück und protokollieren eine Info-Meldung. Dies verhindert einen `FileNotFoundError`, der die Anwendung zum Absturz bringen würde.
        - `try...except json.JSONDecodeError`: Dieser Block versucht, die Datei als JSON zu laden. Wenn `json.load()` einen Fehler feststellt, weil das JSON nicht korrekt ist (z.B. ein Komma fehlt, eine Klammer ist falsch), wird der `json.JSONDecodeError` abgefangen. Wir geben eine Fehlermeldung aus und ebenfalls eine leere Liste zurück. Dies ist wichtig, da eine fehlerhafte JSON-Datei ebenfalls zu einem Absturz führen könnte.
        - `except Exception as e`: Dies ist ein allgemeiner `except`Block, der alle anderen unerwarteten Fehler abfängt, die beim Öffnen oder Lesen der Datei auftreten könnten (z.B. Berechtigungsprobleme). Er dient als Fallback für unbekannte Probleme und sorgt dafür, dass die Anwendung stabil bleibt.
2. **Testen der Robustheit:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - **Test 1: Fehlende Datei:**
        - Benennt eure `data/topics.json` temporär um, z.B. in `data/topics_backup.json`.
        - Sendet eine `GET` Anfrage an `http://127.0.0.1:5000/topics` mit Postman.
        - **Erwartetes Ergebnis:** Die API sollte eine leere JSON-Liste `[]` zurückgeben. Im Terminal, in dem euer Flask-Server läuft, solltet ihr die `INFO: Datei nicht gefunden:`Meldung sehen.
        - Benennt `topics_backup.json` wieder in `topics.json` um.
    - **Test 2: Ungültiges JSON:**
        - Öffnet `data/topics.json`.
        - Macht die Datei absichtlich ungültig, z.B. indem ihr ein Komma am Ende eines Objekts entfernt oder eine Klammer löscht. Speichert die Datei.
        - Sendet eine `GET` Anfrage an `http://127.0.0.1:5000/topics` mit Postman.
        - **Erwartetes Ergebnis:** Die API sollte eine leere JSON-Liste `[]` zurückgeben. Im Terminal solltet ihr die `FEHLER: Ungültiges JSON in Datei:`Meldung sehen.
        - Korrigiert `data/topics.json` wieder in einen gültigen Zustand.
    - **Test 3: Normale Funktion:**
        - Sendet eine `GET` Anfrage an `http://127.0.0.1:5000/topics` (mit korrekter `topics.json`).
        - **Erwartetes Ergebnis:** Die API sollte die vollständige Liste der Topics zurückgeben.
3. **Git Commit (Zwischenstand speichern):**
    - Wenn alles funktioniert, speichert eure Änderungen.
    - Geht ins Terminal (stellt sicher, dass ihr im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day2-topic-skill-refactor` seid).
    - Fügt die geänderten Dateien zum Staging-Bereich hinzu:
        
        ```
        git add data_manager.py
        
        ```
        
    - Erstellt einen Commit:
        
        ```
        git commit -m "PLPG-API-9: Robuste Fehlerbehandlung für JSON-Lesen in JsonDataManager"
        
        ```
        
    - Pusht die Änderungen zu GitHub (auf euren Feature-Branch):
        
        ```
        git push origin feature/day2-topic-skill-refactor
        
        ```
        

### Ticket: PLPG-API-10: Endpunkte zur Verwendung des JsonDataManager aktualisieren

**Beschreibung:** Alle bestehenden `GET`-Endpunkte im `Topic & Skill Service` müssen so angepasst werden, dass sie die neuen, robusten Methoden der `JsonDataManager`-Klasse für den Datenzugriff verwenden. (Dies wurde bereits in Ticket PLPG-API-8 teilweise vorgenommen, aber wir bestätigen hier die vollständige Implementierung und testen alles erneut).

**Schritte zur Implementierung:**

1. **Überprüfung der `app.py`:**
    - Öffnet `app.py`.
    - Stellt sicher, dass alle `GET`Endpunkte (für `/topics`, `/skills`, `/topics/<id>`, `/skills/<id>`) die `data_manager.read_data()`Methode verwenden. Der Code sollte bereits so aussehen, wie in Schritt 3 von Ticket PLPG-API-8 gezeigt.
    
    ```python
    # app.py (Auszug, sollte bereits so sein)
    # ...
    from data_manager import JsonDataManager
    
    # ...
    data_manager = JsonDataManager()
    
    # ...
    
    @app.route('/topics', methods=['GET'])
    def get_topics():
        topics = data_manager.read_data(TOPICS_FILE) # <-- Hier wird der Manager verwendet
        return jsonify(topics)
    
    @app.route('/skills', methods=['GET'])
    def get_skills():
        skills = data_manager.read_data(SKILLS_FILE) # <-- Hier wird der Manager verwendet
        return jsonify(skills)
    
    @app.route('/topics/<id>', methods=['GET'])
    def get_topic_by_id(id):
        topics = data_manager.read_data(TOPICS_FILE) # <-- Hier wird der Manager verwendet
        topic = next((t for t in topics if t['id'] == id), None)
        if topic:
            return jsonify(topic)
        return jsonify({"error": "Topic not found"}), 404
    
    @app.route('/skills/<id>', methods=['GET'])
    def get_skill_by_id(id):
        skills = data_manager.read_data(SKILLS_FILE) # <-- Hier wird der Manager verwendet
        skill = next((s for s in skills if s['id'] == id), None)
        if skill:
            return jsonify(skill)
        return jsonify({"error": "Skill not found"}), 404
    # ...
    
    ```
    
2. **Erstellen der `data/skills.json` Datei (falls noch nicht geschehen):**
    - Um den `/skills`Endpunkt testen zu können, benötigt ihr eine `skills.json`Datei.
    - Erstellt die Datei `data/skills.json` im `data`Verzeichnis mit folgendem Inhalt:
        
        ```json
        [
          {
            "id": "s1",
            "name": "HTML Basics",
            "topicId": "t1",
            "difficulty": "beginner"
          },
          {
            "id": "s2",
            "name": "CSS Layouts (Flexbox/Grid)",
            "topicId": "t1",
            "difficulty": "intermediate"
          },
          {
            "id": "s3",
            "name": "React Hooks",
            "topicId": "t2",
            "difficulty": "advanced"
          }
        ]
        
        ```
        
3. **Alle `GET`Endpunkte erneut testen:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - Verwendet Postman, um alle `GET`Endpunkte, die ihr jetzt habt, erneut zu testen. Achtet auf die korrekten JSON-Antworten und die erwarteten Statuscodes (200 OK, 404 Not Found).
        - `GET http://127.0.0.1:5000/topics`
        - `GET http://127.0.0.1:5000/skills`
        - `GET http://127.0.0.1:5000/topics/t1`
        - `GET http://127.0.0.1:5000/topics/t_nonexistent` (sollte 404 Not Found zurückgeben)
        - `GET http://127.0.0.1:5000/skills/s2`
        - `GET http://127.0.0.1:5000/skills/s_nonexistent` (sollte 404 Not Found zurückgeben)
4. **Git Commit (Abschluss des Tages):**
    - Wenn alle Tests erfolgreich waren und ihr mit der Implementierung zufrieden seid, speichert eure Arbeit.
    - Geht ins Terminal (im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day2-topic-skill-refactor` seid).
    - Fügt alle neuen und geänderten Dateien hinzu (insbesondere `data/skills.json`):
        
        ```
        git add .
        
        ```
        
    - Erstellt einen Commit:
        
        ```
        git commit -m "PLPG-API-10: Alle GET-Endpunkte auf JsonDataManager aktualisiert und skills.json hinzugefügt"
        
        ```
        
    - Pusht die Änderungen zu GitHub (auf euren Feature-Branch):
        
        ```
        git push origin feature/day2-topic-skill-refactor
        
        ```
        

**Herzlichen Glückwunsch!** Ihr habt die Hausaufgaben für Tag 2 erfolgreich abgeschlossen. Ihr habt gelernt, wie man Code mit OOP organisiert und wie man robusten Dateizugriff implementiert. Das ist eine solide Basis für die nächsten Schritte!

### Empfohlene Best Practices (Zusammenfassung)

- **Trennung der Verantwortlichkeiten (Separation of Concerns):** Jede Komponente (z.B. eine Klasse oder ein Modul) sollte eine einzige, klar definierte Aufgabe haben. `app.py` kümmert sich um HTTP-Anfragen und Routen, `data_manager.py` kümmert sich um den Dateizugriff. Dies macht den Code leichter verständlich, testbar und wartbar.
- **Modularität:** Zerlegt euren Code in kleinere, überschaubare Dateien und Funktionen/Klassen. Das verhindert, dass eine einzelne Datei riesig und unübersichtlich wird.
- **Abstraktion:** Versteckt komplexe Implementierungsdetails hinter einer einfacheren Schnittstelle. Die `app.py`muss nicht wissen, *wie* `JsonDataManager` die Daten liest, sondern nur, *dass* sie es tun kann. Dies erleichtert den Austausch von Implementierungen (z.B. Wechsel von JSON-Dateien zu einer Datenbank).
- **Don't Repeat Yourself (DRY):** Vermeidet Code-Duplizierung. Wenn ihr denselben Code an mehreren Stellen seht, überlegt, ob ihr ihn in eine Funktion oder Klasse auslagern könnt. Die `JsonDataManager`Klasse ist ein perfektes Beispiel dafür.
- **Fehlerbehandlung:** Plant immer ein, was passieren soll, wenn etwas schiefgeht (z.B. Datei nicht gefunden, ungültige Daten). Robuste Fehlerbehandlung verhindert Abstürze und gibt nützliches Feedback.
- **Konsistente Datenstrukturen:** Haltet euch an einheitliche JSON-Strukturen für eure Daten. Das erleichtert die Arbeit sowohl im Backend als auch später im Frontend.

### Abschluss des Tages: Branch Mergen

Nachdem ihr alle Aufgaben auf eurem Feature-Branch `feature/day2-topic-skill-refactor` erfolgreich implementiert und gepusht habt, ist es Zeit, eure Änderungen in den Hauptentwicklungszweig (`main`) zu integrieren.

1. **Zum `main`Branch wechseln:**
    - Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist.
    - Wechselt zurück zum `main`Branch:
        
        ```
        git checkout main
        
        ```
        
2. **Neueste Änderungen von GitHub holen:**
    - Es ist immer eine gute Praxis, den `main`Branch zu aktualisieren, bevor ihr eure Änderungen mergt, falls in der Zwischenzeit andere Änderungen vorgenommen wurden (was in einem Team vorkommen kann).
        
        ```
        git pull origin main
        
        ```
        
3. **Feature-Branch in `main` mergen:**
    - Jetzt mergt ihr die Änderungen von eurem Feature-Branch in den `main`Branch.
        
        ```
        git merge feature/day2-topic-skill-refactor
        
        ```
        
    - Git wird euch möglicherweise eine Standard-Commit-Nachricht vorschlagen (z.B. "Merge branch 'feature/day2-topic-skill-refactor' into main"). Speichert diese oder passt sie an.
4. **Den gemergten `main`Branch zu GitHub pushen:**
    - Nachdem der Merge lokal erfolgreich war, pusht ihr den aktualisierten `main`Branch zu GitHub:
        
        ```
        git push origin main
        
        ```
        
5. **Lokalen Feature-Branch löschen (Optional, aber empfohlen):**
    - Nachdem eure Änderungen sicher im `main`Branch auf GitHub sind, könnt ihr den lokalen Feature-Branch aufräumen.
        
        ```
        git branch -d feature/day2-topic-skill-refactor
        
        ```
        
    - **Erklärung:** `git branch -d` löscht den lokalen Branch. Wenn ihr den Remote-Branch auf GitHub auch löschen wollt (was oft der Fall ist, nachdem ein Feature gemergt wurde), könntet ihr `git push origin --delete feature/day2-topic-skill-refactor` verwenden. Für dieses Lernprojekt ist das Löschen des lokalen Branches ausreichend.