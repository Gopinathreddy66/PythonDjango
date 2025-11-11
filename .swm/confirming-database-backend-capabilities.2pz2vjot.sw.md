---
title: Confirming Database Backend Capabilities
---
This document describes how Django confirms a database backend connection and determines which features are supported. The backend is flagged as confirmed, transaction support is tested, and additional capabilities are checked. The result is a backend connection with its feature support identified, enabling Django to use the appropriate ORM features.

# Where is this flow used?

This flow is used multiple times in the codebase as represented in the following diagram:

```mermaid
graph TD;
      70d7fc49a62a8c2014a8228e3afcd410bf8120ea74f1912a9a102a05ce780efa(django/…/mysql/creation.py::create_test_db) --> 759defb0bdfd38f7a4322ee91dd025a146bfeaf34d1f398abf5da46a0d22e3a3(django/…/backends/__init__.py::confirm):::mainFlowStyle

b73fa9e725071082c7af7c931466b5d038dfd869a509c08224aa1e79489f85c7(django/…/mysql/creation.py::create_test_db) --> 759defb0bdfd38f7a4322ee91dd025a146bfeaf34d1f398abf5da46a0d22e3a3(django/…/backends/__init__.py::confirm):::mainFlowStyle

af9ddb0bdc86c88bc4632ccf77fb47f9716ea8fd7647377d2af1fc2013bb699f(django/…/commands/testserver.py::handle) --> b73fa9e725071082c7af7c931466b5d038dfd869a509c08224aa1e79489f85c7(django/…/mysql/creation.py::create_test_db)


classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9

%% Swimm:
%% graph TD;
%%       70d7fc49a62a8c2014a8228e3afcd410bf8120ea74f1912a9a102a05ce780efa(<SwmPath>[django/…/mysql/creation.py](django/contrib/gis/db/backends/mysql/creation.py)</SwmPath>::create_test_db) --> 759defb0bdfd38f7a4322ee91dd025a146bfeaf34d1f398abf5da46a0d22e3a3(<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>::confirm):::mainFlowStyle
%% 
%% b73fa9e725071082c7af7c931466b5d038dfd869a509c08224aa1e79489f85c7(<SwmPath>[django/…/mysql/creation.py](django/contrib/gis/db/backends/mysql/creation.py)</SwmPath>::create_test_db) --> 759defb0bdfd38f7a4322ee91dd025a146bfeaf34d1f398abf5da46a0d22e3a3(<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>::confirm):::mainFlowStyle
%% 
%% af9ddb0bdc86c88bc4632ccf77fb47f9716ea8fd7647377d2af1fc2013bb699f(<SwmPath>[django/…/commands/testserver.py](django/core/management/commands/testserver.py)</SwmPath>::handle) --> b73fa9e725071082c7af7c931466b5d038dfd869a509c08224aa1e79489f85c7(<SwmPath>[django/…/mysql/creation.py](django/contrib/gis/db/backends/mysql/creation.py)</SwmPath>::create_test_db)
%% 
%% 
%% classDef mainFlowStyle color:#000000,fill:#7CB9F4
%% classDef rootsStyle color:#000000,fill:#00FFF4
%% classDef Style1 color:#000000,fill:#00FFAA
%% classDef Style2 color:#000000,fill:#FFFF00
%% classDef Style3 color:#000000,fill:#AA7CB9
```

# Manual Feature Checks and Transaction Support

<SwmSnippet path="/django/db/backends/__init__.py" line="346">

---

In <SwmToken path="django/db/backends/__init__.py" pos="346:3:3" line-data="    def confirm(self):">`confirm`</SwmToken>, we flag the backend as confirmed and check transaction support so we know how to handle database operations going forward.

```python
    def confirm(self):
        "Perform manual checks of any database features that might vary between installs"
        self._confirmed = True
        self.supports_transactions = self._supports_transactions()
```

---

</SwmSnippet>

## Checking Transaction Capability

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start transaction support test"]
    click node1 openCode "django/db/backends/__init__.py:353:355"
    node1 --> node2["Running SQL Statements on the Database"]
    
    node2 --> node3["Running SQL Statements on the Database"]
    
    node3 --> node4["Rollback transaction"]
    click node4 openCode "django/db/backends/__init__.py:359:359"
    node4 --> node5["Running SQL Statements on the Database"]
    
    node5 --> node6{"Is count == 0?"}
    
    node6 -->|"Yes (count == 0)"| node7["Transactions are supported"]
    click node7 openCode "django/db/backends/__init__.py:364:364"
    node6 -->|"No (count != 0)"| node8["Transactions are NOT supported"]
    click node8 openCode "django/db/backends/__init__.py:364:364"
    node7 --> node9["Running SQL Statements on the Database"]
    node8 --> node9
    
    node9 --> node10["Committing Changes to the Database"]
    
    node10 --> node11["Return result"]
    click node11 openCode "django/db/backends/__init__.py:364:364"
    node11 --> node12["End"]
    click node12 openCode "django/db/backends/__init__.py:364:364"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node2 goToHeading "Running SQL Statements on the Database"
node2:::HeadingStyle
click node3 goToHeading "Running SQL Statements on the Database"
node3:::HeadingStyle
click node5 goToHeading "Running SQL Statements on the Database"
node5:::HeadingStyle
click node6 goToHeading "Fetching and Processing Query Results"
node6:::HeadingStyle
click node9 goToHeading "Running SQL Statements on the Database"
node9:::HeadingStyle
click node10 goToHeading "Committing Changes to the Database"
node10:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start transaction support test"]
%%     click node1 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:353:355"
%%     node1 --> node2["Running SQL Statements on the Database"]
%%     
%%     node2 --> node3["Running SQL Statements on the Database"]
%%     
%%     node3 --> node4["Rollback transaction"]
%%     click node4 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:359:359"
%%     node4 --> node5["Running SQL Statements on the Database"]
%%     
%%     node5 --> node6{"Is count == 0?"}
%%     
%%     node6 -->|"Yes (count == 0)"| node7["Transactions are supported"]
%%     click node7 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:364:364"
%%     node6 -->|"No (count != 0)"| node8["Transactions are NOT supported"]
%%     click node8 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:364:364"
%%     node7 --> node9["Running SQL Statements on the Database"]
%%     node8 --> node9
%%     
%%     node9 --> node10["Committing Changes to the Database"]
%%     
%%     node10 --> node11["Return result"]
%%     click node11 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:364:364"
%%     node11 --> node12["End"]
%%     click node12 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:364:364"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node2 goToHeading "Running SQL Statements on the Database"
%% node2:::HeadingStyle
%% click node3 goToHeading "Running SQL Statements on the Database"
%% node3:::HeadingStyle
%% click node5 goToHeading "Running SQL Statements on the Database"
%% node5:::HeadingStyle
%% click node6 goToHeading "Fetching and Processing Query Results"
%% node6:::HeadingStyle
%% click node9 goToHeading "Running SQL Statements on the Database"
%% node9:::HeadingStyle
%% click node10 goToHeading "Committing Changes to the Database"
%% node10:::HeadingStyle
```

<SwmSnippet path="/django/db/backends/__init__.py" line="353">

---

In <SwmToken path="django/db/backends/__init__.py" pos="353:3:3" line-data="    def _supports_transactions(self):">`_supports_transactions`</SwmToken>, we grab a cursor from the connection. This lets us run SQL statements to probe if the database can handle transactions, since we need to actually interact with the backend to verify its capabilities.

```python
    def _supports_transactions(self):
        "Confirm support for transactions"
        cursor = self.connection.cursor()
```

---

</SwmSnippet>

### Choosing the Right Cursor Type

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Request database cursor"] --> node2{"Is use_debug_cursor True?"}
    click node1 openCode "django/db/backends/__init__.py:247:248"
    node2 -->|"Yes"| node3["Create debug-enabled cursor"]
    click node2 openCode "django/db/backends/__init__.py:248:249"
    node2 -->|"No"| node4{"Is use_debug_cursor None and settings.DEBUG True?"}
    click node4 openCode "django/db/backends/__init__.py:249:250"
    node4 -->|"Yes"| node3
    node4 -->|"No"| node5["Create standard cursor"]
    click node3 openCode "django/db/backends/__init__.py:250:251"
    click node5 openCode "django/db/backends/__init__.py:252:253"
    node3 --> node6["Return cursor"]
    node5 --> node6["Return cursor"]
    click node6 openCode "django/db/backends/__init__.py:253:253"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Request database cursor"] --> node2{"Is <SwmToken path="django/db/backends/__init__.py" pos="248:6:6" line-data="        if (self.use_debug_cursor or">`use_debug_cursor`</SwmToken> True?"}
%%     click node1 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:247:248"
%%     node2 -->|"Yes"| node3["Create debug-enabled cursor"]
%%     click node2 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:248:249"
%%     node2 -->|"No"| node4{"Is <SwmToken path="django/db/backends/__init__.py" pos="248:6:6" line-data="        if (self.use_debug_cursor or">`use_debug_cursor`</SwmToken> None and <SwmToken path="django/db/backends/__init__.py" pos="249:12:14" line-data="            (self.use_debug_cursor is None and settings.DEBUG)):">`settings.DEBUG`</SwmToken> True?"}
%%     click node4 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:249:250"
%%     node4 -->|"Yes"| node3
%%     node4 -->|"No"| node5["Create standard cursor"]
%%     click node3 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:250:251"
%%     click node5 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:252:253"
%%     node3 --> node6["Return cursor"]
%%     node5 --> node6["Return cursor"]
%%     click node6 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:253:253"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/django/db/backends/__init__.py" line="247">

---

<SwmToken path="django/db/backends/__init__.py" pos="247:3:3" line-data="    def cursor(self):">`cursor`</SwmToken> picks between a debug cursor and a standard wrapped cursor depending on whether debugging is enabled. This lets us get more info during development, but keeps things lean in production. The next step is to use the cursor to interact with the database backend, like <SwmPath>[django/…/sqlite3/base.py](django/db/backends/sqlite3/base.py)</SwmPath>, depending on which backend is in use.

```python
    def cursor(self):
        if (self.use_debug_cursor or
            (self.use_debug_cursor is None and settings.DEBUG)):
            cursor = self.make_debug_cursor(self._cursor())
        else:
            cursor = util.CursorWrapper(self._cursor(), self)
        return cursor
```

---

</SwmSnippet>

### Getting a Backend-Specific Cursor

See <SwmLink doc-title="Preparing the Database Cursor with Moderation and Signal Handling">[Preparing the Database Cursor with Moderation and Signal Handling](.swm%5Cpreparing-the-database-cursor-with-moderation-and-signal-handling.nkjdc1kf.sw.md)</SwmLink>

### Setting Up Transaction Test Table

<SwmSnippet path="/django/db/backends/__init__.py" line="356">

---

After getting the cursor in <SwmToken path="django/db/backends/__init__.py" pos="349:9:9" line-data="        self.supports_transactions = self._supports_transactions()">`_supports_transactions`</SwmToken>, we create a table called <SwmToken path="django/db/backends/__init__.py" pos="356:10:10" line-data="        cursor.execute(&#39;CREATE TABLE ROLLBACK_TEST (X INT)&#39;)">`ROLLBACK_TEST`</SwmToken>. This sets up a controlled environment to check if the database can handle rollbacks and commits as expected. Next, we use management logic to interact with the database for further checks.

```python
        cursor.execute('CREATE TABLE ROLLBACK_TEST (X INT)')
```

---

</SwmSnippet>

### Running SQL Statements on the Database

See <SwmLink doc-title="Executing management commands">[Executing management commands](.swm%5Cexecuting-management-commands.8gqaaj6i.sw.md)</SwmLink>

### Committing Test Changes

<SwmSnippet path="/django/db/backends/__init__.py" line="357">

---

After executing the SQL to create the test table in <SwmToken path="django/db/backends/__init__.py" pos="349:9:9" line-data="        self.supports_transactions = self._supports_transactions()">`_supports_transactions`</SwmToken>, we commit the change. This makes sure the table exists before we run the actual transaction tests.

```python
        self.connection._commit()
```

---

</SwmSnippet>

### Committing Changes to the Database

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node0["Start commit operation"]
    click node0 openCode "django/db/backends/__init__.py:44:44"
    node0 --> node1{"Is there a database connection?"}
    click node1 openCode "django/db/backends/__init__.py:45:45"
    node1 -->|"Yes"| node2["Commit transaction"]
    click node2 openCode "django/db/backends/__init__.py:46:46"
    node1 -->|"No"| node3["Return without changes"]
    click node3 openCode "django/db/backends/__init__.py:45:45"
    node2 --> node4["End"]
    node3 --> node4
    click node4 openCode "django/db/backends/__init__.py:46:46"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node0["Start commit operation"]
%%     click node0 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:44:44"
%%     node0 --> node1{"Is there a database connection?"}
%%     click node1 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:45:45"
%%     node1 -->|"Yes"| node2["Commit transaction"]
%%     click node2 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:46:46"
%%     node1 -->|"No"| node3["Return without changes"]
%%     click node3 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:45:45"
%%     node2 --> node4["End"]
%%     node3 --> node4
%%     click node4 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:46:46"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/django/db/backends/__init__.py" line="44">

---

<SwmToken path="django/db/backends/__init__.py" pos="44:3:3" line-data="    def _commit(self):">`_commit`</SwmToken> just calls commit on the underlying connection if it's available. This makes sure any changes are saved, which is needed before we check rollback behavior in transaction tests.

```python
    def _commit(self):
        if self.connection is not None:
            return self.connection.commit()
```

---

</SwmSnippet>

### Finalizing and Resetting State After Commit

<SwmSnippet path="/django/db/backends/__init__.py" line="197">

---

In <SwmToken path="django/db/backends/__init__.py" pos="197:3:3" line-data="    def commit(self):">`commit`</SwmToken>, we delegate the actual commit to <SwmToken path="django/db/backends/__init__.py" pos="201:3:3" line-data="        self._commit()">`_commit`</SwmToken>, then handle any extra cleanup like resetting the dirty flag. This keeps the commit logic focused and reusable.

```python
    def commit(self):
        """
        Does the commit itself and resets the dirty flag.
        """
        self._commit()
```

---

</SwmSnippet>

<SwmSnippet path="/django/db/backends/__init__.py" line="202">

---

After <SwmToken path="django/db/backends/__init__.py" pos="44:3:3" line-data="    def _commit(self):">`_commit`</SwmToken> finishes in <SwmToken path="django/db/backends/__init__.py" pos="46:7:7" line-data="            return self.connection.commit()">`commit`</SwmToken>, we reset the dirty flag with <SwmToken path="django/db/backends/__init__.py" pos="202:3:3" line-data="        self.set_clean()">`set_clean`</SwmToken>. This tells the system there are no unsaved changes left, so we're ready for new operations.

```python
        self.set_clean()
```

---

</SwmSnippet>

### Testing Rollback and Data Integrity

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Insert test data into table"] --> node2["Perform rollback"]
  click node1 openCode "django/db/backends/__init__.py:358:359"
  node2 --> node3["Check count of test data in table"]
  click node2 openCode "django/db/backends/__init__.py:359:360"
  node3 --> node4{"Is count of test data zero after rollback?"}
  click node3 openCode "django/db/backends/__init__.py:360:361"
  node4 -->|"Yes"| node5["Database supports transactions"]
  click node4 openCode "django/db/backends/__init__.py:361:361"
  node4 -->|"No"| node6["Database does NOT support transactions"]
  click node5 openCode "django/db/backends/__init__.py:361:361"
  click node6 openCode "django/db/backends/__init__.py:361:361"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Insert test data into table"] --> node2["Perform rollback"]
%%   click node1 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:358:359"
%%   node2 --> node3["Check count of test data in table"]
%%   click node2 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:359:360"
%%   node3 --> node4{"Is count of test data zero after rollback?"}
%%   click node3 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:360:361"
%%   node4 -->|"Yes"| node5["Database supports transactions"]
%%   click node4 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:361:361"
%%   node4 -->|"No"| node6["Database does NOT support transactions"]
%%   click node5 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:361:361"
%%   click node6 openCode "<SwmPath>[django/…/backends/\__init_\_.py](django/db/backends/__init__.py)</SwmPath>:361:361"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/django/db/backends/__init__.py" line="358">

---

After committing in <SwmToken path="django/db/backends/__init__.py" pos="349:9:9" line-data="        self.supports_transactions = self._supports_transactions()">`_supports_transactions`</SwmToken>, we insert a row, roll back, and then check the row count. This sequence tests if the database really undoes changes when rolling back, which is what we care about for transaction support.

```python
        cursor.execute('INSERT INTO ROLLBACK_TEST (X) VALUES (8)')
        self.connection._rollback()
        cursor.execute('SELECT COUNT(X) FROM ROLLBACK_TEST')
```

---

</SwmSnippet>

<SwmSnippet path="/django/db/backends/__init__.py" line="361">

---

After rolling back in <SwmToken path="django/db/backends/__init__.py" pos="349:9:9" line-data="        self.supports_transactions = self._supports_transactions()">`_supports_transactions`</SwmToken>, we fetch the count from the test table. This tells us if the rollback worked—if the count is zero, the database supports transactions as expected.

```python
        count, = cursor.fetchone()
```

---

</SwmSnippet>

### Fetching and Processing Query Results

<SwmSnippet path="/django/db/backends/oracle/base.py" line="666">

---

<SwmToken path="django/db/backends/oracle/base.py" pos="666:3:3" line-data="    def fetchone(self):">`fetchone`</SwmToken> gets a row and hands it off for type conversion so Django gets the right Python types.

```python
    def fetchone(self):
        row = self.cursor.fetchone()
        if row is None:
            return row
        return _rowfactory(row, self.cursor)
```

---

</SwmSnippet>

### Type Casting Database Results

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start: Receive database row"]
    click node1 openCode "django/db/backends/oracle/base.py:713:751"
    subgraph loop1["For each value in the row"]
        node2{"Is value a number?"}
        click node2 openCode "django/db/backends/oracle/base.py:718:747"
        node2 -->|"Yes"| node3{"Check precision, scale, decimal point"}
        click node3 openCode "django/db/backends/oracle/base.py:719:746"
        node3 -->|"scale == -127 and precision == 0"| node4["Convert to int or decimal"]
        click node4 openCode "django/db/backends/oracle/base.py:721:728"
        node3 -->|"scale == -127 and precision > 0"| node5["Convert to float"]
        click node5 openCode "django/db/backends/oracle/base.py:729:732"
        node3 -->|"precision > 0 and scale == 0"| node6["Convert to int"]
        click node6 openCode "django/db/backends/oracle/base.py:733:737"
        node3 -->|"precision > 0 and scale != 0"| node7["Convert to decimal"]
        click node7 openCode "django/db/backends/oracle/base.py:738:739"
        node3 -->|"No type info, decimal point present"| node8["Convert to decimal"]
        click node8 openCode "django/db/backends/oracle/base.py:740:744"
        node3 -->|"No type info, no decimal point"| node9["Convert to int"]
        click node9 openCode "django/db/backends/oracle/base.py:745:746"
        node2 -->|"No"| node10{"Is value a string?"}
        click node10 openCode "django/db/backends/oracle/base.py:747:749"
        node10 -->|"Yes"| node11["Convert to unicode"]
        click node11 openCode "django/db/backends/oracle/base.py:749:749"
    end
    loop1 --> node12["Return tuple of converted values"]
    click node12 openCode "django/db/backends/oracle/base.py:751:751"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start: Receive database row"]
%%     click node1 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:713:751"
%%     subgraph loop1["For each value in the row"]
%%         node2{"Is value a number?"}
%%         click node2 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:718:747"
%%         node2 -->|"Yes"| node3{"Check precision, scale, decimal point"}
%%         click node3 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:719:746"
%%         node3 -->|"scale == -127 and precision == 0"| node4["Convert to int or decimal"]
%%         click node4 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:721:728"
%%         node3 -->|"scale == -127 and precision > 0"| node5["Convert to float"]
%%         click node5 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:729:732"
%%         node3 -->|"precision > 0 and scale == 0"| node6["Convert to int"]
%%         click node6 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:733:737"
%%         node3 -->|"precision > 0 and scale != 0"| node7["Convert to decimal"]
%%         click node7 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:738:739"
%%         node3 -->|"No type info, decimal point present"| node8["Convert to decimal"]
%%         click node8 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:740:744"
%%         node3 -->|"No type info, no decimal point"| node9["Convert to int"]
%%         click node9 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:745:746"
%%         node2 -->|"No"| node10{"Is value a string?"}
%%         click node10 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:747:749"
%%         node10 -->|"Yes"| node11["Convert to unicode"]
%%         click node11 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:749:749"
%%     end
%%     loop1 --> node12["Return tuple of converted values"]
%%     click node12 openCode "<SwmPath>[django/…/oracle/base.py](django/db/backends/oracle/base.py)</SwmPath>:751:751"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/django/db/backends/oracle/base.py" line="713">

---

In <SwmToken path="django/db/backends/oracle/base.py" pos="713:2:2" line-data="def _rowfactory(row, cursor):">`_rowfactory`</SwmToken>, we loop through each value and its description, converting database types to Python types using metadata like precision and scale. This is <SwmToken path="django/db/backends/__init__.py" pos="54:7:9" line-data="        A hook for backend-specific changes required when entering manual">`backend-specific`</SwmToken> and makes sure Django gets the right types for its ORM.

```python
def _rowfactory(row, cursor):
    # Cast numeric values as the appropriate Python type based upon the
    # cursor description, and convert strings to unicode.
    casted = []
    for value, desc in zip(row, cursor.description):
        if value is not None and desc[1] is Database.NUMBER:
            precision, scale = desc[4:6]
            if scale == -127:
                if precision == 0:
                    # NUMBER column: decimal-precision floating point
                    # This will normally be an integer from a sequence,
                    # but it could be a decimal value.
                    if '.' in value:
                        value = Decimal(value)
                    else:
                        value = int(value)
                else:
                    # FLOAT column: binary-precision floating point.
                    # This comes from FloatField columns.
                    value = float(value)
            elif precision > 0:
                # NUMBER(p,s) column: decimal-precision fixed point.
                # This comes from IntField and DecimalField columns.
                if scale == 0:
                    value = int(value)
                else:
                    value = Decimal(value)
            elif '.' in value:
                # No type information. This normally comes from a
                # mathematical expression in the SELECT list. Guess int
                # or Decimal based on whether it has a decimal point.
                value = Decimal(value)
            else:
                value = int(value)
        elif desc[1] in (Database.STRING, Database.FIXED_CHAR,
                         Database.LONG_STRING):
            value = to_unicode(value)
        casted.append(value)
```

---

</SwmSnippet>

<SwmSnippet path="/django/db/backends/oracle/base.py" line="750">

---

After casting all the values in <SwmToken path="django/db/backends/oracle/base.py" pos="670:3:3" line-data="        return _rowfactory(row, self.cursor)">`_rowfactory`</SwmToken>, we return them as a tuple. This gives Django a set of Python-typed values ready for use, assuming the cursor description matches expectations.

```python
        casted.append(value)
    return tuple(casted)
```

---

</SwmSnippet>

### Cleaning Up After Transaction Test

<SwmSnippet path="/django/db/backends/__init__.py" line="362">

---

After checking the row count in <SwmToken path="django/db/backends/__init__.py" pos="349:9:9" line-data="        self.supports_transactions = self._supports_transactions()">`_supports_transactions`</SwmToken>, we drop the test table to clean up. This avoids cluttering the database with temporary tables used for capability checks.

```python
        cursor.execute('DROP TABLE ROLLBACK_TEST')
```

---

</SwmSnippet>

<SwmSnippet path="/django/db/backends/__init__.py" line="363">

---

After dropping the test table in <SwmToken path="django/db/backends/__init__.py" pos="349:9:9" line-data="        self.supports_transactions = self._supports_transactions()">`_supports_transactions`</SwmToken>, we commit to finalize the cleanup. Then we return whether the rollback actually removed the inserted row, which tells us if transactions are supported.

```python
        self.connection._commit()
        return count == 0
```

---

</SwmSnippet>

## Checking Additional Database Capabilities

<SwmSnippet path="/django/db/backends/__init__.py" line="350">

---

After checking transaction support in <SwmToken path="django/db/backends/__init__.py" pos="346:3:3" line-data="    def confirm(self):">`confirm`</SwmToken>, we check if the backend supports standard deviation and foreign key introspection. This tells Django what ORM features are available for this connection.

```python
        self.supports_stddev = self._supports_stddev()
        self.can_introspect_foreign_keys = self._can_introspect_foreign_keys()
```

---

</SwmSnippet>

<SwmSnippet path="/django/db/backends/__init__.py" line="376">

---

<SwmToken path="django/db/backends/__init__.py" pos="376:3:3" line-data="    def _can_introspect_foreign_keys(self):">`_can_introspect_foreign_keys`</SwmToken> just returns True, meaning Django assumes all backends can introspect foreign keys except for <SwmToken path="django/db/backends/__init__.py" pos="378:18:18" line-data="        # Every database can do this reliably, except MySQL,">`MySQL`</SwmToken>'s <SwmToken path="django/db/backends/__init__.py" pos="379:15:15" line-data="        # which can&#39;t do it for MyISAM tables">`MyISAM`</SwmToken> tables, as noted in the comment.

```python
    def _can_introspect_foreign_keys(self):
        "Confirm support for introspected foreign keys"
        # Every database can do this reliably, except MySQL,
        # which can't do it for MyISAM tables
        return True
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBUHl0aG9uRGphbmdvJTNBJTNBR29waW5hdGhyZWRkeTY2" repo-name="PythonDjango"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
