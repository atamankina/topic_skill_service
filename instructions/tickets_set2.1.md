## Lernpfad-Generator - Stufe 1, Tag 2 (Dienstag) - Schritt-für-Schritt-Implementierung (Erweitert mit Git-Branches)

Dies ist die detaillierte Schritt-für-Schritt-Anleitung zur Implementierung der Hausaufgaben-Tickets für Tag 2. Konzentriert euch auf jedes Detail, um die Konzepte der Objektorientierten Programmierung (OOP) und des robusten Datenzugriffs zu verstehen.

**Wichtig:** Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist und eure virtuelle Umgebung (`(venv)`) aktiv ist, bevor ihr beginnt!

### Epic: PLPG-API-TOPICS (Topics API)

Dieses Epic konzentriert sich weiterhin auf die Verbesserung und Erweiterung unseres `Topic & Skill Service`.

**Vorbereitung für den Tag: Feature-Branch erstellen**

Bevor wir mit der Arbeit an den Tickets beginnen, erstellen wir einen neuen Git-Branch. Das ist eine Best Practice, da es uns erlaubt, an neuen Funktionen zu arbeiten, ohne den Hauptentwicklungszweig (`main`) zu beeinflussen. Sollte etwas schiefgehen, bleibt `main` sauber.

1. **Branch erstellen und wechseln:**
    - Stellt sicher, dass ihr auf dem `main`Branch seid und dieser aktuell ist:
        
        ```bash
        git checkout main
        git pull origin main
        
        ```
        
    - Erstellt einen neuen Branch für die heutigen Aufgaben. Ein guter Name könnte `feature/topic-skill-refactor` sein, da wir uns auf die Refaktorisierung und Robustheit des Topic & Skill Services konzentrieren.
        
        ```
        git checkout -b feature/topic-skill-refactor
        
        ```
        
    - **Erklärung:**
        - `git checkout main`: Wechselt zum `main`Branch.
        - `git pull origin main`: Holt die neuesten Änderungen vom `main`Branch auf GitHub.
        - `git checkout -b feature/day2-topic-skill-refactor`: Erstellt einen neuen Branch namens `feature/topic-skill-refactor` und wechselt sofort zu diesem. Ihr solltet im Terminal sehen, dass ihr auf diesem neuen Branch seid.

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
        - Testet auch mit IDs, die nicht existieren, um die 404-Fehler zu überprüfen.
    - Alle Endpunkte sollten weiterhin wie erwartet funktionieren.
5. **Git Commit (Zwischenstand speichern):**
    - Wenn alles funktioniert, ist es Zeit, eure Änderungen zu speichern.
    - Geht ins Terminal (stellt sicher, dass ihr im `topic_skill_service`Verzeichnis und auf dem Branch `feature/topic-skill-refactor` seid).
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
        git push origin feature/topic-skill-refactor
        
        ```