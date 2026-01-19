# Ревью проекта Симуляция для [@Vlad_VS92](https://github.com/Vlad06091992/java_roadmap_simulation)

## Хорошо:
1. Симуляция работает, прогресс отрисовывается
2. Класс Application(main) содержит только вызов метода `run()`
3. У проекта есть структура
4. Есть базовые entity от которых наследуется всё остальное

---

## Недостатки, которые следует исправить:

### 1. Нейминг, нейминг и ещё раз нейминг

Здесь хочется отметить две важные детали. Первая — игнорирование нейминга из ТЗ. Он сделан не просто так, пожалуйста исправь это.

Вторая — нарушен нейминг для классов:

**а)** [Application](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Application.java#L7) — по смыслу этот класс является классом Main, либо можно было продублировать название программы и назвать его SimulationApp. Класс Application может быть частью какой-то библиотеки или фреймворка, поэтому лучше использовать название, которое будет обозначать именно твоё приложение.

Сейчас:
```java
public class Application {
    public static void main(String[] args) throws InterruptedException {
        Simulation simulation = new Simulation();
        simulation.run(20,20,500);
    }
}
```

Лучше:
```java
public class SimulationApp {
    public static void main(String[] args) throws InterruptedException {
        Simulation simulation = new Simulation();
        simulation.run(20, 20, 500);
    }
}
```

**б)** [AliveEntity](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/entities/AliveEntity.java#L10) — в ТЗ чётко прописано название Creature и это отличное название, понятно что это класс, который отвечает за существ. Что такое AliveEntity? Может это Entity, которую не вычистили из хипа, или может что-то, что отвечает за сохранение состояния приложения? Поди знай, очень плохое название.

Сейчас:
```java
public abstract class AliveEntity extends Entity {
```

Лучше:
```java
public abstract class Creature extends Entity {
```

**в)** [Simulation](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Simulation.java#L20) — этот класс скорее должен быть назван SimulationRunner.

Сейчас:
```java
public class Simulation {
```

Лучше:
```java
public class SimulationRunner {
```

**Общая рекомендация:** если есть ТЗ, делаем как в ТЗ, если ТЗ нет, называем классы таким именем, чтобы по его названию было очевидно, что этот класс делает и что в себе содержит.

А так же нейминг пакетов, например в `entities` понятно, что хранится, но вот в пакете `data` совершенно не очевидно, что хранится. Я бы подумал над этим названием ещё раз, и назвал его в духе `helpers`, и уже стало бы понятнее, что там лежит ±. И туда же можно было бы поместить класс SimulationRunner из подпункта В. А так же пакет `statics` — я совершенно не понимаю что там лежит и как это работает. По этому неймингу я подумал, что там лежат классы с полностью статическими полями или хотя-бы константы, но это не так. Я бы переименовал этот пакет во что-то типа `environment` или, как ещё один вариант, отправил эти классы в пакет `helpers`.

---

### 2. Распределение по пакетам

С одной стороны это хорошо, что оно есть. С другой же — его можно и нужно улучшить:

**а)** Не дублировать название пакетов. У нас есть пакет `entities`, в которам лежат сущности существ, но в `herbivores` и `carnivores` у нас так же есть `entities`, что в целом может запутать. Было бы лучше создать пакет внутри `entities` с названием `base`, и базовые сущности поместить туда.

Сейчас:
```
entities/
├── AliveEntity.java
├── Entity.java
├── herbivores/
│   ├── Herbivore.java
│   └── entities/          <-- дублирование
│       ├── Cow.java
│       └── ...
└── predators/
    ├── Predator.java
    └── entities/          <-- дублирование
        ├── Wolf.java
        └── ...
```

Лучше:
```
entities/
├── base/
│   ├── Creature.java
│   └── Entity.java
├── herbivores/
│   ├── Herbivore.java
│   ├── Cow.java
│   └── ...
└── predators/
    ├── Predator.java
    ├── Wolf.java
    └── ...
```

**б)** Как я упоминал в прошлом пункте — подправить нейминг.

**в)** Твои классы AliveEntity и просто Entity можно было бы поместить в тот же пакет `entities -> base`

---

### 3. Класс [Simulation](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Simulation.java#L20)

**а)** Содержит метод `action()`, который ему не принадлежит. Я бы поместил метод `action()` в класс SimulationService.

**б)** Поскольку мы нигде больше не модифицируем список entities, можно сделать его `final`

Сейчас:
```java
private static ArrayList<Entity> entities = new ArrayList<>(...);
```

Лучше:
```java
private static final List<Entity> entities = new ArrayList<>(...);
```

**в)** Сам метод `action()` слишком большой. Его следовало бы разбить на подметоды, и вызывать уже их внутри метода `action()`.

Например для блока ["сущности меняют позиции"](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Simulation.java#L84-L103):

Сейчас:
```java
public void action() throws InterruptedException {
    // ... много кода выше ...
    
    //сущности меняют свои позиции
    for (int i = 0; i < entities.size(); i++) {
        Entity entity = entities.get(i);
        if (entity instanceof Predator) {
            predatorsCount++;
        }
        if (entity instanceof Herbivore) {
            herbivoresCount++;
        }
        if (entity instanceof Grass) {
            grassCount++;
        }
        if (entity instanceof AliveEntity) {
            ((AliveEntity) entity).run();
        }
    }
    
    // ... много кода ниже ...
}
```

Лучше:
```java
public void action() throws InterruptedException {
    removeDeadEntities();
    moveEntities();
    printStatistics();
    checkGameOver();
    field.showMap(entitiesMap);
}

private void moveEntities() {
    for (Entity entity : entities) {
        if (entity instanceof AliveEntity) {
            ((AliveEntity) entity).run();
        }
    }
}
```

[Блок с очисткой от погибших существ](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Simulation.java#L74-L82) можно вынести в метод `removeDeadEntities()`.

[Блок с проверкой оставшихся существ или еды](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Simulation.java#L111-L114) можно вынести в метод `isGameOver()`.

---

### 4. [AliveEntity](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/entities/AliveEntity.java#L10)

Асбтрактный класс содержит метод `run()`, который только переопределяется в других классах, имеет смысл сделать этот метод абстрактным. Метод `move()` должен называться `makeMove()`, читать первый пункт.

Сейчас:
```java
protected void move(Point targetPoint) {
    entitiesMap.remove(getPoint());
    super.setPoint(targetPoint);
    entitiesMap.put(getPoint(), this);
}
```

Лучше:
```java
protected void makeMove(Point targetPoint) {
    entitiesMap.remove(getPoint());
    super.setPoint(targetPoint);
    entitiesMap.put(getPoint(), this);
}
```

---

### 5. Убери `static` где не нужно

[В классе Simulation](https://github.com/Vlad06091992/java_roadmap_simulation/blob/3e3237a513332d0e7b4dbd9967b52c154af281b4/src/main/java/simulation/Simulation.java#L24-L26) поля объявлены как `static`, 
в твоём случае это не требуется, лучше убрать этот модификатор:

Сейчас:
```java
private static Field field;
private static final Map<Point, Entity> entitiesMap = new HashMap<>();
private static ArrayList<Entity> entities = new ArrayList<>(...);
private static final Utils helpers = new Utils();
```

Лучше:
```java
private Field field;
private final Map<Point, Entity> entitiesMap = new HashMap<>();
private final List<Entity> entities = new ArrayList<>(...);
private final Utils helpers = new Utils();
```

Почему? `static` означает, что поле принадлежит классу, а не объекту. Если ты создашь две симуляции, они будут использовать одни и те же данные. Используй `static` только для констант (например `private static final int MAX_SIZE = 100`). В целом это глобальная рекомендация, если ты можешь избежать использования слова `static` — не используй его.
