# Modul 1: Fondasi C# & Konsep OOP Utama

Pada modul pertama ini, kita akan mengulas tiga konsep dasar Object-Oriented Programming (OOP) dalam konteks pemrograman game menggunakan C#: **Encapsulation**, **Inheritance & Polymorphism**, serta **Abstraction & Interfaces**.

---

## 1. Encapsulation & Properties

Encapsulation (Pengkapsulan) membantu kita membungkus data dan variabel di dalam kelas, sekaligus membatasi akses langsung dari luar agar nilai variabel tersebut tidak diubah secara tidak sengaja.

### Contoh Pendekatan Sederhana

Pada bentuk kode berikut, variabel `health` dibuka secara publik. Kelas lain dapat mengubah angka ini secara bebas tanpa melalui validasi.

```csharp
using System;

public class PlayerHealthBad
{
    public float health = 100f;
    public bool isDead = false;
}

public class TrapBad
{
    public void TriggerTrap(PlayerHealthBad player)
    {
        // Variabel diubah langsung dari luar tanpa validasi batas minimal 0
        player.health -= 500f; 
    }
}
```

### Contoh Penerapan Encapsulation

Dengan memanfaatkan Property C# (`get; private set;`), perubahan variabel `CurrentHealth` hanya dapat dilakukan melalui method `TakeDamage()` yang memiliki validasi batas.

```csharp
using System;

public class PlayerHealthGood
{
    private readonly float maxHealth;
    
    public float CurrentHealth { get; private set; }
    public bool IsDead => CurrentHealth <= 0f;

    public event Action OnDeath;
    public event Action<float> OnHealthChanged;

    public PlayerHealthGood(float maxHealth = 100f)
    {
        this.maxHealth = maxHealth;
        CurrentHealth = maxHealth;
    }

    public void TakeDamage(float amount)
    {
        if (IsDead || amount <= 0f) return;

        CurrentHealth = Math.Max(0f, CurrentHealth - amount);
        OnHealthChanged?.Invoke(CurrentHealth);

        if (IsDead)
        {
            Die();
        }
    }

    private void Die()
    {
        Console.WriteLine("Player telah gugur.");
        OnDeath?.Invoke();
    }
}

public class TrapGood
{
    private readonly float trapDamage;

    public TrapGood(float damage = 50f)
    {
        trapDamage = damage;
    }

    public void TriggerTrap(PlayerHealthGood playerHealth)
    {
        playerHealth.TakeDamage(trapDamage);
    }
}
```

---

## 2. Inheritance & Polymorphism

Polymorphism memungkinkan kita memperlakukan berbagai kelas turunan melalui tipe kelas induk (*base class*) yang sama.

### Contoh Pendekatan Sederhana

Tanpa Polymorphism, penanganan berbagai tipe musuh biasanya menggunakan `enum` dan percabangan `switch-case`.

```csharp
using System;

public enum EnemyType { Melee, Ranged, Boss }

public class EnemyBad
{
    public EnemyType type;
    public float attackRange;

    public void ExecuteAttack()
    {
        switch (type)
        {
            case EnemyType.Melee:
                Console.WriteLine("Serangan jarak dekat.");
                break;
            case EnemyType.Ranged:
                Console.WriteLine("Menembakkan proyektil.");
                break;
            case EnemyType.Boss:
                Console.WriteLine("Menggunakan skill spesial boss.");
                break;
        }
    }
}
```

### Contoh Penerapan Polymorphism

Kita dapat membuat kelas abstrak `EnemyBase` dengan method `Attack()`, lalu menurunkan kelas spesifik seperti `MeleeEnemy` dan `RangedEnemy`.

```csharp
using System;
using System.Collections.Generic;

public abstract class EnemyBase
{
    public string Name { get; protected set; }
    protected float attackRange;
    protected float damage;

    public EnemyBase(string name, float attackRange, float damage)
    {
        Name = name;
        this.attackRange = attackRange;
        this.damage = damage;
    }

    public abstract void Attack();

    public virtual void MoveToPlayer()
    {
        Console.WriteLine($"{Name} bergerak mendekati player.");
    }
}

public class MeleeEnemy : EnemyBase
{
    public MeleeEnemy(string name) : base(name, 1.5f, 15f) { }

    public override void Attack()
    {
        Console.WriteLine($"{Name} menyerang dengan pedang (Damage: {damage}).");
    }
}

public class RangedEnemy : EnemyBase
{
    public RangedEnemy(string name) : base(name, 10f, 8f) { }

    public override void Attack()
    {
        Console.WriteLine($"{Name} menembak dari jarak jauh (Damage: {damage}).");
    }
}

public class EnemyWaveManager
{
    private readonly List<EnemyBase> activeEnemies = new List<EnemyBase>();

    public void AddEnemy(EnemyBase enemy) => activeEnemies.Add(enemy);

    public void CommandAllEnemiesToAttack()
    {
        foreach (var enemy in activeEnemies)
        {
            enemy.Attack(); 
        }
    }
}
```

---

## 3. Abstraction & Interfaces

Interface mendefinisikan *kontrak perilaku* yang dapat diimplementasikan oleh kelas manapun tanpa memandang hierarki kelas tersebut.

### Contoh Pendekatan Sederhana

Pengecekan tipe objek secara manual dengan pengkondisian `if-else` atau `as` casting.

```csharp
using System;

public class PlayerInteractionBad
{
    public void InteractWith(object target)
    {
        Door door = target as Door;
        if (door != null) { door.OpenDoor(); return; }

        Chest chest = target as Chest;
        if (chest != null) { chest.OpenChest(); return; }

        NPC npc = target as NPC;
        if (npc != null) { npc.Talk(); return; }
    }
}

public class Door { public void OpenDoor() => Console.WriteLine("Pintu terbuka."); }
public class Chest { public void OpenChest() => Console.WriteLine("Peti terbuka."); }
public class NPC { public void Talk() => Console.WriteLine("NPC berbicara."); }
```

### Contoh Penerapan Interface `IInteractable`

Dengan membuat interface `IInteractable`, logika interaksi player menjadi lebih ringkas dan tidak terikat pada kelas spesifik.

```csharp
using System;

public interface IInteractable
{
    string InteractionPrompt { get; }
    void Interact(object instigator);
}

public class InteractiveDoor : IInteractable
{
    public string InteractionPrompt => "Buka Pintu";

    public void Interact(object instigator)
    {
        Console.WriteLine("Pintu terbuka.");
    }
}

public class InteractiveChest : IInteractable
{
    public string InteractionPrompt => "Buka Peti";

    public void Interact(object instigator)
    {
        Console.WriteLine("Peti terbuka dan item didapatkan.");
    }
}

public class PlayerInteractionGood
{
    public void InteractWith(IInteractable interactable, object instigator)
    {
        Console.WriteLine($"Petunjuk: {interactable.InteractionPrompt}");
        interactable.Interact(instigator);
    }
}
```
