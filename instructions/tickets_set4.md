## Personalized Learning Path Generator - PUT Endpunkte. DELETE Endpunkte

Dies ist die detaillierte Schritt-für-Schritt-Anleitung zur Implementierung der Tickets für Tag 4. Heute vervollständigen wir die **CRUD-Operationen (Create, Read, Update, Delete)** für unsere Topics und Skills im `Topic & Skill Service`.

**Wichtig:** Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist und eure virtuelle Umgebung (`(venv)`) aktiv ist, bevor ihr beginnt!

### Epic: PLPG-API-TOPICS (Topics API)

Dieses Epic konzentriert sich auf die Vervollständigung der CRUD-Operationen (Create, Read, Update, Delete) für unsere Topics und Skills.

**Vorbereitung für den Tag: Feature-Branch erstellen**

Wie an den Vortagen beginnen wir mit der Erstellung eines neuen Git-Branches für die heutigen Aufgaben. Dies hält unsere Arbeit isoliert und den `main`-Branch sauber.

1. **Branch erstellen und wechseln:**
    - Stellt sicher, dass ihr auf dem `main`Branch seid und dieser aktuell ist (vom Merge am Ende von Tag 3):
        
        ```bash
        git checkout main
        git pull origin main
        
        ```
        
    - Erstellt einen neuen Branch für die heutigen Aufgaben. Ein passender Name wäre `feature/day4-crud-topics-skills`, da wir die restlichen CRUD-Operationen für Topics und Skills implementieren.
        
        ```bash
        git checkout -b feature/day4-crud-topics-skills
        
        ```
        
    - **Erklärung:**
        - `git checkout main`: Wechselt zum `main`Branch.
        - `git pull origin main`: Holt die neuesten Änderungen vom `main`Branch auf GitHub. Dies ist wichtig, um sicherzustellen, dass euer lokaler `main`Branch auf dem neuesten Stand ist, bevor ihr einen neuen Feature-Branch davon ableitet.
        - `git checkout -b feature/day4-crud-topics-skills`: Erstellt einen neuen Branch namens `feature/day4-crud-topics-skills` und wechselt sofort zu diesem. Euer Terminal-Prompt sollte sich ändern und den neuen Branch-Namen anzeigen. Alle Änderungen, die ihr jetzt vornehmt, werden auf diesem Branch gespeichert.

### Ticket: PLPG-API-13: PUT /topics/<id> Endpunkt implementieren

**Beschreibung:** Implementierung eines API-Endpunkts, der es Clients ermöglicht, ein bestehendes Lern-Topic vollständig zu aktualisieren. Der Endpunkt soll die Topic-ID aus der URL entnehmen und die aktualisierten Daten (JSON) aus dem Request-Body lesen.

**Schritte zur Implementierung:**

1. **`app.py` aktualisieren: `PUT /topics/<id>` Route hinzufügen:**
    - Öffnet eure `app.py`Datei im Code-Editor.
    - Fügt die neue `PUT /topics/<id>`Route hinzu. Diese Route wird die ID des Topics aus der URL lesen, die aktualisierten Daten aus dem Request-Body entgegennehmen, das entsprechende Topic finden und aktualisieren.
    
    ```python
    # app.py (Auszug)
    import os
    import json
    import uuid
    from flask import Flask, jsonify, request
    
    from data_manager import JsonDataManager
    
    app = Flask(__name__)
    
    DATA_DIR = os.path.join(os.path.dirname(__file__), 'data')
    TOPICS_FILE = os.path.join(DATA_DIR, 'topics.json')
    SKILLS_FILE = os.path.join(DATA_DIR, 'skills.json')
    
    data_manager = JsonDataManager()
    
    # --- Bestehende Routen (GET, POST) bleiben gleich ---
    @app.route('/')
    def hello_world():
        return 'Hello from Topic & Skill Service!'
    
    @app.route('/topics', methods=['GET'])
    def get_topics():
        topics = data_manager.read_data(TOPICS_FILE)
        return jsonify(topics)
    
    @app.route('/topics/<id>', methods=['GET'])
    def get_topic_by_id(id):
        topics = data_manager.read_data(TOPICS_FILE)
        topic = next((t for t in topics if t['id'] == id), None)
        if topic:
            return jsonify(topic)
        return jsonify({"error": "Topic not found"}), 404
    
    @app.route('/topics', methods=['POST'])
    def create_topic():
        new_topic_data = request.json
        if not new_topic_data or 'name' not in new_topic_data or 'description' not in new_topic_data:
            return jsonify({"error": "Name und Beschreibung für das Topic sind erforderlich"}), 400
    
        new_topic_id = str(uuid.uuid4())
        topic = {
            "id": new_topic_id,
            "name": new_topic_data['name'],
            "description": new_topic_data['description']
        }
    
        topics = data_manager.read_data(TOPICS_FILE)
        topics.append(topic)
        data_manager.write_data(TOPICS_FILE, topics)
        return jsonify(topic), 201
    
    @app.route('/skills', methods=['GET'])
    def get_skills():
        skills = data_manager.read_data(SKILLS_FILE)
        return jsonify(skills)
    
    @app.route('/skills/<id>', methods=['GET'])
    def get_skill_by_id(id):
        skills = data_manager.read_data(SKILLS_FILE)
        skill = next((s for s in skills if s['id'] == id), None)
        if skill:
            return jsonify(skill)
        return jsonify({"error": "Skill not found"}), 404
    
    @app.route('/skills', methods=['POST'])
    def create_skill():
        new_skill_data = request.json
        if not new_skill_data or 'name' not in new_skill_data or 'topicId' not in new_skill_data:
            return jsonify({"error": "Name und Topic ID für den Skill sind erforderlich"}), 400
    
        new_skill_id = str(uuid.uuid4())
        skill = {
            "id": new_skill_id,
            "name": new_skill_data['name'],
            "topicId": new_skill_data['topicId'],
            "difficulty": new_skill_data.get('difficulty', 'unknown')
        }
    
        skills = data_manager.read_data(SKILLS_FILE)
        skills.append(skill)
        data_manager.write_data(SKILLS_FILE, skills)
        return jsonify(skill), 201
    
    # --- NEUER ENDPUNKT für PUT /topics/<id> ---
    @app.route('/topics/<id>', methods=['PUT'])
    def update_topic(id):
        # 1. Daten aus dem Request-Body lesen
        updated_data = request.json
    
        # 2. Validierung der eingehenden Daten
        # Für eine PUT-Anfrage, die eine vollständige Aktualisierung darstellt,
        # erwarten wir, dass alle erforderlichen Felder (name, description) vorhanden sind.
        if not updated_data or 'name' not in updated_data or 'description' not in updated_data:
            return jsonify({"error": "Name und Beschreibung für das Topic sind erforderlich"}), 400
    
        # 3. Alle Topics laden
        topics = data_manager.read_data(TOPICS_FILE)
    
        # 4. Topic in der Liste finden und Index speichern
        # Wir suchen nach dem Topic mit der übergebenen ID.
        found_index = -1
        for i, t in enumerate(topics):
            if t['id'] == id:
                found_index = i
                break
    
        # 5. Überprüfen, ob Topic gefunden wurde
        if found_index == -1:
            # Wenn das Topic nicht gefunden wurde, geben wir 404 Not Found zurück.
            return jsonify({"error": "Topic not found"}), 404
    
        # 6. Topic aktualisieren
        # Das gefundene Topic wird mit den neuen Daten überschrieben.
        # Wichtig: Die ID bleibt unverändert.
        topics[found_index]['name'] = updated_data['name']
        topics[found_index]['description'] = updated_data['description']
        # Hier könnten weitere Felder aktualisiert werden, falls sie im Request-Body sind.
        # Beispiel: topics[found_index]['prerequisites'] = updated_data.get('prerequisites', [])
    
        # 7. Aktualisierte Liste speichern
        data_manager.write_data(TOPICS_FILE, topics)
    
        # 8. Erfolgreiche Antwort zurückgeben
        # Bei erfolgreicher Aktualisierung geben wir den Status 200 OK zurück.
        # Die Antwort enthält die vollständigen Daten des aktualisierten Topics.
        return jsonify(topics[found_index]), 200
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung des Codes für `PUT /topics/<id>`:**
        - `@app.route('/topics/<id>', methods=['PUT'])`: Definiert den Endpunkt für `PUT`Anfragen. `<id>` ist ein Platzhalter in der URL, dessen Wert als Argument an die Funktion `update_topic`übergeben wird.
        - `updated_data = request.json`: Liest die JSON-Daten aus dem Request-Body, die die aktualisierten Topic-Informationen enthalten.
        - **Validierung:** Ähnlich wie bei `POST` prüfen wir, ob die erforderlichen Felder (`name`, `description`) im `updated_data` vorhanden sind. Für `PUT` ist es üblich, dass der Client die *vollständige* Ressource sendet, auch wenn nur ein Teil geändert wird (Idempotenz).
        - **Topic finden:** Wir durchsuchen die Liste der Topics, um das Topic mit der passenden `id` zu finden. `found_index` speichert die Position des Topics in der Liste.
        - **`404 Not Found`:** Wenn das Topic mit der gegebenen ID nicht gefunden wird (`found_index == -1`), geben wir den Statuscode `404 Not Found` zurück. Dies ist eine Standard-Antwort, wenn eine angeforderte Ressource nicht existiert.
        - **Aktualisierung:** Wenn das Topic gefunden wird, aktualisieren wir seine Attribute mit den Werten aus `updated_data`. Wichtig ist, dass die `id` des Topics nicht geändert wird.
        - `data_manager.write_data(TOPICS_FILE, topics)`: Die gesamte Liste der Topics wird mit dem aktualisierten Topic zurück in die Datei geschrieben.
        - `return jsonify(topics[found_index]), 200`: Bei Erfolg senden wir das aktualisierte Topic zurück und den Statuscode `200 OK`. Dieser Code signalisiert, dass die Anfrage erfolgreich bearbeitet wurde.
2. **Testen des `PUT /topics/<id>` Endpunkts mit Postman:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - **Vorbereitung:** Stellt sicher, dass ihr ein Topic in eurer `data/topics.json` habt, das ihr aktualisieren könnt. Ihr könnt auch zuerst eine `POST`Anfrage an `/topics` senden, um ein neues Topic zu erstellen und dessen ID zu erhalten.
    - **Test 1: Erfolgreiche Aktualisierung:**
        - Erstellt eine neue Anfrage in Postman.
        - Wählt die Methode `PUT`.
        - Gebt die URL ein: `http://127.0.0.1:5000/topics/<ID_eines_existierenden_Topics>` (ersetzt `<ID_eines_existierenden_Topics>` durch eine tatsächliche ID, z.B. `t1` oder die ID eines Topics, das ihr gerade erstellt habt).
        - Geht zum Tab "Body".
        - Wählt `raw` und `JSON`.
        - Fügt den folgenden JSON-Body ein, um das Topic zu aktualisieren:
            
            ```json
            {
                "name": "Web Dev Grundlagen (Aktualisiert)",
                "description": "Alle Kernkonzepte des Web Development, jetzt mit Fokus auf moderne Standards."
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `200 OK`
            - **Body:** Das zurückgegebene JSON sollte das aktualisierte Topic mit den neuen `name` und `description` enthalten.
            - **Überprüfung der Datei:** Öffnet eure `data/topics.json`Datei. Ihr solltet sehen, dass das entsprechende Topic aktualisiert wurde.
    - **Test 2: Topic nicht gefunden:**
        - Methode: `PUT`, URL: `http://127.0.0.1:5000/topics/nicht_existierende_id`
        - Body (raw, JSON): (kann gültige Daten enthalten, da der Fehler vor der Validierung der Daten auftritt)
            
            ```json
            {
                "name": "Test",
                "description": "Test"
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `404 Not Found`
            - **Body:** `{"error": "Topic not found"}`
    - **Test 3: Ungültige Anfrage (fehlender Name):**
        - Methode: `PUT`, URL: `http://127.0.0.1:5000/topics/<ID_eines_existierenden_Topics>`
        - Body (raw, JSON):
            
            ```json
            {
                "description": "Nur Beschreibung, kein Name."
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `400 Bad Request`
            - **Body:** `{"error": "Name und Beschreibung für das Topic sind erforderlich"}`

### Ticket: PLPG-API-15: PUT /skills/<id> Endpunkt implementieren

**Beschreibung:** Implementierung eines API-Endpunkts, der es Clients ermöglicht, einen bestehenden Lern-Skill vollständig zu aktualisieren. Der Endpunkt soll die Skill-ID aus der URL entnehmen und die aktualisierten Daten (JSON) aus dem Request-Body lesen.

**Schritte zur Implementierung:**

1. **`app.py` aktualisieren: `PUT /skills/<id>` Route hinzufügen:**
    - Öffnet eure `app.py`Datei.
    - Fügt die neue `PUT /skills/<id>`Route hinzu. Die Logik ist fast identisch mit `PUT /topics/<id>`, aber angepasst für Skills.
    
    ```python
    # app.py (Auszug)
    # ... (alle Imports, Dateipfade, data_manager, bestehende Routen bleiben gleich)
    
    # --- NEUER ENDPUNKT für PUT /skills/<id> ---
    @app.route('/skills/<id>', methods=['PUT'])
    def update_skill(id):
        updated_data = request.json
    
        # Validierung für Skill-Daten: 'name' und 'topicId' sind erforderlich
        if not updated_data or 'name' not in updated_data or 'topicId' not in updated_data:
            return jsonify({"error": "Name und Topic ID für den Skill sind erforderlich"}), 400
    
        skills = data_manager.read_data(SKILLS_FILE)
    
        found_index = -1
        for i, s in enumerate(skills):
            if s['id'] == id:
                found_index = i
                break
    
        if found_index == -1:
            return jsonify({"error": "Skill not found"}), 404
    
        # Skill aktualisieren
        skills[found_index]['name'] = updated_data['name']
        skills[found_index]['topicId'] = updated_data['topicId']
        skills[found_index]['difficulty'] = updated_data.get('difficulty', skills[found_index].get('difficulty', 'unknown'))
        # .get('difficulty', skills[found_index].get('difficulty', 'unknown'))
        # Bedeutung: Versuche, 'difficulty' aus den updated_data zu holen.
        # Wenn nicht vorhanden, versuche es aus dem bestehenden Skill zu holen.
        # Wenn auch dort nicht vorhanden, setze 'unknown'.
    
        data_manager.write_data(SKILLS_FILE, skills)
    
        return jsonify(skills[found_index]), 200
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung des Codes für `PUT /skills/<id>`:**
        - Die Logik ist fast identisch mit `PUT /topics/<id>`.
        - Die Validierung prüft auf `name` und `topicId`.
        - Die Aktualisierung des `difficulty`Feldes verwendet `get()` mit zwei Fallback-Werten: Zuerst wird versucht, den Wert aus den `updated_data` zu holen. Wenn er dort nicht ist, wird der *bestehende* Wert des Skills beibehalten (`skills[found_index].get('difficulty', 'unknown')`). Dies stellt sicher, dass `difficulty` nicht verloren geht, wenn es nicht in der `PUT`Anfrage enthalten ist, aber auch einen Standardwert hat, falls es noch nie gesetzt wurde.
2. **Testen des `PUT /skills/<id>` Endpunkts mit Postman:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - **Vorbereitung:** Stellt sicher, dass ihr einen Skill in eurer `data/skills.json` habt, den ihr aktualisieren könnt. Ihr könnt auch zuerst eine `POST`Anfrage an `/skills` senden, um einen neuen Skill zu erstellen und dessen ID zu erhalten.
    - **Test 1: Erfolgreiche Aktualisierung:**
        - Methode: `PUT`, URL: `http://127.0.0.1:5000/skills/<ID_eines_existierenden_Skills>`
        - Body (raw, JSON):
            
            ```json
            {
                "name": "HTML & CSS Layouts (Aktualisiert)",
                "topicId": "t1",
                "difficulty": "intermediate"
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:** Status `200 OK`, aktualisierter Skill im Body.
    - **Test 2: Skill nicht gefunden:**
        - Methode: `PUT`, URL: `http://127.0.0.1:5000/skills/nicht_existierende_id`
        - Body (gültige Daten):
            
            ```json
            {
                "name": "Test Skill",
                "topicId": "t1"
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:** Status `404 Not Found`, Body `{"error": "Skill not found"}`.
    - **Test 3: Ungültige Anfrage (fehlende `topicId`):**
        - Methode: `PUT`, URL: `http://127.0.0.1:5000/skills/<ID_eines_existierenden_Skills>`
        - Body (raw, JSON):
            
            ```json
            {
                "name": "Nur Name, keine Topic ID."
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:** Status `400 Bad Request`, Body `{"error": "Name und Topic ID für den Skill sind erforderlich"}`.

### Git Commit (Zwischenstand nach PUT-Endpunkten):

- Wenn beide `PUT`Endpunkte funktionieren, speichert eure Arbeit.
- Geht ins Terminal (im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day4-crud-topics-skills` seid).
- Fügt die geänderten Dateien hinzu:
    
    ```bash
    git add app.py
    
    ```
    
- Erstellt einen Commit:
    
    ```bash
    git commit -m "PLPG-API-13, PLPG-API-15: PUT /topics/<id> und PUT /skills/<id> Endpunkte implementiert"
    
    ```
    
- Pusht die Änderungen zu GitHub:
    
    ```bash
    git push origin feature/day4-crud-topics-skills
    
    ```
    

### Ticket: PLPG-API-14: DELETE /topics/<id> Endpunkt implementieren

**Beschreibung:** Implementierung eines API-Endpunkts, der es Clients ermöglicht, ein bestehendes Lern-Topic zu löschen. Der Endpunkt soll die Topic-ID aus der URL entnehmen.

**Schritte zur Implementierung:**

1. **`app.py` aktualisieren: `DELETE /topics/<id>` Route hinzufügen:**
    - Öffnet eure `app.py`Datei.
    - Fügt die neue `DELETE /topics/<id>`Route hinzu.
    
    ```python
    # app.py (Auszug)
    # ... (alle Imports, Dateipfade, data_manager, bestehende Routen bleiben gleich)
    
    # --- NEUER ENDPUNKT für DELETE /topics/<id> ---
    @app.route('/topics/<id>', methods=['DELETE'])
    def delete_topic(id):
        topics = data_manager.read_data(TOPICS_FILE)
    
        # Topic finden und dessen Index speichern
        found_index = -1
        for i, t in enumerate(topics):
            if t['id'] == id:
                found_index = i
                break
    
        if found_index == -1:
            # Wenn das Topic nicht gefunden wurde, 404 Not Found
            return jsonify({"error": "Topic not found"}), 404
    
        # Topic aus der Liste entfernen
        # pop(index) entfernt das Element an der angegebenen Position
        deleted_topic = topics.pop(found_index)
    
        # Aktualisierte Liste speichern
        data_manager.write_data(TOPICS_FILE, topics)
    
        # Erfolgreiche Antwort: 204 No Content
        # 204 No Content ist die Best Practice für DELETE-Anfragen, die erfolgreich waren
        # und keine Antwortdaten zurückgeben müssen.
        return '', 204 # Leerer String als Body, 204 als Statuscode
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung des Codes für `DELETE /topics/<id>`:**
        - `topics = data_manager.read_data(TOPICS_FILE)`: Lädt die aktuelle Liste der Topics.
        - **Topic finden:** Wir suchen nach dem Topic mit der übergebenen `id`.
        - **`404 Not Found`:** Wenn das Topic nicht gefunden wird, geben wir `404 Not Found` zurück.
        - `deleted_topic = topics.pop(found_index)`: Wenn das Topic gefunden wird, wird es mit der `pop()`Methode aus der Liste entfernt. `pop()` entfernt das Element am angegebenen Index und gibt es zurück (hier in `deleted_topic` gespeichert, was aber nicht unbedingt benötigt wird, da wir 204 zurückgeben).
        - `data_manager.write_data(TOPICS_FILE, topics)`: Die aktualisierte Liste (ohne das gelöschte Topic) wird zurück in die Datei geschrieben.
        - `return '', 204`: Bei erfolgreicher Löschung geben wir den Statuscode `204 No Content` zurück. Dies ist der empfohlene Statuscode für `DELETE`Anfragen, wenn keine Daten im Antwort-Body zurückgegeben werden müssen. Der leere String `''` dient als Body.
2. **Testen des `DELETE /topics/<id>` Endpunkts mit Postman:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - **Vorbereitung:** Stellt sicher, dass ihr ein Topic in eurer `data/topics.json` habt, das ihr löschen könnt. Am besten erstellt ihr ein neues über `POST /topics`, um es dann zu löschen.
    - **Test 1: Erfolgreiche Löschung:**
        - Erstellt eine neue Anfrage.
        - Wählt die Methode `DELETE`.
        - Gebt die URL ein: `http://127.0.0.1:5000/topics/<ID_eines_existierenden_Topics>`
        - **Wichtig:** Für DELETE-Anfragen ist normalerweise kein Request-Body erforderlich. Lasst den Body-Tab auf "none".
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `204 No Content`
            - **Body:** Sollte leer sein.
            - **Überprüfung der Datei:** Öffnet eure `data/topics.json`Datei. Das entsprechende Topic sollte nicht mehr in der Liste sein.
            - **Zusätzliche Überprüfung:** Sendet eine `GET` Anfrage an dieselbe ID (`http://127.0.0.1:5000/topics/<gelöschte_ID>`). Diese sollte nun `404 Not Found`zurückgeben.
    - **Test 2: Topic nicht gefunden:**
        - Methode: `DELETE`, URL: `http://127.0.0.1:5000/topics/nicht_existierende_id`
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `404 Not Found`
            - **Body:** `{"error": "Topic not found"}`

### Ticket: PLPG-API-16: DELETE /skills/<id> Endpunkt implementieren

**Beschreibung:** Implementierung eines API-Endpunkts, der es Clients ermöglicht, einen bestehenden Lern-Skill zu löschen. Der Endpunkt soll die Skill-ID aus der URL entnehmen.

**Schritte zur Implementierung:**

1. **`app.py` aktualisieren: `DELETE /skills/<id>` Route hinzufügen:**
    - Öffnet eure `app.py`Datei.
    - Fügt die neue `DELETE /skills/<id>`Route hinzu. Die Logik ist identisch mit `DELETE /topics/<id>`, aber angepasst für Skills.
    
    ```python
    # app.py (Auszug)
    # ... (alle Imports, Dateipfade, data_manager, bestehende Routen bleiben gleich)
    
    # --- NEUER ENDPUNKT für DELETE /skills/<id> ---
    @app.route('/skills/<id>', methods=['DELETE'])
    def delete_skill(id):
        skills = data_manager.read_data(SKILLS_FILE)
    
        found_index = -1
        for i, s in enumerate(skills):
            if s['id'] == id:
                found_index = i
                break
    
        if found_index == -1:
            return jsonify({"error": "Skill not found"}), 404
    
        skills.pop(found_index)
        data_manager.write_data(SKILLS_FILE, skills)
    
        return '', 204
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung des Codes für `DELETE /skills/<id>`:**
        - Die Logik ist exakt dieselbe wie bei `DELETE /topics/<id>`, nur dass wir hier die `skills`Liste und `SKILLS_FILE` verwenden.
2. **Testen des `DELETE /skills/<id>` Endpunkts mit Postman:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - **Vorbereitung:** Stellt sicher, dass ihr einen Skill in eurer `data/skills.json` habt, den ihr löschen könnt.
    - **Test 1: Erfolgreiche Löschung:**
        - Methode: `DELETE`, URL: `http://127.0.0.1:5000/skills/<ID_eines_existierenden_Skills>`
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:** Status `204 No Content`, leerer Body.
    - **Test 2: Skill nicht gefunden:**
        - Methode: `DELETE`, URL: `http://127.0.0.1:5000/skills/nicht_existierende_id`
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:** Status `404 Not Found`, Body `{"error": "Skill not found"}`.

### Git Commit (Abschluss der CRUD-Operationen für Topics & Skills):

- Wenn alle `PUT`und `DELETE`Endpunkte für Topics und Skills funktionieren, speichert eure Arbeit.
- Geht ins Terminal (im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day4-crud-topics-skills` seid).
- Fügt die geänderten Dateien hinzu:
    
    ```bash
    git add app.py
    # Wenn sich eure JSON-Dateien durch Tests geändert haben und ihr diese auch versionieren wollt:
    # git add data/topics.json data/skills.json
    
    ```
    
- Erstellt einen Commit:
    
    ```bash
        git commit -m "PLPG-API-14, PLPG-API-16: DELETE /topics/<id> und DELETE /skills/<id> Endpunkte implementiert"
    
    ```
    
- Pusht die Änderungen zu GitHub:
    
    ```bash
    git push origin feature/day4-crud-topics-skills
    
    ```
    

**Herzlichen Glückwunsch!** Ihr habt alle CRUD-Operationen für den `Topic & Skill Service` erfolgreich implementiert. Dieser Service ist nun funktionsfähig!

### Abschluss des Tages: Branch Mergen

Nachdem ihr alle Aufgaben auf eurem Feature-Branch `feature/day4-crud-topics-skills` erfolgreich implementiert und gepusht habt, ist es Zeit, eure Änderungen in den Hauptentwicklungszweig (`main`) zu integrieren.

1. **Zum `main`Branch wechseln:**
    - Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist.
    - Wechselt zurück zum `main`Branch:
        
        ```bash
        git checkout main
        
        ```
        
2. **Neueste Änderungen von GitHub holen:**
    - Es ist immer eine gute Praxis, den `main`Branch zu aktualisieren, bevor ihr eure Änderungen mergt, falls in der Zwischenzeit andere Änderungen vorgenommen wurden.
        
        ```bash
        git pull origin main
        
        ```
        
3. **Feature-Branch in `main` mergen:**
    - Jetzt mergt ihr die Änderungen von eurem Feature-Branch in den `main`Branch.
        
        ```bash
        git merge feature/day4-crud-topics-skills
        
        ```
        
    - Git wird euch möglicherweise eine Standard-Commit-Nachricht vorschlagen (z.B. "Merge branch 'feature/day4-crud-topics-skills' into main"). Speichert diese oder passt sie an.
4. **Den gemergten `main`Branch zu GitHub pushen:**
    - Nachdem der Merge lokal erfolgreich war, pusht ihr den aktualisierten `main`Branch zu GitHub:
        
        ```bash
        git push origin main
        
        ```
        
5. **Lokalen Feature-Branch löschen (Optional, aber empfohlen):**
    - Nachdem eure Änderungen sicher im `main`Branch auf GitHub sind, könnt ihr den lokalen Feature-Branch aufräumen.
        
        ```bash
        git branch -d feature/day4-crud-topics-skills
        
        ```
        
    - **Erklärung:** `git branch -d` löscht den lokalen Branch. Wenn ihr den Remote-Branch auf GitHub auch löschen wollt (was oft der Fall ist, nachdem ein Feature gemergt wurde), könntet ihr `git push origin --delete feature/day4-crud-topics-skills` verwenden. Für dieses Lernprojekt ist das Löschen des lokalen Branches ausreichend.