# Modul 3: Fitur Lanjutan C# untuk Game Development

Modul ketiga ini mengulas beberapa fitur C# yang sering digunakan saat membangun sistem game: **Generics**, **Delegates & Events**, serta **Async/Await Task**.

---

## 1. Generics & Custom Data Structures

Generics memungkinkan kita membuat kelas atau method yang bisa bekerja dengan berbagai tipe data tanpa mengorbankan keamanan tipe (*type safety*) dan efisiensi memori.

### Contoh Pendekatan Sederhana

Menggunakan koleksi non-generic seperti `ArrayList` yang membutuhkan proses casting tipe data secara manual.

```csharp
using System;
using System.Collections;

public class InventoryBad
{
    private ArrayList items = new ArrayList();

    public void AddItem(object newItem)
    {
        items.Add(newItem);
    }

    public object GetItem(int index)
    {
        return items[index]; 
    }
}
```

### Contoh Penerapan Generics

Membuat kelas `GenericInventory<T>` dengan batasan tipe `where T : ItemBase`.

```csharp
using System;
using System.Collections.Generic;

public abstract class ItemBase
{
    public string ItemName { get; protected set; }
    public int Value { get; protected set; }

    public ItemBase(string name, int value)
    {
        ItemName = name;
        Value = value;
    }
}

public class HealthPotion : ItemBase
{
    public HealthPotion() : base("Ramuan Kesehatan", 50) { }
}

public class GenericInventory<T> where T : ItemBase
{
    private readonly List<T> items = new List<T>();

    public void AddItem(T newItem)
    {
        items.Add(newItem);
        Console.WriteLine($"Item ditambahkan: {newItem.ItemName}");
    }

    public T GetItem(int index)
    {
        if (index >= 0 && index < items.Count)
        {
            return items[index];
        }
        return null;
    }
}
```

---

## 2. Delegates, Events, & Action/Func

Delegates dan Events memfasilitasi komunikasi antar-komponen secara longgar (*loose coupling*) melalui mekanisme **Event-Driven**.

### Contoh Pendekatan Sederhana

Pemeriksaan status darah secara terus-menerus di setiap siklus game loop (*polling*).

```csharp
using System;

public class HealthUIBad
{
    public PlayerHealthBad playerHealth;

    public void RenderTick()
    {
        Console.WriteLine($"[Polling UI] HP Player saat ini: {playerHealth.health}");
    }
}
```

### Contoh Penerapan Event `System.Action`

Kelas `PlayerHealthEvent` memancarkan event `OnHealthChanged` hanya saat nilai darah berubah.

```csharp
using System;

public class PlayerHealthEvent
{
    private readonly float maxHealth;
    public float CurrentHealth { get; private set; }

    public event Action<float, float> OnHealthChanged;

    public PlayerHealthEvent(float maxHealth = 100f)
    {
        this.maxHealth = maxHealth;
        CurrentHealth = maxHealth;
    }

    public void TakeDamage(float amount)
    {
        CurrentHealth = Math.Max(0f, CurrentHealth - amount);
        OnHealthChanged?.Invoke(CurrentHealth, maxHealth);
    }
}

public class HealthUIGood
{
    public HealthUIGood(PlayerHealthEvent playerHealth)
    {
        playerHealth.OnHealthChanged += UpdateHealthBar;
    }

    private void UpdateHealthBar(float current, float max)
    {
        float ratio = current / max;
        Console.WriteLine($"[Update UI] Tampilan darah diperbarui: {ratio * 100}% ({current}/{max})");
    }
}
```

---

## 3. Async/Await & Timer Sederhana

Pengelolaan durasi waktu tunda (seperti cooldown skill) tanpa menghentikan eksekusi utama program.

### Contoh Pendekatan Sederhana

Memanggil `Thread.Sleep()` yang membuat seluruh aplikasi terhenti sementara.

```csharp
using System;
using System.Threading;

public class CooldownBad
{
    public void UseAbilityWithBlockingCooldown()
    {
        Console.WriteLine("Skill digunakan.");
        Thread.Sleep(3000); // Menghentikan eksekusi thread utama selama 3 detik
        Console.WriteLine("Skill siap kembali.");
    }
}
```

### Contoh Penerapan Async/Await

Memanfaatkan `async Task` dan `Task.Delay` untuk penanganan jeda secara non-blocking.

```csharp
using System;
using System.Threading.Tasks;

public class CooldownGood
{
    public bool IsOnCooldown { get; private set; }

    public async Task TriggerAbilityAsync(float durationSeconds)
    {
        if (IsOnCooldown)
        {
            Console.WriteLine("Skill masih dalam cooldown.");
            return;
        }

        IsOnCooldown = true;
        Console.WriteLine("Skill berhasil digunakan.");

        await Task.Delay((int)(durationSeconds * 1000));

        IsOnCooldown = false;
        Console.WriteLine("Skill kembali siap.");
    }
}
```
