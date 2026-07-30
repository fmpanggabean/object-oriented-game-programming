# Modul 5: Arsitektur Game & Best Practices

Modul penutup ini membahas penataan arsitektur game tingkat lanjut: **Data-Driven Architecture**, **Perancangan Component System Mandiri (Composition over Inheritance)**, serta **Manajemen Memori & Alokasi Garbage Collector**.

---

## 1. Data-Driven Architecture dalam C#

Memisahkan data (seperti atribut karakter, statistik item, atau harga) dari logika utama program agar perubahan data tidak memerlukan pengubahan struktur kode.

### Contoh Pendekatan Sederhana

Statistik senjata ditulis keras (*hardcoded*) di dalam kelas logika.

```csharp
using System;

public class WeaponHardcoded
{
    public string weaponName = "Pedang";
    public float baseDamage = 150f;
    public float attackSpeed = 1.2f;
}
```

### Contoh Penerapan Data Container (POCO Data)

Data disimpan di dalam kelas wadah `WeaponData` yang dapat diisi dari file konfigurasi luar seperti JSON.

```csharp
using System;

public class WeaponData
{
    public string WeaponName { get; set; }
    public float BaseDamage { get; set; }
    public float AttackSpeed { get; set; }
}

public class WeaponHolder
{
    private readonly WeaponData data;

    public WeaponHolder(WeaponData data)
    {
        this.data = data;
    }

    public void PerformAttack()
    {
        Console.WriteLine($"Menyerang dengan {data.WeaponName} (Damage: {data.BaseDamage}).");
    }
}
```

---

## 2. Composition over Inheritance (Merancang Component System Mandiri)

Pendekatan **Component-Based Architecture** memungkinkan kita merancang entity game (`GameObject`) secara fleksibel dengan menggabungkan komponen-komponen terpisah tanpa perlu bergantung pada struktur pewarisan bertingkat yang kaku.

### Contoh Pendekatan Sederhana

Hierarki pewarisan bertingkat yang sulit disesuaikan ketika ada kebutuhan unit baru dengan kombinasi atribut berbeda.

```csharp
using System;

public class Entity { }
public class LivingEntity : Entity { public float hp; }
public class MovingEntity : LivingEntity { public float speed; }
public class AttackingEntity : MovingEntity { public float damage; }
public class HeroEntity : AttackingEntity { public string heroName; }
```

### Contoh Penerapan Component System dari Nol (Pure C#)

Implementasi sederhana wadah `GameObject` dan `IComponent` menggunakan C# Generics:

```csharp
using System;
using System.Collections.Generic;

// 1. Interface dasar untuk setiap komponen
public interface IComponent
{
    void Update(float deltaTime);
}

// 2. Wadah GameObject utama
public class GameObject
{
    public string Name { get; set; }
    private readonly List<IComponent> components = new List<IComponent>();

    public GameObject(string name)
    {
        Name = name;
    }

    public T AddComponent<T>(T component) where T : IComponent
    {
        components.Add(component);
        return component;
    }

    public T GetComponent<T>() where T : class, IComponent
    {
        foreach (var comp in components)
        {
            if (comp is T target) return target;
        }
        return null;
    }

    public void Update(float deltaTime)
    {
        foreach (var comp in components)
        {
            comp.Update(deltaTime);
        }
    }
}

// 3. Komponen-komponen independen
public class HealthComponent : IComponent
{
    public float CurrentHealth { get; private set; } = 100f;

    public void TakeDamage(float amount)
    {
        CurrentHealth = Math.Max(0, CurrentHealth - amount);
        Console.WriteLine($"[HealthComponent] Menerima damage {amount}. Sisa HP: {CurrentHealth}");
    }

    public void Update(float deltaTime) { }
}

public class MovementComponent : IComponent
{
    public float MoveSpeed { get; set; } = 5f;

    public void Update(float deltaTime)
    {
        Console.WriteLine($"[MovementComponent] Karakter bergerak dengan kecepatan {MoveSpeed}.");
    }
}

public class AttackComponent : IComponent
{
    public float AttackPower { get; set; } = 25f;

    public void Attack(GameObject target)
    {
        Console.WriteLine("[AttackComponent] Melakukan serangan ke target.");
        target.GetComponent<HealthComponent>()?.TakeDamage(AttackPower);
    }

    public void Update(float deltaTime) { }
}

// 4. Simulasi penggunaan di main program
public class Program
{
    public static void Main()
    {
        // Objek Player: Komposisi Gerak + HP + Serangan
        GameObject player = new GameObject("Player Hero");
        player.AddComponent(new MovementComponent { MoveSpeed = 10f });
        player.AddComponent(new HealthComponent());
        player.AddComponent(new AttackComponent { AttackPower = 35f });

        // Objek Tower: Komposisi HP + Serangan (Tanpa Gerak)
        GameObject tower = new GameObject("Defense Tower");
        tower.AddComponent(new HealthComponent());
        tower.AddComponent(new AttackComponent { AttackPower = 50f });

        Console.WriteLine("--- SIMULASI PENGUPDATEAN LOOP ---");
        player.Update(0.016f);
        tower.Update(0.016f);

        Console.WriteLine("\n--- SIMULASI INTERAKSI SERANGAN ---");
        player.GetComponent<AttackComponent>()?.Attack(tower);
    }
}
```

---

## 3. Manajemen Memori & Alokasi Garbage Collector

Beberapa hal yang perlu diperhatikan saat menulis C# untuk loop eksekusi utama agar tidak memicu alokasi memori acak secara terus-menerus.

### Contoh Pendekatan Sederhana

Penggabungan string berulang dan instansiasi array baru di dalam siklus loop.

```csharp
using System;

public class MemoryLeakBad
{
    public void UpdateTick()
    {
        // Penggabungan string di dalam loop menghasilkan alokasi string baru di memori heap
        string scoreText = "Score: " + 1000;

        // Instansiasi array baru di setiap tick
        int[] temporaryData = new int[100];
    }
}
```

### Contoh Penerapan Optimasi Memori

Menggunakan `StringBuilder` dan mendaur ulang array buffer yang sudah ada.

```csharp
using System;
using System.Text;

public class MemoryOptimizationGood
{
    private readonly StringBuilder stringBuilder = new StringBuilder();
    private readonly int[] reusableBuffer = new int[100];

    public void UpdateTick()
    {
        // Penggunaan StringBuilder untuk menghindari pembentukan string baru berulang
        stringBuilder.Clear();
        stringBuilder.Append("Score: ").Append(1000);

        // Membersihkan buffer tanpa mengalokasikan array baru
        Array.Clear(reusableBuffer, 0, reusableBuffer.Length);
    }
}
```
