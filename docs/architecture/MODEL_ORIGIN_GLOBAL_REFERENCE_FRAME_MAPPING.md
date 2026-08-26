# Сопоставление Model Origin и глобальной системы координат

**Проект:** CADConverter  
**Направление:** КОМПАС-3D → SOLIDWORKS  
**Статус:** Draft  
**Приоритет:** P0  
**Идентификатор сопоставления:** RG-001  
**Область:** детали (`.m3d → .sldprt`)

---

## 1. Цель

Этот документ определяет архитектурные правила сопоставления глобальной системы координат модели при конвертации детали КОМПАС-3D в деталь SOLIDWORKS.

В область документа входят:

- начало координат модели;
- глобальная декартова система координат;
- оси X/Y/Z;
- три базовые плоскости;
- стратегия доступа к объектам через API исходной и целевой CAD;
- представление в Semantic CAD IR;
- разрешение системных ссылок на стороне SOLIDWORKS;
- правила валидации.

Это сопоставление должно быть определено до начала конвертации эскизов и 3D-операций, поскольку последующие элементы зависят от глобальной системы координат.

---

## 2. Архитектурный принцип

Начало координат и базовая reference geometry не рассматриваются как обычные пользовательские операции дерева построения.

Они представляются как единая семантическая системная сущность:

```text
GlobalReferenceFrame
 ├─ Origin
 ├─ AxisX
 ├─ AxisY
 ├─ AxisZ
 ├─ PlaneXY
 ├─ PlaneXZ
 └─ PlaneYZ
```

Конвертер не должен создавать эти элементы в SOLIDWORKS как новые пользовательские reference features.

Правильная стратегия:

```text
Системная ссылка КОМПАС
        ↓
Каноническая семантическая идентичность
        ↓
Существующая системная ссылка SOLIDWORKS
```

Такой подход классифицируется как **semantic reuse**.

---

## 3. КОМПАС-3D: исходное сопоставление

### 3.1 Основной API

Основным API для исходной стороны является КОМПАС-3D API 7.

Системные reference entities предполагается получать через:

```text
IPart7.DefaultObject(...)
```

с использованием соответствующих значений `ksObj3dTypeEnum`.

### 3.2 Ожидаемые системные объекты

| Семантическая сущность | Идентификатор КОМПАС API |
|---|---|
| Начало координат | `o3d_pointCS` |
| Ось X | `o3d_axisOX` |
| Ось Y | `o3d_axisOY` |
| Ось Z | `o3d_axisOZ` |
| Плоскость XY | `o3d_planeXOY` |
| Плоскость XZ | `o3d_planeXOZ` |
| Плоскость YZ | `o3d_planeYOZ` |

Ожидаемый шаблон API:

```csharp
IModelObject origin =
    part.DefaultObject(ksObj3dTypeEnum.o3d_pointCS);

IModelObject planeXY =
    part.DefaultObject(ksObj3dTypeEnum.o3d_planeXOY);

IModelObject axisX =
    part.DefaultObject(ksObj3dTypeEnum.o3d_axisOX);
```

> **Статус проверки:** точные имена API, сигнатуры и значения enum должны быть повторно проверены на актуальном установленном КОМПАС SDK перед реализацией.

COM identity, внутренний указатель на объект или runtime-only identifier КОМПАС не должны попадать в CAD IR.

---

## 4. SOLIDWORKS: целевое сопоставление

Новая деталь SOLIDWORKS уже содержит системные reference entities:

```text
Origin
Front Plane
Top Plane
Right Plane
```

Конвертер должен переиспользовать эти системные объекты.

Ожидаемое семантическое соответствие:

| Каноническая плоскость | Системная плоскость SOLIDWORKS |
|---|---|
| Plane XY | Front Plane |
| Plane XZ | Top Plane |
| Plane YZ | Right Plane |

### 4.1 Ограничение

Production-код не должен полагаться только на отображаемые или локализованные имена:

```text
"Front Plane"
"Top Plane"
"Right Plane"
```

Причины:

- язык интерфейса SOLIDWORKS может отличаться;
- корпоративные шаблоны могут быть локализованы;
- пользователь может переименовать элементы;
- enterprise-шаблоны могут не сохранять стандартные display names.

Необходим production-resolver, определяющий системные элементы семантически и геометрически.

---

## 5. Canonical CAD IR

### 5.1 Семантическая идентичность глобальных ссылок

```csharp
public enum GlobalReferenceKind
{
    Origin,

    AxisX,
    AxisY,
    AxisZ,

    PlaneXY,
    PlaneXZ,
    PlaneYZ
}
```

Системные ссылки должны храниться по семантической идентичности, а не по runtime ID исходной CAD.

Пример:

```csharp
public sealed record GlobalReference(
    GlobalReferenceKind Kind);
```

Эскиз на глобальной плоскости XY должен ссылаться так:

```text
Sketch.Support =
    GlobalReference(PlaneXY)
```

а не так:

```text
Sketch.Support =
    SourceObjectId = 123456
```

---

## 6. Каноническая система координат

CAD IR должен использовать независимую от конкретной CAD правую декартову систему координат.

```csharp
public sealed record CoordinateFrame3D(
    Vector3 Origin,
    Vector3 XAxis,
    Vector3 YAxis,
    Vector3 ZAxis);
```

Каноническая система:

```text
Origin = (0, 0, 0)

X = (1, 0, 0)
Y = (0, 1, 0)
Z = (0, 0, 1)
```

Инвариант:

```text
X × Y = Z
```

Каноническая система принадлежит CADConverter, а не КОМПАС или SOLIDWORKS.

Это предотвращает зависимость Semantic CAD IR от исходной CAD.

---

## 7. Conversion Context

Контекст конвертации должен явно поддерживать преобразования:

```csharp
public sealed class CadConversionContext
{
    public CoordinateTransform SourceToCanonical { get; init; }

    public CoordinateTransform CanonicalToTarget { get; init; }

    public GlobalReferenceMap GlobalReferences { get; init; }
}
```

Для ожидаемого случая КОМПАС → SOLIDWORKS оба преобразования могут оказаться единичными:

```text
SourceToCanonical = Identity
CanonicalToTarget = Identity
```

Но архитектура не должна жёстко это предполагать.

Это потребуется позже для:

- локальных систем координат;
- transform сборок;
- imported bodies;
- зеркальной геометрии;
- видов чертежей.

---

## 8. Target Global Reference Resolver

Необходим отдельный resolver на стороне SOLIDWORKS.

Предлагаемый контракт:

```csharp
public interface ISolidWorksGlobalReferenceResolver
{
    object Resolve(GlobalReferenceKind kind);
}
```

Конкретная реализация должна возвращать корректный native SOLIDWORKS system entity для текущего документа.

Resolver не должен зависеть только от имени feature.

Ожидаемая production-стратегия:

```text
1. Обойти начальные системные/reference features
2. Определить reference-plane feature types
3. Получить геометрическую ориентацию / transform
4. Сопоставить с канонической семантической идентичностью
5. Закэшировать mapping на время жизни документа
```

---

## 9. Совпадения плоскости недостаточно

Для CAD-конвертации плоскость не определяется только геометрическим уравнением.

Для корректного переноса эскизов необходимо сохранить:

```text
Origin
X direction
Y direction
Normal
Handedness
```

Пример риска:

```text
Исходная плоскость:
X →
Y ↑
```

и:

```text
Целевая плоскость:
X ←
Y ↑
```

Геометрически это может быть одна и та же плоскость, но перенесённый эскиз окажется зеркальным.

Поэтому validation должна сравнивать полный basis, а не только нормаль плоскости.

---

## 10. Правила сопоставления

### RG-001.1 Origin

```text
КОМПАС:
o3d_pointCS

CAD IR:
GlobalReferenceKind.Origin

SOLIDWORKS:
существующий system Origin
```

**Стратегия:** Exact semantic reuse  
**Создание target feature:** Нет  
**Fallback:** Запрещён

---

### RG-001.2 Axis X

```text
КОМПАС:
o3d_axisOX

CAD IR:
GlobalReferenceKind.AxisX

SOLIDWORKS:
глобальное направление X
```

**Стратегия:** Exact semantic mapping

---

### RG-001.3 Axis Y

```text
КОМПАС:
o3d_axisOY

CAD IR:
GlobalReferenceKind.AxisY

SOLIDWORKS:
глобальное направление Y
```

**Стратегия:** Exact semantic mapping

---

### RG-001.4 Axis Z

```text
КОМПАС:
o3d_axisOZ

CAD IR:
GlobalReferenceKind.AxisZ

SOLIDWORKS:
глобальное направление Z
```

**Стратегия:** Exact semantic mapping

---

### RG-001.5 Plane XY

```text
КОМПАС:
o3d_planeXOY

CAD IR:
GlobalReferenceKind.PlaneXY

SOLIDWORKS:
system Front Plane
```

**Стратегия:** Exact semantic reuse

---

### RG-001.6 Plane XZ

```text
КОМПАС:
o3d_planeXOZ

CAD IR:
GlobalReferenceKind.PlaneXZ

SOLIDWORKS:
system Top Plane
```

**Стратегия:** Exact semantic reuse

---

### RG-001.7 Plane YZ

```text
КОМПАС:
o3d_planeYOZ

CAD IR:
GlobalReferenceKind.PlaneYZ

SOLIDWORKS:
system Right Plane
```

**Стратегия:** Exact semantic reuse

---

## 11. Запись Feature Mapping Matrix

| Поле | Значение |
|---|---|
| ID | `RG-001` |
| Name | Global Coordinate Frame |
| Source CAD | KOMPAS-3D |
| Target CAD | SOLIDWORKS |
| Source API | `IPart7.DefaultObject(...)` |
| Source entities | `o3d_pointCS`, `o3d_axisOX/OY/OZ`, `o3d_planeXOY/XOZ/YOZ` |
| Target model | System Origin + default reference planes |
| Target feature type | `RefPlane` для плоскостей |
| Mapping strategy | Semantic reuse |
| Quality | Exact |
| Create target feature | Нет |
| Parametric history | Full |
| Topology resolver | Не требуется |
| Global reference resolver | Требуется |
| Geometry fallback | Запрещён |
| Priority | P0 |

---

## 12. Требования к валидации

### V1 — Origin

Проверить:

```text
KOMPAS Origin = (0,0,0)
SOLIDWORKS Origin = (0,0,0)
```

в канонической системе координат конвертера.

### V2 — Сопоставление осей

Проверить:

```text
KOMPAS X ↔ SOLIDWORKS X
KOMPAS Y ↔ SOLIDWORKS Y
KOMPAS Z ↔ SOLIDWORKS Z
```

### V3 — Handedness

Обе системы должны удовлетворять:

```text
X × Y = Z
```

Левая или зеркальная система координат должна приводить к ошибке validation.

### V4 — Соответствие плоскостей

Проверить:

```text
XOY ↔ XY ↔ Front Plane
XOZ ↔ XZ ↔ Top Plane
YOZ ↔ YZ ↔ Right Plane
```

### V5 — Ориентация эскиза

Использовать асимметричную тестовую геометрию для выявления скрытого зеркалирования или поворота.

Не использовать только симметричные элементы:

```text
окружность
квадрат
центрированный прямоугольник
```

так как они могут скрыть ошибку ориентации.

---

## 13. Эталонная тестовая модель

Рекомендуемая тестовая деталь КОМПАС:

```text
Plane XOY
  ↓
Asymmetric Sketch A

Plane XOZ
  ↓
Asymmetric Sketch B

Plane YOZ
  ↓
Asymmetric Sketch C
```

Каждый эскиз должен содержать:

- однозначно выраженное положительное направление X;
- однозначно выраженное положительное направление Y;
- разные размеры по направлениям;
- хотя бы одно смещение относительно Origin;
- несимметричную форму.

Критерии приёмки:

- отсутствует зеркалирование;
- отсутствует неожиданный поворот на 90°/180°;
- выбрана правильная support plane;
- Origin совпадает;
- положительные направления осей совпадают.

---

## 14. Подтверждённые или ожидаемые API-факты

Следующие положения рассматриваются как API-факты, но должны оставаться трассируемыми к официальной документации производителя:

1. КОМПАС предоставляет доступ к системной reference geometry через API 7.
2. `IPart7.DefaultObject(...)` является предполагаемым механизмом доступа к default model objects.
3. В КОМПАС существуют семантические сущности:
   - Origin;
   - OX/OY/OZ axes;
   - XOY/XOZ/YOZ planes.
4. Деталь SOLIDWORKS содержит system Origin и три default reference planes.
5. Reference planes SOLIDWORKS представлены как reference-plane features.

---

## 15. Требует проверки на актуальном SDK

Перед production-реализацией необходимо проверить на конкретных целевых версиях CAD.

### КОМПАС

- точную сигнатуру `IPart7.DefaultObject(...)`;
- точные enum names;
- числовые значения enum;
- фактический возвращаемый COM interface;
- доступность стабильной геометрической информации для системных осей;
- точные правила orientation глобальной системы координат.

### SOLIDWORKS

- production-grade способ определить default system planes без использования локализованных имён;
- точные feature type strings для поддерживаемых версий;
- надёжный API-путь получения plane transform/origin/basis;
- нужен ли native SOLIDWORKS object для Origin или достаточно математического target-frame origin;
- orientation локальной системы эскиза на каждой системной плоскости.

---

## 16. Архитектурные решения проекта

Следующие положения являются архитектурными решениями CADConverter, а не API-фактами производителей:

1. CADConverter использует каноническую правую систему координат XYZ.
2. `GlobalReferenceFrame` является first-class сущностью Semantic CAD IR.
3. Системные reference entities идентифицируются через `GlobalReferenceKind`.
4. Runtime COM identity не сериализуется в CAD IR.
5. Существующие системные ссылки SOLIDWORKS переиспользуются, а не создаются заново.
6. Для системных reference entities geometry fallback запрещён.
7. Production mapping не может зависеть только от локализованных имён.
8. Полная validation включает basis orientation и handedness.
9. Asymmetric sketch tests обязательны для acceptance RG-001.
10. `RG-001` имеет приоритет P0 и должен быть реализован до конвертации эскизов и 3D features.

---

## 17. Открытые вопросы

До проверки на актуальных SDK остаются открытыми:

1. Точный production-grade алгоритм идентификации системных плоскостей SOLIDWORKS.
2. Должны ли глобальные оси иметь native target feature handle или могут оставаться математическими направлениями внутри target adapter.
3. Нужен ли native SOLIDWORKS selection object для system Origin в downstream operations.
4. Полностью ли совпадает sketch-local basis КОМПАС и SOLIDWORKS на всех трёх базовых плоскостях.
5. Требуется ли unit/transform normalization даже при identity mapping глобальных систем координат.

---

## 18. Acceptance Criteria для RG-001

`RG-001` считается принятым только если:

- system Origin КОМПАС корректно разрешается в canonical Origin;
- все три глобальные оси сопоставлены корректно;
- все три базовые плоскости КОМПАС сопоставлены с ожидаемыми default planes SOLIDWORKS;
- дополнительные эквивалентные reference planes в SOLIDWORKS не создаются;
- handedness сохранён;
- асимметричные эскизы на всех трёх плоскостях конвертируются без зеркалирования и поворота;
- mapping не зависит от языка UI;
- source COM object identity отсутствует в serialized CAD IR;
- mapping покрыт автоматизированным или воспроизводимым integration test.
