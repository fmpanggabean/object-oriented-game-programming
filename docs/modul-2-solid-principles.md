# Modul 2: Prinsip SOLID dalam Game Programming

Prinsip **SOLID** mencakup 5 panduan arsitektur perangkat lunak yang bertujuan membuat kode lebih rapi, fleksibel, serta mudah diuji dan dikembangkan seiring berjalannya project.

---

## 1. Single Responsibility Principle (SRP)

> *"Satu kelas hendaknya hanya memiliki satu tanggung jawab utama."*

### Contoh Pendekatan Sederhana

Satu kelas `PlayerControllerBad` menangani banyak tugas sekaligus (pergerakan, status kesehatan, audio, dan UI).

```csharp
using System;

public class PlayerControllerBad
{
    public float health = 100f;
    public float speed = 5f;

    public void UpdateGameLoop(string inputKey)
    {
        if (inputKey == "W") Console.WriteLine("Bergerak maju");
        Console.WriteLine("Tampilkan UI Health: " + health);
    }

    public void TakeDamage(float damage)
    {
        health -= damage;
        Console.WriteLine("Mainkan Efek Suara Hurt");

        if (health <= 0)
        {
            Console.WriteLine("Tampilkan Layar Game Over");
        }
    }
}
```

### Contoh Penerapan SRP

Tanggung jawab dipisah menjadi beberapa kelas kecil yang fokus pada tugasnya masing-masing.

```csharp
using System;

public class PlayerHealthSRP
{
    public float CurrentHealth { get; private set; } = 100f;

    public event Action<float> OnHealthChanged;
    public event Action OnDeath;

    public void TakeDamage(float damage)
    {
        CurrentHealth = Math.Max(0, CurrentHealth - damage);
        OnHealthChanged?.Invoke(CurrentHealth);

        if (CurrentHealth <= 0)
        {
            OnDeath?.Invoke();
        }
    }
}

public class PlayerMotor
{
    public float MoveSpeed { get; set; } = 5f;

    public void Move(float x, float y)
    {
        Console.WriteLine($"Bergerak ke posisi ({x * MoveSpeed}, {y * MoveSpeed}).");
    }
}

public class PlayerAudioNotifier
{
    public void PlayHurtSound() => Console.WriteLine("Memainkan SFX Hurt.");
    public void PlayDeathSound() => Console.WriteLine("Memainkan SFX Death.");
}
```

---

## 2. Open/Closed Principle (OCP)

> *"Komponen kode terbuka untuk diperluas fiturnya, tetapi tertutup untuk modifikasi pada kode utamanya."*

### Contoh Pendekatan Sederhana

Menambah tipe senjata baru membuat kita harus mengubah baris kode di dalam percabangan `switch-case`.

```csharp
using System;

public enum WeaponType { Sword, Bow, Laser }

public class WeaponSystemBad
{
    public void Attack(WeaponType type)
    {
        if (type == WeaponType.Sword)
        {
            Console.WriteLine("Menebas dengan pedang.");
        }
        else if (type == WeaponType.Bow)
        {
            Console.WriteLine("Menembakkan panah.");
        }
        else if (type == WeaponType.Laser)
        {
            Console.WriteLine("Menembakkan laser.");
        }
    }
}
```

### Contoh Penerapan OCP

Dengan memanfaatkan interface `IWeapon`, penambahan senjata baru cukup dilakukan dengan membuat kelas baru tanpa perlu mengubah kode `WeaponSystemGood`.

```csharp
using System;

public interface IWeapon
{
    void Shoot();
}

public class SwordWeapon : IWeapon
{
    public void Shoot() => Console.WriteLine("Serangan tebasan pedang.");
}

public class BowWeapon : IWeapon
{
    public void Shoot() => Console.WriteLine("Menembakkan anak panah.");
}

public class MagicWandWeapon : IWeapon
{
    public void Shoot() => Console.WriteLine("Menembakkan bola sihir.");
}

public class WeaponSystemGood
{
    private IWeapon currentWeapon;

    public void SetWeapon(IWeapon newWeapon)
    {
        currentWeapon = newWeapon;
    }

    public void ExecuteAttack()
    {
        currentWeapon?.Shoot();
    }
}
```

---

## 3. Liskov Substitution Principle (LSP)

> *"Kelas turunan harus dapat menggantikan posisi kelas induknya tanpa merusak perilaku sistem."*

### Contoh Pendekatan Sederhana

Kelas `FlyingEnemy` menurunkan `EnemyLSPBad`, tetapi melempar error pada method `MoveOnGround()` karena musuh terbang tidak berjalan di tanah.

```csharp
using System;

public abstract class EnemyLSPBad
{
    public abstract void MoveOnGround();
}

public class GroundEnemy : EnemyLSPBad
{
    public override void MoveOnGround()
    {
        Console.WriteLine("Berjalan di tanah.");
    }
}

public class FlyingEnemyBad : EnemyLSPBad
{
    public override void MoveOnGround()
    {
        throw new NotImplementedException("Musuh terbang tidak berjalan di tanah.");
    }
}
```

### Contoh Penerapan LSP

Pemisahan kemampuan gerak ke dalam interface yang sesuai.

```csharp
using System;

public interface IMovableGround
{
    void Walk();
}

public interface IMovableAir
{
    void Fly();
}

public class GoblinEnemy : IMovableGround
{
    public void Walk() => Console.WriteLine("Goblin berjalan di atas tanah.");
}

public class BatEnemy : IMovableAir
{
    public void Fly() => Console.WriteLine("Kelelawar terbang di udara.");
}
```

---

## 4. Interface Segregation Principle (ISP)

> *"Interface sebaiknya bersifat spesifik dan tidak memaksa kelas mengimplementasikan method yang tidak dibutuhkannya."*

### Contoh Pendekatan Sederhana

Interface `IGameUnitBad` yang terlalu luas membuat kelas `TurretBad` harus mengimplementasikan method kosong.

```csharp
using System;

public interface IGameUnitBad
{
    void TakeDamage(float damage);
    void Walk();
    void Jump();
}

public class TurretBad : IGameUnitBad
{
    public void TakeDamage(float damage) { Console.WriteLine("Turret menerima damage."); }
    public void Walk() { }
    public void Jump() { }
}
```

### Contoh Penerapan ISP

Interface dipecah menjadi bagian-bagian kecil yang lebih khusus.

```csharp
using System;

public interface IDamageable
{
    void TakeDamage(float damage);
}

public interface IMovable
{
    void Move(float x, float y);
}

public class StationaryTurret : IDamageable
{
    public void TakeDamage(float damage)
    {
        Console.WriteLine($"Turret menerima damage sebesar {damage}.");
    }
}
```

---

## 5. Dependency Inversion Principle (DIP)

> *"Modul tingkat tinggi sebaiknya tidak bergantung langsung pada kelas konkret tingkat rendah, melainkan pada abstraksinya."*

### Contoh Pendekatan Sederhana

Kelas `PlayerSaveManagerBad` bergantung secara langsung pada `FileDiskStorage`.

```csharp
using System;

public class FileDiskStorage
{
    public void SaveToFile(string data) { Console.WriteLine("Menyimpan data ke disk."); }
}

public class PlayerSaveManagerBad
{
    private FileDiskStorage storage = new FileDiskStorage();

    public void SaveProgress()
    {
        storage.SaveToFile("Level=10");
    }
}
```

### Contoh Penerapan DIP

Kelas pengelola penyimpanan menerima interface `ISaveService` melalui konstruktor.

```csharp
using System;

public interface ISaveService
{
    void SaveData(string key, string data);
}

public class FileSaveService : ISaveService
{
    public void SaveData(string key, string data)
    {
        Console.WriteLine($"Simpan file lokal: {key} -> {data}");
    }
}

public class CloudSaveService : ISaveService
{
    public void SaveData(string key, string data)
    {
        Console.WriteLine($"Unggah server cloud: {key} -> {data}");
    }
}

public class PlayerSaveManagerGood
{
    private readonly ISaveService saveService;

    public PlayerSaveManagerGood(ISaveService saveService)
    {
        this.saveService = saveService;
    }

    public void SaveProgress()
    {
        saveService?.SaveData("PlayerProgress", "Gold=500;Level=3");
    }
}
```
