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

## ✅ Zusammenfassung

| Thema                        | Beispiel / Funktion                 |
|-----------------------------|-------------------------------------|
| Dynamisches SQL             | `EXECUTE IMMEDIATE '...'`           |
| Parameter übergeben         | `USING`-Klausel                     |
| Rückgabe mit `INTO`         | `EXECUTE IMMEDIATE ... INTO x`      |
| REF CURSOR                  | Dynamisch steuerbarer Cursor        |
| Data Dictionary (Grundlagen)| Metadaten der DB                    |

---

📦 Repository: `https://github.com/ad220296/NativeDynamicSQL`
