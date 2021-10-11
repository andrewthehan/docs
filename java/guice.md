# Guice

## Purpose

Imagine there is a `Soldier` that wants to use a `Sword`.

```java
class Sword {
  private final int damage;

  public Sword(int damage) {
    this.damage = damage;
  }

  void attack(Entity entity) {
    ...
  }
}
```

```java
class Soldier {
  private final Sword sword;

  public Soldier() {
    this.sword = new Sword(10);
  }
}
```

This design presents several problems.

### 1. All soldiers have a sword that deals the same damage.

To solve this, we can have callers pass in the desired `Sword` through the constructor.

```java
class Soldier extends Entity {
  private final Sword sword;

  public Soldier(Sword sword) {
    this.sword = sword;
  }
}
```

Now callers can construct any number of soldiers with different swords.

```java
Soldier infantry = new Soldier(new Sword(5));
Soldier commander = new Soldier(new Sword(10));
```

This is known as **dependency injection**. Any dependencies of an object are _injected_ into the instance.

### 2. All soldiers use the same weapon.

What if soldiers want to use an `Axe` or a `Bow`? Soldiers are coupled directly with the weapon type.

Instead, an interface should be defined that describes what the soldier needs and soldier should depend on this.

```java
interface Weapon {
  void attack(Entity entity);
}

class Soldier extends Entity {
  private final Weapon weapon;

  public Soldier(Weapon weapon) {
    this.weapon = weapon;
  }
}
```

Now soldiers can be constructed with any weapon.

```java
new Soldier(new Sword(10 /* damage */));
new Soldier(new Axe(8 /* damage */));
new Soldier(new Bow(3 /* damage */, .8 /* accuracy */));
```

This is known as **dependency inversion**. Objects are decoupled from the actual implementation and only depend on higher level abstractions.

## Modules

Dependency injection and dependency inversion allow decoupling of objects allowing for better code reuse and flexibility. However, this often leads to extensive boilerplate for various factories and long constructor invocations.

```java
class SwordFactory extends WeaponFactory {
  static Weapon generate() {
    return new Sword(...);
  }
}
```

```java
new Soldier(SwordFactory.generate(), ChainMailFactory.generate(), BronzeHelmetFactory.generate());
```

Instead we can use Java annotations and Guice to simplify this.

```java
class Soldier {
  @Inject
  public Soldier(Weapon weapon, Armor armor, Helmet helmet) {
    ...
  }
}
```

```java
class SwordModule extends AbstractModule {
  @Provides
  Weapon providesSword() {
    return new Sword(...);
  }
}
```

`SwordModule` binds a `Sword` instance to the `Weapon` type.

Modules can also define bindings using a different syntax.

```java
class SwordModule extends AbstractModule {
  @Override
  void configure() {
    bind(Weapon.class).toInstance(new Sword(...));
  }
}
```

Modules can provide multiple bindings.

```java
class RangerModule extends AbstractModule {
  @Provides
  Weapon providesWeapon() {
    return new Bow(...);
  }

  @Provides
  Armor providesArmor() {
    return new LeatherMantle(...);
  }

  @Provides
  Helmet providesHelmet() {
    return new Hood(...);
  }
}
```

## Binding Annotations

Imagine we wanted to provide a different set of weapon/armor/helmet for different soldiers.

```java
class MagicianModule extends AbstractModule {
  @Provides
  Weapon providesWeapon() {
    return new Wand(...);
  }

  @Provides
  Armor providesArmor() {
    return new Coat(...);
  }

  @Provides
  Helmet providesHelmet() {
    return new MagicianHat(...);
  }
}
```

```java
class Ranger extends Soldier {
  @Inject
  public Ranger(Weapon weapon, Armor armor, Helmet helmet) {
    ...
  }
}
```

```java
class Magician extends Soldier {
  @Inject
  public Magician(Weapon weapon, Armor armor, Helmet helmet) {
    ...
  }
}
```

This leads to ambiguity on which binding to inject into each soldier type. Bindings can be qualified using a `@Qualifier` annotation.

```java
@Qualifier
@Target({ FIELD, PARAMETER, METHOD })
@Retention(RUNTIME)
public @interface Ranger {}

@Qualifier
@Target({ FIELD, PARAMETER, METHOD })
@Retention(RUNTIME)
public @interface Magician {}
```

These annotations should be declared on the binding and injection point.

```java
class RangerModule extends AbstractModule {
  @Provides
  @Ranger
  Weapon providesWeapon() {
    return new Bow(...);
  }

  @Provides
  @Ranger
  Armor providesArmor() {
    return new LeatherMantle(...);
  }

  @Provides
  @Ranger
  Helmet providesHelmet() {
    return new Hood(...);
  }
}
```

```java
class Ranger extends Soldier {
  @Inject
  public Ranger(@Ranger Weapon weapon, @Ranger Armor armor, @Ranger Helmet helmet) {
    ...
  }
}
```

### Multibindings

Imagine you wanted to create an archive view of all weapon types. This can be achieved using a set binder.

```java
class RangerModule extends AbstractModule {
  @ProvidesIntoSet
  @Ranger
  Weapon providesWeapon() {
    return new Bow(...);
  }
}
```

```java
class MagicianModule extends AbstractModule {
  @ProvidesIntoSet
  @Magician
  Weapon providesWeapon() {
    return new Wand(...);
  }
}
```

Now these can be injected directly into the weapon archive.

```java
class WeaponArchive {
  @Inject
  public WeaponArchive(Set<Weapon> weapons) {
    ...
  }
}
```

A map binder can also be used to inject multiple bindings with a custom key.

```java
enum WeaponType {
  SWORD,
  BOW,
  WAND,
}
```

```java
@MapKey(unwrapValue = true)
@Retention(RUNTIME)
public @interface WeaponTypeKey {
  WeaponType value();
}
```

```java
class RangerModule extends AbstractModule {
  @WeaponTypeKey(WeaponType.BOW)
  @ProvidesIntoMap
  @Ranger
  Weapon providesWeapon() {
    return new Bow(...);
  }
}
```

```java
class MagicianModule extends AbstractModule {
  @WeaponTypeKey(WeaponType.WAND)
  @ProvidesIntoMap
  @Magician
  Weapon providesWeapon() {
    return new Wand(...);
  }
}
```

And this can be injected like:

```java
class WeaponArchive {
  @Inject
  public WeaponArchive(Map<WeaponType, Weapon> weapons) {
    ...
  }
}
```