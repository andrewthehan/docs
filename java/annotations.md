# Annotations

## Purpose

Annotations are syntactic metadata. They can be used for:
 - information for the compiler (e.g. [`@Override`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Override.html), [`@Deprecated`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Deprecated.html))
 - compile-time and deployment-time processing (e.g. code generation like [`@AutoValue`](https://github.com/google/auto/tree/master/value))
 - runtime processing

## Syntax

`@<annotation name>`

`@<annotation name>(<content>)`

`@<annotation name>(<parameter name> = <content>(, ...))`

If there is only one parameter named `value`, it can be omitted during usage.

## Usage

Suppose we wanted to create an annotation that describes the necessary equipment pieces of an entity.

```java
enum EquipType {
  WEAPON,
  SHIELD,
  HELMET,
  ARMOR,
}
```

```java
public @interface DependsOn {
  EquipType value();
}
```

This annotation should only be applied to types (i.e. classes). `@Target` documentation [here](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html).

```java
@Target(ElementType.TYPE)
public @interface DependsOn {
  EquipType value();
}
```

This annotation should be available through reflection during runtime. `@Retention` documentation [here](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Retention.html).

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DependsOn {
  EquipType value();
}
```

This can now be applied to types.

```java
@DependsOn(EquipType.WEAPON)
class Soldier implements Entity {
  ...
}
```

This alone does nothing. Something must now process these annotations.

```java
static void checkEntityEquips(Entity entity, Map<EquipType, Equip> equips) {
  DependsOn dependency = entity.getClass().getAnnotation(DependsOn.class);
  if (dependency == null) {
    return;
  }

  EquipType equipType = dependency.value();
  if (equips.containsKey(equipType)) {
    return;
  }

  throw new MissingEquipException(entity, equipType);
}
```

Now if a soldier is missing their weapon, an exception will be thrown.

However, soldiers often depend on more than just a weapon. Soldiers should also have armor.

`@DependsOn` must be updated to be repeatable. `@Repeatable` documentation [here](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Repeatable.html).

First create the wrapper annotation. This should support the same element type targets and retention policies and must have an element named `value` supporting an array of the original annotation.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DependsOnRepeated {
  DependsOn[] value();
}
```

Now update the original annotation to specify its wrapper annotation.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Repeated(DependsOnRepeated.class)
public @interface DependsOn {
  EquipType value();
}
```

Now the soldier can specify all its dependencies.

```java
@DependsOn(EquipType.WEAPON)
@DependsOn(EquipType.ARMOR)
class Soldier implements Entity {
  ...
}
```

The logic processing this annotation must be updated.

```java
static void checkEntityEquips(Entity entity, Map<EquipType, Equip> equips) {
  DependsOn[] dependencies = entity.getClass().getAnnotationByType(DependsOn.class);
  for (DependsOn dependency : dependencies) {
    EquipType equipType = dependency.value();
    if (equips.containsKey(equipType)) {
      continue;
    }

    throw new MissingEquipException(entity, equipType);
  }
}
```