# Правила подготовки архитектурных документов CADConverter

**Проект:** CADConverter  
**Направление:** КОМПАС-3D → SOLIDWORKS  
**Статус:** Нормативный документ  
**Назначение:** единые правила анализа, оформления и сопровождения архитектурной документации  
**Язык документации:** русский

---

## 1. Назначение

Этот документ определяет обязательные правила создания архитектурных, аналитических и тестовых документов проекта CADConverter.

Все последующие документы должны создаваться на основании этих правил.

Цель — обеспечить:

- единый формат архитектурной документации;
- трассируемость от первоисточника до решения;
- разделение фактов, предположений и архитектурных решений;
- воспроизводимость анализа;
- проверяемость API-сопоставлений;
- возможность использовать документы как техническое основание для реализации и тестирования.

---

## 2. Язык документации

Вся документация репозитория CADConverter ведётся **на русском языке**.

На английском допускаются:

- названия API;
- имена интерфейсов;
- классы;
- enum;
- методы;
- свойства;
- типы features;
- идентификаторы;
- устойчивые технические термины, перевод которых снижает точность.

Пример:

```text
КОМПАС API 7
IPart7.DefaultObject(...)
SOLIDWORKS IFeature
RefPlane
GlobalReferenceKind
Semantic CAD IR
```

Объяснение и архитектурный текст при этом должны быть на русском языке.

---

## 3. Основной принцип документации

Каждый архитектурный документ должен отвечать на пять вопросов:

1. **Что существует в исходной CAD?**
2. **Как это доступно через API?**
3. **Как это представляется в Semantic CAD IR?**
4. **Как это восстанавливается в целевой CAD?**
5. **Как доказать, что перенос выполнен корректно?**

Базовый поток анализа:

```text
КОМПАС
  ↓
Source API
  ↓
Semantic Analysis
  ↓
CAD IR
  ↓
Mapping Strategy
  ↓
SOLIDWORKS API
  ↓
Validation
  ↓
Acceptance Criteria
```

---

## 4. Типы утверждений

Каждое важное утверждение в документе должно относиться к одной из категорий.

### 4.1 Подтверждённый API-факт

Факт, подтверждённый:

- официальной документацией производителя;
- SDK;
- официальным примером API;
- экспериментом на установленной версии CAD;
- исходным кодом официального примера.

Пример:

```text
Подтверждено официальной документацией:
IPart7.DefaultObject(...) используется для получения default model objects.
```

### 4.2 Требует проверки SDK

Информация выглядит корректной, но перед реализацией должна быть проверена на актуальной версии SDK.

Пример:

```text
Требует проверки:
точная сигнатура метода;
числовое значение enum;
возвращаемый COM interface.
```

### 4.3 Вывод / inference

Логический вывод, сделанный на основании наблюдений или нескольких источников.

Он не должен оформляться как подтверждённый факт.

Пример:

```text
Вывод:
вероятно, mapping может быть реализован без создания новой target feature.
```

### 4.4 Архитектурное решение CADConverter

Решение проекта, которое не является поведением КОМПАС или SOLIDWORKS.

Пример:

```text
Архитектурное решение:
все длины в Semantic CAD IR хранятся в метрах.
```

### 4.5 Открытый вопрос

Нерешённый вопрос, который должен быть закрыт до соответствующего этапа реализации.

---

## 5. Приоритет источников

Использовать источники в следующем порядке.

### P1 — официальный SDK и официальная API-документация

Основной источник для API mapping.

### P2 — официальные примеры производителя

Используются для подтверждения реального API usage.

### P3 — реальные тесты на установленной CAD

Используются для проверки фактического поведения версии.

### P4 — документация существующих CAD-конвертеров

Используется как архитектурный precedent, но не как доказательство API-поведения обратного направления.

### P5 — статьи, форумы, сторонние примеры

Используются только как вспомогательные материалы.

---

## 6. Обязательная трассируемость

Любое значимое API-утверждение должно иметь возможность быть связано с первоисточником.

Для каждого источника желательно фиксировать:

```text
Source ID
Производитель
Название
Тип источника
Версия
Дата
URL / путь
Какие утверждения подтверждает
Статус проверки
```

Документ не должен смешивать:

```text
Vendor fact
```

и:

```text
CADConverter decision
```

---

# 7. Идентификаторы Mapping

Каждая анализируемая сущность должна иметь постоянный идентификатор.

Рекомендуемая схема:

```text
RG     Reference Geometry
SK     Sketch
SK-GEO Sketch Geometry
SK-CON Sketch Constraints
FT     Solid Feature
SM     Sheet Metal
ASM    Assembly
DRW    Drawing
EXP    Expressions
TOP    Topology
VAL    Validation
```

Примеры:

```text
RG-001 Global Coordinate Frame
RG-002 Default Reference Plane
SK-001 Sketch
SK-GEO-001 Circle
FT-001 Extrusion
SM-001 Base Sheet
```

Идентификатор после публикации не переиспользуется для другой сущности.

---

# 8. Статус документа

Каждый документ должен содержать статус:

```text
Draft
Verified
Accepted
Deprecated
Superseded
```

### Draft

Анализ продолжается.

### Verified

API-факты проверены, но архитектурное решение ещё может обсуждаться.

### Accepted

Документ утверждён как основание для реализации.

### Deprecated

Не использовать для новой реализации.

### Superseded

Заменён другим документом.

---

# 9. Приоритет реализации

Каждый mapping должен иметь приоритет.

```text
P0 — необходим для первого работающего vertical slice
P1 — необходим для MVP
P2 — необходим для расширенной функциональности
P3 — optional / future
```

---

# 10. Классификация качества mapping

Для каждого mapping обязательно указывать уровень соответствия.

### Exact

Исходная семантика может быть воспроизведена практически напрямую.

### Equivalent

Целевая feature отличается внутренне, но сохраняет design intent.

### Reconstructed

Исходная feature восстанавливается несколькими target features.

### Partial

Сохраняется только часть семантики.

### GeometryOnly

Сохраняется геометрия без полноценной параметрики.

### Unsupported

Корректная конвертация пока не поддерживается.

Нельзя использовать статус `Exact`, если он не подтверждён validation.

---

# 11. Обязательная структура каждого Mapping-документа

Каждый документ должен по возможности иметь следующие разделы.

## 11.1 Заголовок

```text
Название
Project
Direction
Status
Priority
Mapping ID
Scope
```

## 11.2 Цель

Что именно анализируется и зачем.

## 11.3 Область действия

Что входит и что не входит в текущий mapping.

## 11.4 Исходная семантика КОМПАС

Описание сущности с точки зрения CAD-модели.

## 11.5 КОМПАС API

Необходимо зафиксировать:

- API generation;
- интерфейсы;
- методы;
- enum;
- типы объектов;
- зависимости;
- ограничения;
- что требует SDK verification.

## 11.6 Семантическая модель CAD IR

Должно быть определено, какие данные нужны независимо от CAD API.

Пример:

```csharp
Circle2D
{
    CenterX
    CenterY
    Radius
}
```

Запрещено включать в Semantic CAD IR:

- COM objects;
- runtime pointers;
- CAD-specific interop types;
- process-local object references.

## 11.7 Целевая семантика SOLIDWORKS

Что должно появиться в целевой модели.

## 11.8 SOLIDWORKS API

Зафиксировать:

- interfaces;
- methods;
- feature type;
- selection requirements;
- document context;
- rebuild requirements;
- ограничения.

## 11.9 Mapping strategy

Указать:

```text
Exact
Equivalent
Reconstructed
Partial
GeometryOnly
Unsupported
```

Также описать алгоритм:

```text
Source
  ↓
IR
  ↓
Target
```

## 11.10 Dependencies

Какие mapping должны быть реализованы раньше.

Пример:

```text
SK-GEO-001 Circle
depends on:
RG-001
RG-002
SK-001
UNITS
SKETCH_COORDINATE_FRAME
```

## 11.11 Units

Если feature содержит числа, обязательно определить единицы.

По умолчанию проект должен стремиться к canonical SI representation.

## 11.12 Coordinate Systems

Для геометрии обязательно определить:

- глобальную систему координат;
- локальную систему feature;
- transformation rules;
- handedness;
- axis orientation.

## 11.13 References

Если feature зависит от других объектов:

- plane;
- face;
- edge;
- axis;
- point;
- sketch entity;

необходимо определить способ reference resolution.

## 11.14 Error Policy

Определить поведение при:

- отсутствующей зависимости;
- ambiguous reference;
- invalid parameters;
- unsupported feature;
- failed target rebuild.

## 11.15 Fallback Policy

Fallback должен быть явным.

Запрещён:

```text
silent fallback
```

Если используется STEP/BREP или GeometryOnly, это должно быть записано в результате конвертации.

## 11.16 Validation

Необходимо описать, как проверяется результат.

## 11.17 Reference Test Model

Каждая новая поддерживаемая сущность должна иметь минимальную тестовую модель.

## 11.18 Acceptance Criteria

Условия, при которых mapping считается реализованным.

## 11.19 Подтверждённые факты

Отдельный список verified facts.

## 11.20 Требует проверки

Отдельный список SDK/runtime verification.

## 11.21 Архитектурные решения

Что утверждено именно проектом CADConverter.

## 11.22 Открытые вопросы

Что ещё не решено.

---

# 12. Semantic CAD IR

IR должен отражать CAD-семантику, а не копировать API КОМПАС.

Неправильно:

```text
KompasSketchObject
KompasEntityType
KompasFeaturePointer
```

Правильно:

```text
SketchIR
Circle2D
ReferencePlane
ExtrusionFeature
SheetMetalDefinition
```

IR должен быть:

- serializable;
- versioned;
- CAD-independent;
- testable без запуска CAD;
- пригоден для диагностики.

---

# 13. Versioning IR

Версия IR должна быть явной.

Пример:

```text
CAD-IR 1.0
```

Breaking change требует изменения версии schema.

Документ mapping должен указывать, для какой IR schema он актуален.

---

# 14. Feature Dependencies

Feature tree нельзя рассматривать только как линейный список.

Необходимо анализировать dependency graph.

Пример:

```text
Plane
  ↓
Sketch
  ↓
Extrusion
  ↓
Generated Face
  ↓
Sketch2
```

Для каждого feature желательно фиксировать:

```text
FeatureOrder
Dependencies[]
References[]
```

---

# 15. Reference Resolution

Ссылки на topology нельзя строить только на индексах.

Запрещён production-подход:

```text
Face #3
Edge #12
```

Для topology должна существовать semantic/geometric resolution strategy.

В случае неоднозначности:

```text
Ambiguous
```

должно приводить к diagnostic, а не к случайному выбору.

---

# 16. Validation Layers

По мере развития converter validation должна делиться на уровни.

## 16.1 Structural

Проверка структуры feature tree.

## 16.2 Geometric

Проверка:

- координат;
- размеров;
- площади;
- объёма;
- bounding box;
- center of mass.

## 16.3 Parametric

Проверка:

- размеров;
- constraints;
- expressions;
- suppression;
- dependencies.

## 16.4 Behavioral

Изменение driving parameter и проверка rebuild behavior.

Для первого PoC допускается использовать только необходимый поднабор.

---

# 17. Test-first Documentation

Каждый mapping должен иметь тест ещё до начала production-реализации.

Для теста фиксируются:

```text
Test ID
Source model
Input values
Expected target tree
Expected numeric values
Tolerance
Expected mapping quality
Acceptance criteria
```

Пример:

```text
TEST-001
Circle on XOY

CenterX = 20 mm
CenterY = 10 mm
Radius  = 5 mm
```

---

# 18. Numeric Tolerance

Для численных проверок запрещено использовать прямое сравнение floating-point значений.

Документ должен определять tolerance.

Пример:

```text
abs(source - target) <= epsilon
```

Значение epsilon должно быть обосновано типом данных и API.

---

# 19. Units and Numeric Conventions

Все feature с числовыми параметрами должны анализироваться с учётом единиц.

Документация должна явно указывать:

```text
Source unit
IR unit
Target API unit
Conversion
```

Желательное canonical правило:

```text
Length → meter
Angle  → radian
```

Окончательное правило утверждается отдельным архитектурным документом.

---

# 20. Naming

Файлы архитектурных документов именуются:

```text
UPPER_SNAKE_CASE.md
```

Примеры:

```text
MODEL_ORIGIN_GLOBAL_REFERENCE_FRAME_MAPPING.md
DEFAULT_REFERENCE_PLANE_MAPPING.md
SKETCH_MAPPING.md
SKETCH_CIRCLE_MAPPING.md
```

Тестовые документы:

```text
TEST_001_CIRCLE_ON_DEFAULT_PLANE.md
```

---

# 21. Правила анализа API

Перед утверждением mapping необходимо проверить минимум:

### КОМПАС

- как найти feature;
- как получить параметры;
- как получить references;
- какие interface/version differences существуют;
- что является transient COM object;
- что является stable semantic data.

### SOLIDWORKS

- как создать target entity;
- какие selections required;
- какой document state required;
- какие units использует API;
- требуется ли edit mode;
- требуется ли rebuild;
- как прочитать созданную entity обратно.

---

# 22. Обратное чтение результата

По возможности после создания target entity converter должен уметь прочитать созданный объект обратно через SOLIDWORKS API.

Паттерн:

```text
IR
 ↓
Write SOLIDWORKS
 ↓
Read SOLIDWORKS
 ↓
Compare
```

Это предпочтительнее проверки только результата вызова create-method.

---

# 23. Запрет Silent Success

Конвертация не может считаться успешной только потому, что target document был сохранён.

Успех должен соответствовать фактическому уровню:

```text
Exact
Equivalent
Reconstructed
Partial
GeometryOnly
```

Если часть семантики потеряна, это должно быть отражено в diagnostics.

---

# 24. Diagnostic Requirements

Mapping должен предусматривать диагностические коды.

Пример:

```text
RG001_REFERENCE_NOT_FOUND
SK001_SUPPORT_UNRESOLVED
SKGEO001_INVALID_RADIUS
SKGEO001_TARGET_CREATE_FAILED
VAL001_NUMERIC_MISMATCH
```

Формат кодов может быть уточнён отдельным документом.

---

# 25. Правила изменения документа

При изменении архитектурного решения необходимо:

1. изменить соответствующий документ;
2. обновить status/version;
3. обновить связанные mapping;
4. обновить test documents;
5. при breaking architecture change создать ADR;
6. не оставлять старое решение как неявно действующее.

---

# 26. Когда требуется ADR

ADR создаётся, если решение:

- влияет на несколько mapping;
- меняет границы подсистем;
- вводит новый architectural invariant;
- меняет IR;
- меняет fallback policy;
- меняет process architecture;
- влияет на совместимость.

Примеры:

```text
Canonical units = SI
Separate CAD worker processes
Semantic CAD IR
Topology resolver strategy
```

---

# 27. Definition of Ready для реализации

Mapping готов к реализации, если:

- определена source semantic;
- найдены необходимые source API;
- определена IR representation;
- определена target semantic;
- найдены target API;
- определены dependencies;
- определены units;
- определены coordinate frames;
- определена mapping strategy;
- определён test model;
- определены validation checks;
- определены acceptance criteria;
- открытые вопросы не блокируют реализацию.

---

# 28. Definition of Done для mapping

Mapping считается завершённым, если:

- implementation существует;
- test model успешно конвертируется;
- target entity является native и редактируемой в рамках заявленного quality level;
- validation проходит;
- diagnostics реализованы;
- documentation соответствует фактическому поведению;
- API assumptions подтверждены либо явно оставлены version-specific;
- acceptance criteria выполнены.

---

# 29. Минимальная последовательность документов для первого vertical slice

Для первого теста переноса окружности рекомендуется следующий порядок:

```text
1. MODEL_ORIGIN_GLOBAL_REFERENCE_FRAME_MAPPING.md
2. DEFAULT_REFERENCE_PLANE_MAPPING.md
3. UNITS_AND_NUMERIC_CONVENTIONS.md
4. SKETCH_COORDINATE_SYSTEM_MAPPING.md
5. SKETCH_MAPPING.md
6. SKETCH_CIRCLE_MAPPING.md
7. TEST_001_CIRCLE_ON_DEFAULT_PLANE.md
```

Только после утверждения этих документов следует переходить к production-реализации `TEST-001`.

---

# 30. Основной invariant проекта

Главный принцип CADConverter:

> Конвертер должен переносить не только геометрию, а максимально возможную CAD-семантику и design intent. Любая потеря семантики должна быть известна, классифицирована, проверяема и отражена в результате конвертации.
