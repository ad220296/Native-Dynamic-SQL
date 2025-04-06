# 🧩 Native Dynamic SQL in PL/SQL

In diesem Kapitel geht es um **Native Dynamic SQL** in Oracle PL/SQL –  
ein wichtiges Werkzeug, wenn **SQL-Anweisungen erst zur Laufzeit** bekannt sind.

---

## 📌 Inhalt laut Teststoff

1. Dynamisch generierte Statements ausführen  
2. Verwendung einer Cursor-Variable als Alternative zu statischen Cursorn  
3. Grundlagen Data Dictionary (nur wissen, **was es ist** – keine Namen auswendig!)

---

## ✅ 1. Dynamische SQL-Ausführung

### 🔹 Warum dynamisch?

Manchmal ist ein SQL-Statement **nicht im Voraus bekannt**, z. B.:
- wenn der **Tabellenname** zur Laufzeit gewählt wird,
- wenn **Bedingungen** abhängig vom Benutzer generiert werden,
- oder wenn man **DDL-Befehle** (z. B. `CREATE TABLE`) braucht.

### 🔹 `EXECUTE IMMEDIATE`

Damit kannst du **beliebige SQL-Strings zur Laufzeit** ausführen.

#### 🧪 Beispiel:
```sql
BEGIN
  EXECUTE IMMEDIATE 'DELETE FROM emp WHERE deptno = 10';
END;
```

#### ✨ Auch für DDL nutzbar:
```sql
EXECUTE IMMEDIATE 'CREATE TABLE test (id NUMBER)';
```

#### ⚠️ Achtung:
```sql
-- Das geht NICHT:
EXECUTE IMMEDIATE 'CREATE TABLE :tabname (id NUMBER)' USING 'demo';  -- Fehler!
```
> ❌ Bind-Variablen können NICHT in DDL-Statements verwendet werden

---

## 🔄 2. Übergabe von Parametern mit `USING`

Du kannst auch **Werte übergeben**, damit der SQL-String **sicher und flexibel** bleibt.

#### 🧪 Beispiel:
```sql
DECLARE
  sql_stmt VARCHAR2(100);
  v_deptno NUMBER := 20;
BEGIN
  sql_stmt := 'UPDATE emp SET sal = sal * 1.1 WHERE deptno = :x';
  EXECUTE IMMEDIATE sql_stmt USING v_deptno;
END;
```

---

### ⚠️ Hinweis: Reihenfolge bei `USING`

Die Reihenfolge in der `USING`-Klausel muss **genau zur Reihenfolge der Platzhalter `:1`, `:2`, … im SQL-String passen**.

#### Beispiel:

```sql
EXECUTE IMMEDIATE
  'SELECT sal FROM emp WHERE ename = :1 AND deptno = :2'
  INTO v_sal
  USING v_ename, v_deptno;
```

> 🔄 **Falsch wäre z. B.**: `USING v_deptno, v_ename;` – denn dann bekommt `:1` den falschen Wert.

---

## 🧠 Zusatz: Einzelwert zurückgeben mit `INTO`

```sql
DECLARE
  sql_stmt VARCHAR2(100);
  v_ename emp.ename%TYPE;
  v_empno NUMBER := 7788;
BEGIN
  sql_stmt := 'SELECT ename FROM emp WHERE empno = :1';
  EXECUTE IMMEDIATE sql_stmt INTO v_ename USING v_empno;
  DBMS_OUTPUT.PUT_LINE('Name: ' || v_ename);
END;
```

> Nur möglich, wenn genau **eine Zeile** zurückgegeben wird!

---

## 🧠 3. Cursor-Variablen (REF CURSOR)

Statt statischer Cursor kann man **REF CURSORs** verwenden, um dynamisch Daten abzufragen.

### 🔹 Vorteil:
- Abfrage-Text kann **zur Laufzeit** gesetzt werden.
- Wird oft für **flexible Prozeduren oder Reports** genutzt.

#### 🧪 Beispiel:
```sql
DECLARE
  TYPE ref_cursor IS REF CURSOR;
  my_cursor ref_cursor;
  v_ename emp.ename%TYPE;
BEGIN
  OPEN my_cursor FOR 'SELECT ename FROM emp WHERE deptno = 10';
  LOOP
    FETCH my_cursor INTO v_ename;
    EXIT WHEN my_cursor%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(v_ename);
  END LOOP;
  CLOSE my_cursor;
END;
```

> Hinweis: Es gibt **starke** (`RETURN ...%ROWTYPE`) und **schwache** Cursor-Typen (`REF CURSOR`) – für den Test reicht das Verständnis.

---

## 📖 4. Data Dictionary (DD)

### ❓ Was ist das?

Das **Data Dictionary** ist die interne Verwaltung der Datenbank – es enthält:
- alle Tabellen, Spalten, Indizes, Views, Rechte, Benutzer u. v. m.
- abrufbar über sogenannte **Views** wie `USER_TABLES`, `ALL_OBJECTS` usw.

👉 Für den Test musst du **nicht die Namen** der Views oder Spalten auswendig wissen –  
aber du solltest **verstehen**, dass man dort **Metadaten über die DB-Struktur** abfragen kann.

---

### 📘 Mini-Faktenkasten: Data Dictionary (für den Test)

- Das Data Dictionary enthält **Metainformationen über die Datenbank**
- Zugriff über `SELECT` möglich (z. B. `SELECT * FROM USER_TABLES`)
- ❗ **Du musst keine Namen oder Spalten auswendig wissen**
- Wenn etwas gebraucht wird, wird es **in der Aufgabenstellung angegeben**

#### Beispiel (nicht prüfungsrelevant, nur zur Info):

```sql
SELECT table_name FROM user_tables;
SELECT column_name FROM user_tab_columns WHERE table_name = 'EMP';
```

---

## ✅ Zusammenfassung

| Thema                        | Beispiel / Funktion                 |
|-----------------------------|-------------------------------------|
| Dynamisches SQL             | `EXECUTE IMMEDIATE '...'`           |
| Parameter übergeben         | `USING`-Klausel                     |
| Rückgabe mit `INTO`         | `EXECUTE IMMEDIATE ... INTO x`      |
| REF CURSOR                  | Dynamisch steuerbarer Cursor        |
| Data Dictionary (Grundlagen)| Metadaten der DB                    |


---
# ⚙️ Native Dynamic SQL – Quiz & Übungsbeispiel

> Ergänzung zum Kapitel: Native Dynamic SQL in PL/SQL (Stand: 06.04.2025)

---

## 🧠 Multiple-Choice-Test – Native Dynamic SQL

Wähle jeweils **eine oder mehrere** richtige Antworten!

---

### ❓ 1. Was macht `EXECUTE IMMEDIATE` in PL/SQL?

- [x] a) Führt ein SQL-Statement zur Laufzeit aus  
- [ ] b) Übersetzt PL/SQL in C-Code  
- [ ] c) Führt nur DDL-Befehle aus  
- [x] d) Kann auch mit `USING` verwendet werden  

---

### ❓ 2. Was passiert bei folgendem Code?

```sql
EXECUTE IMMEDIATE 'CREATE TABLE :1 (id NUMBER)' USING 'TEST';
```

- [ ] a) Die Tabelle TEST wird erstellt  
- [x] b) Es kommt zu einem Fehler, da DDL keine Bind-Variablen erlaubt  
- [ ] c) Der Code läuft nur im SQL*Plus  
- [ ] d) Das Statement funktioniert, wenn es in einer Prozedur steht  

---

### ❓ 3. Welche Aussage zu REF CURSORs ist korrekt?

- [x] a) Sie werden zur Laufzeit geöffnet  
- [ ] b) Sie funktionieren nur mit fixen SELECTs  
- [x] c) Man kann sie mit `OPEN FOR` starten  
- [x] d) Sie geben mehrere Spalten zurück  

---

### ❓ 4. Wozu dient das Data Dictionary?

- [ ] a) Zum Speichern von Benutzerdaten  
- [x] b) Zum Abfragen von Metainformationen über Tabellen, Views usw.  
- [x] c) Es enthält u. a. `USER_TABLES`, `ALL_OBJECTS`  
- [ ] d) Es muss vor der Verwendung aktiviert werden  

---

### ❓ 5. Was ist beim Verwenden von `EXECUTE IMMEDIATE ... INTO` zu beachten?

- [ ] a) Die Abfrage darf mehrere Zeilen zurückgeben  
- [x] b) Es muss genau eine Zeile zurückkommen  
- [x] c) Es wird oft mit `USING` kombiniert  
- [x] d) Nur mit SELECT möglich  

---

## 🧩 Beispielaufgabe: Dynamisches DELETE mit Parameter

### 💬 Aufgabenstellung:
> Schreiben Sie ein PL/SQL-Programm, das eine Abteilung löscht, deren Nummer zur Laufzeit übergeben wird.

---

### ✅ Lösung:

```sql
DECLARE
  v_deptno NUMBER := 50;
BEGIN
  EXECUTE IMMEDIATE 'DELETE FROM dept WHERE deptno = :1' USING v_deptno;
  DBMS_OUTPUT.PUT_LINE('Abteilung gelöscht!');
END;
/
```

---

Dieses Beispiel zeigt:
- **EXECUTE IMMEDIATE mit USING**
- sichere **Parameterübergabe**
- einfache Anwendung von dynamischem SQL

---

✅ Damit deckt dieses Dokument alle **testrelevanten Inhalte zu Native Dynamic SQL** ab.

