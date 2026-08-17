# Historical Reconstruction — Semantic Graph / Project IR

**Статус:** историческая реконструкция  
**Назначение:** восстановить ранее обсуждавшуюся структуру Semantic Graph до кристаллизации текущих Prototype Structures  
**Важно:** этот документ не является новой спецификацией и не заменяет `Semantic Graph Prototype Structures v2.3`.

---

## 1. Историческая точка, которую удалось восстановить

В предыдущем обсуждении была сформирована модель **Project Knowledge Model**, позднее переосмысленная как **Project IR (Intermediate Representation)**.

Исходная идея была:

```text
Разнородные артефакты проекта
        ↓
Semantic Graph / Project Knowledge Model
        ↓
единая семантическая модель проекта
```

Граф должен был объединять знания, которые в обычном проекте распределены между:

- исходным кодом;
- требованиями;
- архитектурными решениями;
- ограничениями;
- тестами;
- документацией;
- SQL;
- историей изменений;
- людьми и ролями;
- другими проектными артефактами.

---

# 2. Первая восстановленная структура

В первоначальной модели явно обсуждались следующие основные типы узлов.

```text
CodeEntity
Requirement
Decision
Constraint
Test
Person
Component
```

Смысл был не в том, чтобы создать узел для каждого файла.

Например:

```text
File
Class
Function
```

могли быть источниками для `CodeEntity`, но Semantic Graph должен был представлять семантически значимые сущности.

---

# 3. Исторически обсуждавшиеся Relations

Был сформирован набор типизированных семантических отношений:

```text
implements
depends_on
decided_by
constrains
tests
supersedes
```

Пример:

```text
Requirement
      ↓
implements
      ↓
CodeEntity
```

или:

```text
Decision
      ↓
constrains
      ↓
Component
```

или:

```text
Test
      ↓
tests
      ↓
CodeEntity
```

Таким образом, граф с самого начала задумывался не как обычный dependency graph.

---

# 4. Provenance

В ранней модели provenance уже рассматривался как обязательная часть семантики.

Для каждого знания должно быть возможно ответить:

```text
Откуда это известно?
Когда это было обнаружено?
Какой артефакт является источником?
Какая версия источника использовалась?
```

То есть:

```text
Semantic Entity
      ↓
Provenance
      ↓
Source Artifact
      ↓
Version / Time
```

Это решение является прямым предшественником нынешней модели:

```text
SourceSnapshot
SourceFact
Evidence
SemanticClaim
```

---

# 5. Temporal Versioning

Также обсуждалась временная модель.

Сущность не должна просто «перезаписываться».

Например:

```text
Requirement R1
    ↓
version 1
    ↓
version 2
```

или:

```text
Decision D1
    ↓
superseded by
    ↓
Decision D2
```

Это означает, что изменение знания должно сохранять историю.

Позднее эта идея получила более строгую форму через:

```text
Snapshot
ResearchPass
SUPERSEDED
parent_snapshot_id
```

---

# 6. Project IR

Следующим шагом модель была переосмыслена как **Project IR**.

Ключевая схема:

```text
Source Artifacts
      ↓
Extraction
      ↓
Reconciliation
      ↓
Project IR
      ↓
Materialized Views
```

То есть IR должен был быть промежуточным представлением, из которого можно получать различные представления проекта.

Например:

```text
Project IR
   ├── Agent Context
   ├── Memory
   ├── Documentation
   ├── RAG
   ├── Tests
   └── Viewer
```

---

# 7. Важное уточнение: IR не является Git и не заменяет Source of Truth

В ходе дальнейшего обсуждения была проведена важная граница.

Исходные проектные артефакты остаются редактируемыми источниками:

```text
Git
Documents
DB
Tests
Figma
SQL
etc.
```

Project IR является:

```text
derived
enriched
versioned
semantic / structural overlay
```

Он не должен становиться единственным Source of Truth проекта.

---

# 8. Проблема доверия к Semantic IR

Затем было установлено, что полностью семантический IR опасно считать абсолютно точным.

LLM и автоматические анализаторы могут:

- ошибаться;
- неправильно интерпретировать код;
- терять контекст;
- выдавать разные интерпретации одного факта.

Поэтому модель была смещена к:

```text
Structural Index
        +
Semantic Overlay
```

где semantic layer имеет:

```text
confidence
provenance
temporal information
```

а при недостатке уверенности допускается обращение к исходным источникам.

---

# 9. Критический архитектурный поворот

Позднее была сформулирована ещё более строгая модель:

```text
Observation / Fact Log
        ↓
Identity / Dedup Compiler
        ↓
Semantic Graph
        ↓
Materialized Views
```

То есть Semantic Graph перестал рассматриваться как первичный журнал истины.

Первичным слоем становится **append-only Observation/Fact log**.

Он сохраняет:

```text
что было обнаружено
откуда
когда
какой версией источника
```

А граф является производным представлением этих наблюдений.

---

# 10. Эта модель напрямую совпадает с нынешними Prototype Structures

Текущая цепочка:

```text
SourceSnapshot
      ↓
SourceFact
      ↓
Evidence
      ↓
SemanticClaim
      ↓
Frame / Relation
```

не появилась изолированно.

Она является развитием исторической модели:

```text
Source Artifact
      ↓
Observation / Fact
      ↓
Semantic Interpretation
      ↓
Graph
```

Причём нынешняя модель стала значительно строже в отношении epistemology.

---

# 11. Что было усилено нынешней методикой

Историческая модель в основном отвечала на вопрос:

> Как представить знания проекта?

Нынешняя методика дополнительно отвечает:

> Как доказать, что это знание действительно следует из источников?

Для этого появились:

```text
SourceFact
Evidence
SemanticClaim
confidence
CONFIRMED
INFERRED
UNKNOWN
Conflict
ResearchPass
```

То есть:

```text
Historical Project IR
        ↓
        + epistemic discipline
        ↓
Prototype Structures v2.3
```

---

# 12. Соотношение старых и нынешних структур

| Историческая модель | Текущая модель |
|---|---|
| Source Artifact | SourceSnapshot |
| Observation / Fact | SourceFact |
| Provenance | SourceSnapshot + Evidence |
| Semantic interpretation | SemanticClaim |
| Knowledge entity | Frame |
| Typed semantic edge | Relation |
| Version history | Snapshot + ResearchPass + superseding |
| Confidence | Claim confidence |
| Conflict | Conflict |
| Project IR | Semantic Graph projection |
| Derived views | будущие Graph Views |

Это не означает, что соответствие один-к-одному уже окончательно утверждено. Таблица является результатом исторической реконструкции.

---

# 13. Что изменилось концептуально

Историческая модель:

```text
Project Knowledge Model
```

была ориентирована на:

```text
что знает система о проекте
```

Текущая модель:

```text
Prototype Structures
```

ориентирована сначала на:

```text
что система может доказать о проекте
```

И только затем:

```text
что из этого можно кристаллизовать
в Semantic Graph.
```

Это важное развитие, а не противоречие.

---

# 14. Что нельзя переносить из старой модели автоматически

Исторические узлы:

```text
CodeEntity
Requirement
Decision
Constraint
Test
Person
Component
```

не следует просто вставлять в нынешний `Frame.type`.

Причина:

текущая модель ещё должна определить, какие из них являются:

```text
Frame
Capability
Artifact
Actor
Decision
Requirement
```

а какие должны быть специализированными представлениями.

Например:

```text
CodeEntity
```

может оказаться специализированным Frame над структурным Code Fact.

Но это ещё не принятое решение.

---

# 15. Что уже можно считать исторически устойчивым

По восстановленным обсуждениям устойчивыми являются следующие решения:

### 15.1. Граф должен быть многослойным

```text
Source
  ↓
Fact
  ↓
Semantic
  ↓
Graph
  ↓
Views
```

### 15.2. Provenance является частью модели

Знание без возможности установить источник неполноценно.

### 15.3. История изменений должна сохраняться

Нельзя просто переписывать старое знание.

### 15.4. Semantic Graph не должен быть единственным Source of Truth

Исходные артефакты сохраняют статус источников.

### 15.5. Semantic layer должен иметь epistemic status

В нынешней форме:

```text
CONFIRMED
INFERRED
UNKNOWN
```

### 15.6. Граф должен иметь materialized views

Один и тот же underlying knowledge должен быть доступен различным потребителям:

```text
AI Agent
Documentation
RAG
Viewer
Testing
Developer Context
```

---

# 16. Что осталось нерешённым

Не следует считать исторически утверждёнными:

```text
точный окончательный набор node types;
точный окончательный набор relation types;
точную физическую схему хранения;
graph database как обязательную БД;
точный способ materialized views;
точный способ identity resolution;
точный алгоритм дедупликации;
точный способ агрегации Claims в Frames.
```

Именно эти вопросы должны решаться при создании `Semantic Graph Structure v1`.

---

# 17. Связь с текущим Backend исследованием

Теперь текущий эксперимент можно расположить в общей архитектуре:

```text
SourceSnapshot
   │
   ├── Core Backend S0
   ├── DB S0
   └── Taxi Web S0
          │
          ↓
      Research Pass
          │
          ↓
      SourceFacts
          │
          ↓
       Evidence
          │
          ↓
    Semantic Claims
          │
          ↓
     Backend / Client
       Frames
          │
          ↓
    Semantic Graph v1
```

Таким образом, RP-01 и RP-02 уже являются не отдельными документами «про анализ кода», а первыми производителями данных для будущего графа.

---

# 18. Главный вывод реконструкции

Историческая архитектурная линия выглядит так:

```text
Project Knowledge Model
          ↓
Project IR
          ↓
Structural + Semantic Overlay
          ↓
Provenance + Versioning
          ↓
Observation / Fact Log
          ↓
Semantic Graph as Projection
          ↓
Materialized Views
```

А текущие Prototype Structures:

```text
SourceSnapshot
    ↓
SourceFact
    ↓
Evidence
    ↓
SemanticClaim
    ↓
Frame / Relation
    ↓
Candidate
```

являются **эпистемологически строгим исследовательским слоем**, который теперь позволяет безопасно получать такую Graph Structure.

---

# 19. Следующий архитектурный шаг

Не следует сейчас немедленно переписывать `Prototype Structures`.

Следует использовать эту реконструкцию как вход для следующего документа:

```text
Semantic_Graph_Structure_v1.md
```

Его задача — не описывать методику исследования и не описывать очередной Research Pass.

Он должен ответить на другой вопрос:

> Какую минимальную каноническую структуру должен иметь Semantic Graph, чтобы принять результаты Research Pass и сохранить их provenance, versioning и semantic relations?

И только после этого можно определить:

```text
Prototype Structures
        ↓
Graph Structure v1
        ↓
кристаллизация
        ↓
Machine-readable representation
```

