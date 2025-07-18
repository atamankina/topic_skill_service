## Lernpfad-Generator - Stufe 1, Tag 3 (Mittwoch) - Schritt-für-Schritt-Implementierung

Dies ist die detaillierte Schritt-für-Schritt-Anleitung zur Implementierung der Tickets für Tag 3. Heute konzentrieren wir uns auf das **Erstellen von Ressourcen** mittels `POST`-Anfragen.

**Wichtig:** Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist und eure virtuelle Umgebung (`(venv)`) aktiv ist, bevor ihr beginnt!

### Epic: PLPG-API-TOPICS (Topics API)

Dieses Epic konzentriert sich weiterhin auf die Erweiterung unseres `Topic & Skill Service` um Schreiboperationen.

**Vorbereitung für den Tag: Feature-Branch erstellen**

Wie am Vortag beginnen wir mit der Erstellung eines neuen Git-Branches für die heutigen Aufgaben. Dies hält unsere Arbeit isoliert und den `main`-Branch sauber.

1. **Branch erstellen und wechseln:**
    - Stellt sicher, dass ihr auf dem `main`Branch seid und dieser aktuell ist (vom Merge am Ende von Tag 2):
        
        ```bash
        git checkout main
        git pull origin main
        
        ```
        
    - Erstellt einen neuen Branch für die heutigen Aufgaben. Ein passender Name wäre `feature/day3-post-endpoints`, da wir uns auf die Implementierung von `POST`Endpunkten konzentrieren.
        
        ```bash
        git checkout -b feature/day3-post-endpoints
        
        ```
        
    - **Erklärung:**
        - `git checkout main`: Wechselt zum `main`Branch.
        - `git pull origin main`: Holt die neuesten Änderungen vom `main`Branch auf GitHub. Dies ist wichtig, um sicherzustellen, dass euer lokaler `main`Branch auf dem neuesten Stand ist, bevor ihr einen neuen Feature-Branch davon ableitet.
        - `git checkout -b feature/day3-post-endpoints`: Erstellt einen neuen Branch namens `feature/day3-post-endpoints` und wechselt sofort zu diesem. Euer Terminal-Prompt sollte sich ändern und den neuen Branch-Namen anzeigen. Alle Änderungen, die ihr jetzt vornehmt, werden auf diesem Branch gespeichert.

### Ticket: PLPG-API-11: POST /topics Endpunkt implementieren

**Beschreibung:** Implementierung eines neuen API-Endpunkts, der es Clients (z.B. dem Frontend) ermöglicht, neue Lern-Topics zu erstellen. Dieser Endpunkt soll JSON-Daten im Request-Body entgegennehmen, eine eindeutige ID für das neue Topic generieren und das Topic in der `topics.json`-Datei speichern.

**Schritte zur Implementierung:**

1. **`app.py` aktualisieren: Imports und Route hinzufügen:**
    - Öffnet eure `app.py`Datei im Code-Editor.
    - Wir benötigen das `request`Objekt von Flask, um auf den Request-Body zuzugreifen, und das `uuid`Modul, um eindeutige IDs zu generieren.
    - Fügt die Imports hinzu und implementiert die neue `POST /topics`Route:
    
    ```python
    # app.py (Auszug)
    import os
    import json
    import uuid # Importiere das UUID-Modul, um eindeutige IDs zu generieren
    from flask import Flask, jsonify, request # request ist notwendig, um auf den Request-Body zuzugreifen
    
    # ... (DATA_DIR, TOPICS_FILE, SKILLS_FILE, data_manager bleiben gleich)
    
    # ... (hello_world und alle GET-Endpunkte bleiben gleich)
    
    @app.route('/topics', methods=['POST'])
    def create_topic():
        # 1. Daten aus dem Request-Body lesen
        # request.json versucht, den Request-Body als JSON zu parsen.
        # Wenn der Body kein gültiges JSON ist oder fehlt, ist request.json None.
        new_topic_data = request.json
    
        # 2. Grundlegende Validierung der eingehenden Daten
        # Überprüfen, ob die notwendigen Felder 'name' und 'description' vorhanden sind.
        if not new_topic_data or 'name' not in new_topic_data or 'description' not in new_topic_data:
            # Wenn Daten fehlen oder ungültig sind, geben wir einen 400 Bad Request zurück.
            # Dies ist eine Best Practice, um dem Client mitzuteilen, dass seine Anfrage fehlerhaft war.
            return jsonify({"error": "Name und Beschreibung für das Topic sind erforderlich"}), 400
    
        # 3. Eindeutige ID generieren
        # uuid.uuid4() erzeugt eine universell eindeutige ID (UUID).
        # str() wandelt sie in einen String um, da JSON-Schlüssel Strings sein müssen.
        new_topic_id = str(uuid.uuid4())
    
        # 4. Neues Topic-Objekt erstellen
        # Wir erstellen ein neues Dictionary für unser Topic.
        # Die generierte ID wird hinzugefügt.
        # Zusätzliche Felder aus den eingehenden Daten können hier übernommen werden.
        topic = {
            "id": new_topic_id,
            "name": new_topic_data['name'],
            "description": new_topic_data['description']
            # Hier könnten weitere Felder aus new_topic_data übernommen werden,
            # falls sie optional sind oder eine Standardwertlogik haben.
        }
    
        # 5. Bestehende Topics laden, neues Topic hinzufügen und speichern
        # Zuerst laden wir alle aktuellen Topics aus der Datei.
        topics = data_manager.read_data(TOPICS_FILE)
        # Dann fügen wir unser neues Topic zur Liste hinzu.
        topics.append(topic)
        # Und schreiben die aktualisierte Liste zurück in die Datei.
        data_manager.write_data(TOPICS_FILE, topics)
    
        # 6. Erfolgreiche Antwort zurückgeben
        # Bei erfolgreicher Erstellung eines neuen Ressourcen geben wir den Status 201 Created zurück.
        # Die Antwort enthält das neu erstellte Topic.
        return jsonify(topic), 201
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung des Codes für `POST /topics`:**
        - `@app.route('/topics', methods=['POST'])`: Dieser Decorator definiert einen Endpunkt, der auf `POST`Anfragen an die URL `/topics` reagiert. `POST` ist die Standardmethode zum Erstellen neuer Ressourcen.
        - `new_topic_data = request.json`: Das `request`Objekt von Flask enthält alle Informationen über die eingehende HTTP-Anfrage. `request.json` versucht automatisch, den Body der Anfrage als JSON zu parsen und gibt ein Python-Dictionary zurück. Wenn der Body kein JSON ist oder leer ist, ist `request.json` `None`.
        - `if not new_topic_data or 'name' not in new_topic_data or 'description' not in new_topic_data:`: Dies ist eine grundlegende **Validierung**. Wir prüfen, ob überhaupt Daten gesendet wurden (`new_topic_data`) und ob die erforderlichen Felder (`name`, `description`) darin enthalten sind.
        - `return jsonify({"error": "..."}), 400`: Wenn die Validierung fehlschlägt, senden wir eine JSON-Antwort mit einer Fehlermeldung und dem HTTP-Statuscode `400 Bad Request`. Dieser Code signalisiert dem Client, dass seine Anfrage syntaktisch oder semantisch fehlerhaft war.
        - `new_topic_id = str(uuid.uuid4())`: `uuid.uuid4()` generiert eine **universell eindeutige ID (UUID)**. Das ist eine sehr lange Zeichenkette, die mit extrem hoher Wahrscheinlichkeit einzigartig auf der Welt ist. Dies ist eine Best Practice, um sicherzustellen, dass eure IDs niemals kollidieren, selbst wenn viele Benutzer gleichzeitig Topics erstellen. `str()` wandelt das UUID-Objekt in einen String um, damit es in JSON gespeichert werden kann.
        - `topic = {...}`: Wir erstellen ein Python-Dictionary, das unser neues Topic repräsentiert. Die generierte ID wird hinzugefügt, und die Daten aus dem Request-Body werden übernommen.
        - `topics = data_manager.read_data(TOPICS_FILE)`: Wir lesen die *gesamte* aktuelle Liste der Topics aus der JSON-Datei.
        - `topics.append(topic)`: Wir fügen das neu erstellte Topic zur Liste hinzu.
        - `data_manager.write_data(TOPICS_FILE, topics)`: Wir schreiben die *komplette, aktualisierte*Liste der Topics zurück in die JSON-Datei. **Beachtet:** Bei JSON-Dateien als Speicher müssen wir immer die gesamte Datei neu schreiben, wenn sich etwas ändert. Bei Datenbanken ist das effizienter.
        - `return jsonify(topic), 201`: Bei Erfolg senden wir das neu erstellte Topic als JSON-Antwort zurück. Der HTTP-Statuscode `201 Created` ist eine **Best Practice** für erfolgreiche `POST`Anfragen, die eine neue Ressource erstellen. Er teilt dem Client mit, dass die Ressource erfolgreich angelegt wurde.
2. **Testen des `POST /topics` Endpunkts mit Postman:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - Öffnet Postman.
    - **Test 1: Erfolgreiche Erstellung:**
        - Erstellt eine neue Anfrage.
        - Wählt die Methode `POST`.
        - Gebt die URL ein: `http://127.0.0.1:5000/topics`
        - Geht zum Tab "Body".
        - Wählt den Radio-Button `raw` und dann aus dem Dropdown-Menü rechts `JSON`.
        - Fügt den folgenden JSON-Body ein:
            
            ```json
            {
                "name": "Grundlagen der Künstlichen Intelligenz",
                "description": "Eine Einführung in die Konzepte und Anwendungen von KI, einschließlich maschinellem Lernen und neuronalen Netzen."
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `201 Created`
            - **Body:** Das zurückgegebene JSON sollte das neu erstellte Topic enthalten, inklusive der automatisch generierten `id`.
            - **Überprüfung der Datei:** Öffnet eure `data/topics.json`Datei. Ihr solltet sehen, dass das neue Topic hinzugefügt wurde.
    - **Test 2: Ungültige Anfrage (fehlender Name):**
        - Erstellt eine neue `POST`Anfrage (oder passt die vorherige an).
        - Methode: `POST`, URL: `http://127.0.0.1:5000/topics`
        - Body (raw, JSON):
            
            ```json
            {
                "description": "Eine Einführung in die Konzepte und Anwendungen von KI."
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `400 Bad Request`
            - **Body:** `{"error": "Name und Beschreibung für das Topic sind erforderlich"}`
    - **Test 3: Ungültige Anfrage (leerer Body):**
        - Erstellt eine neue `POST`Anfrage.
        - Methode: `POST`, URL: `http://127.0.0.1:5000/topics`
        - Body: Lasst ihn leer oder sendet ungültiges JSON.
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `400 Bad Request`
            - **Body:** `{"error": "Name und Beschreibung für das Topic sind erforderlich"}`
3. **Git Commit (Zwischenstand speichern):**
    - Wenn alles funktioniert, speichert eure Änderungen.
    - Geht ins Terminal (stellt sicher, dass ihr im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day3-post-endpoints` seid).
    - Fügt die geänderten Dateien zum Staging-Bereich hinzu:
        
        ```bash
        git add app.py
        # Wenn sich topics.json geändert hat (durch Tests), könnt ihr auch diese hinzufügen,
        # aber für die Versionskontrolle ist es oft besser, nur Codeänderungen zu committen.
        # git add data/topics.json
        
        ```
        
    - Erstellt einen Commit mit einer beschreibenden Nachricht:
        
        ```bash
        git commit -m "PLPG-API-11: POST /topics Endpunkt implementiert mit Validierung und UUID"
        
        ```
        
    - Pusht die Änderungen zu GitHub (auf euren Feature-Branch):
        
        ```bash
        git push origin feature/day3-post-endpoints
        
        ```
        

### Ticket: PLPG-API-12: POST /skills Endpunkt implementieren

**Beschreibung:** Implementierung eines neuen API-Endpunkts, der es Clients ermöglicht, neue Lern-Skills zu erstellen. Dieser Endpunkt soll JSON-Daten im Request-Body entgegennehmen, eine eindeutige ID für den neuen Skill generieren und den Skill in der `skills.json`-Datei speichern. Es sollte eine grundlegende Validierung der eingehenden Skill-Daten erfolgen (z.B. dass `name` und `topicId` vorhanden sind).

**Schritte zur Implementierung:**

1. **`app.py` aktualisieren: Neue Route für `POST /skills` hinzufügen:**
    - Öffnet eure `app.py`Datei.
    - Fügt die neue `POST /skills`Route unterhalb der `POST /topics`Route hinzu. Die Logik ist sehr ähnlich, aber die Validierung muss für die Skill-spezifischen Felder (`name`, `topicId`) angepasst werden.
    
    ```python
    # app.py (Auszug)
    # ... (alle Imports, DATA_DIR, TOPICS_FILE, SKILLS_FILE, data_manager, GET-Routen bleiben gleich)
    
    @app.route('/skills', methods=['POST'])
    def create_skill():
        new_skill_data = request.json
    
        # Validierung für Skill-Daten: 'name' und 'topicId' sind erforderlich
        if not new_skill_data or 'name' not in new_skill_data or 'topicId' not in new_skill_data:
            return jsonify({"error": "Name und Topic ID für den Skill sind erforderlich"}), 400
    
        # Optional: Weitere Validierung, z.B. ob die topicId tatsächlich existiert
        # Für dieses Stadium überspringen wir das, aber es wäre eine gute Best Practice.
        # topics = data_manager.read_data(TOPICS_FILE)
        # if not any(t['id'] == new_skill_data['topicId'] for t in topics):
        #     return jsonify({"error": "Angegebene Topic ID existiert nicht"}), 400
    
        new_skill_id = str(uuid.uuid4())
    
        # Skill-Objekt erstellen. Optional können weitere Felder wie 'difficulty' hinzugefügt werden.
        skill = {
            "id": new_skill_id,
            "name": new_skill_data['name'],
            "topicId": new_skill_data['topicId'],
            "difficulty": new_skill_data.get('difficulty', 'unknown') # .get() gibt einen Standardwert zurück, falls 'difficulty' fehlt
        }
    
        skills = data_manager.read_data(SKILLS_FILE)
        skills.append(skill)
        data_manager.write_data(SKILLS_FILE, skills)
    
        return jsonify(skill), 201
    
    if __name__ == '__main__':
        app.run(debug=True, port=5000)
    
    ```
    
    - **Erklärung des Codes für `POST /skills`:**
        - Die Struktur ist identisch mit `POST /topics`. Der Hauptunterschied liegt in der **Validierungslogik**.
        - `if not new_skill_data or 'name' not in new_skill_data or 'topicId' not in new_skill_data:`: Hier prüfen wir, ob `name` und `topicId` im eingehenden JSON vorhanden sind, da diese für einen Skill unerlässlich sind.
        - `new_skill_data.get('difficulty', 'unknown')`: Dies ist eine sichere Methode, um auf optionale Felder im Dictionary zuzugreifen. Wenn `'difficulty'` im `new_skill_data`Dictionary vorhanden ist, wird sein Wert verwendet. Andernfalls wird der Standardwert `'unknown'` zugewiesen. Dies verhindert `KeyError`, wenn ein optionales Feld nicht gesendet wird.
        - Die Logik zum Lesen, Hinzufügen und Schreiben der Daten in `skills.json` ist dieselbe wie bei Topics.
2. **Testen des `POST /skills` Endpunkts mit Postman:**
    - Stoppt euren Flask-Server (`Ctrl+C`) und startet ihn neu (`python app.py`).
    - Öffnet Postman.
    - **Test 1: Erfolgreiche Erstellung:**
        - Erstellt eine neue Anfrage.
        - Methode: `POST`.
        - URL: `http://127.0.0.1:5000/skills`
        - Body (raw, JSON):
            
            ```json
            {
                "name": "Grundlagen von Python",
                "topicId": "t3",
                "difficulty": "beginner"
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `201 Created`
            - **Body:** Das zurückgegebene JSON sollte den neu erstellten Skill enthalten, inklusive der generierten `id`.
            - **Überprüfung der Datei:** Öffnet eure `data/skills.json`Datei. Ihr solltet sehen, dass der neue Skill hinzugefügt wurde.
    - **Test 2: Ungültige Anfrage (fehlende `topicId`):**
        - Erstellt eine neue `POST`Anfrage.
        - Methode: `POST`, URL: `http://127.0.0.1:5000/skills`
        - Body (raw, JSON):
            
            ```json
            {
                "name": "Fortgeschrittene Algorithmen"
            }
            
            ```
            
        - Klickt auf "Send".
        - **Erwartetes Ergebnis:**
            - **Status:** `400 Bad Request`
            - **Body:** `{"error": "Name und Topic ID für den Skill sind erforderlich"}`
3. **Git Commit (Abschluss des Tages):**
    - Wenn alle Tests erfolgreich waren und ihr mit der Implementierung zufrieden seid, speichert eure Arbeit.
    - Geht ins Terminal (stellt sicher, dass ihr im `topic_skill_service`Verzeichnis und auf dem Branch `feature/day3-post-endpoints` seid).
    - Fügt alle neuen und geänderten Dateien hinzu (insbesondere `app.py` und die aktualisierte `data/skills.json`):
        
        ```bash
        git add .
        
        ```
        
    - Erstellt einen Commit:
        
        ```bash
        git commit -m "PLPG-API-11, PLPG-API-12: POST /topics und POST /skills Endpunkte implementiert"
        
        ```
        
    - Pusht die Änderungen zu GitHub (auf euren Feature-Branch):
        
        ```bash
        git push origin feature/day3-post-endpoints
        
        ```
        

**Herzlichen Glückwunsch!** Ihr habt die Aufgaben für Tag 3 erfolgreich abgeschlossen. Ihr habt gelernt, wie man neue Ressourcen über eine API erstellt und dabei grundlegende Validierungen und Best Practices anwendet.

### Abschluss des Tages: Branch Mergen

Nachdem ihr alle Aufgaben auf eurem Feature-Branch `feature/day3-post-endpoints` erfolgreich implementiert und gepusht habt, ist es Zeit, eure Änderungen in den Hauptentwicklungszweig (`main`) zu integrieren.

1. **Zum `main`Branch wechseln:**
    - Stellt sicher, dass euer Terminal im Verzeichnis `topic_skill_service` ist.
    - Wechselt zurück zum `main`Branch:
        
        ```bash
        git checkout main
        
        ```
        
2. **Neueste Änderungen von GitHub holen:**
    - Es ist immer eine gute Praxis, den `main`Branch zu aktualisieren, bevor ihr eure Änderungen mergt, falls in der Zwischenzeit andere Änderungen vorgenommen wurden (was in einem Team vorkommen kann).
        
        ```bash
        git pull origin main
        
        ```
        
3. **Feature-Branch in `main` mergen:**
    - Jetzt mergt ihr die Änderungen von eurem Feature-Branch in den `main`Branch.
        
        ```bash
        git merge feature/day3-post-endpoints
        
        ```
        
    - Git wird euch möglicherweise eine Standard-Commit-Nachricht vorschlagen (z.B. "Merge branch 'feature/day3-post-endpoints' into main"). Speichert diese oder passt sie an.
4. **Den gemergten `main`Branch zu GitHub pushen:**
    - Nachdem der Merge lokal erfolgreich war, pusht ihr den aktualisierten `main`Branch zu GitHub:
        
        ```bash
        git push origin main
        
        ```
        
5. **Lokalen Feature-Branch löschen (Optional, aber empfohlen):**
    - Nachdem eure Änderungen sicher im `main`Branch auf GitHub sind, könnt ihr den lokalen Feature-Branch aufräumen.
        
        ```bash
        git branch -d feature/day3-post-endpoints
        
        ```
        
    - **Erklärung:** `git branch -d` löscht den lokalen Branch. Wenn ihr den Remote-Branch auf GitHub auch löschen wollt (was oft der Fall ist, nachdem ein Feature gemergt wurde), könntet ihr `git push origin --delete feature/day3-post-endpoints` verwenden. Für dieses Lernprojekt ist das Löschen des lokalen Branches ausreichend.