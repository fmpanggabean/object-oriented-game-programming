# Modul 4: Game Design Patterns dalam C#

**Design Patterns** merupakan sekumpulan solusi standar untuk mengatasi permasalahan umum dalam merancang struktur kode game. Pada modul ini, kita akan membahas enam pola yang sering diimplementasikan.

---

## 1. Singleton Pattern

Memastikan sebuah kelas hanya memiliki satu instance utama yang diakses secara global di dalam aplikasi.

### Contoh Pendekatan Sederhana

Variabel statis tanpa pembatasan instansiasi.

```csharp
using System;

public class GameManagerBad
{
    public static GameManagerBad instance;

    public GameManagerBad()
    {
        instance = this;
    }
}
```

### Contoh Penerapan Thread-Safe Singleton

Menggunakan `Lazy<T>` untuk memastikan instansiasi tunggal secara aman.

```csharp
using System;

public class Singleton<T> where T : class, new()
{
    private static readonly Lazy<T> lazyInstance = new Lazy<T>(() => new T());

    public static T Instance => lazyInstance.Value;

    protected Singleton() { }
}

public class GameManagerGood : Singleton<GameManagerGood>
{
    public int CurrentScore { get; private set; }

    public void AddScore(int amount)
    {
        CurrentScore += amount;
        Console.WriteLine($"Skor diperbarui: {CurrentScore}");
    }
}
```

---

## 2. State Pattern (Finite State Machine / FSM)

Mengorganisir perilaku status karakter (seperti status Musuh: `Idle`, `Patrol`, `Attack`) ke dalam kelas-kelas terpisah.

### Contoh Pendekatan Sederhana

Menggunakan banyak variabel penanda boolean dalam satu method pengelola.

```csharp
using System;

public class EnemyAIBad
{
    public bool isIdle;
    public bool isPatrolling;
    public bool isAttacking;

    public void UpdateLogic()
    {
        if (isAttacking)
        {
            // Logika menyerang
        }
        else if (isPatrolling)
        {
            // Logika patroli
        }
    }
}
```

### Contoh Penerapan State Pattern

Setiap status diisolasi ke dalam kelas yang mengimplementasikan interface `IState`.

```csharp
using System;

public interface IState
{
    void Enter();
    void Execute();
    void Exit();
}

public class StateMachine
{
    public IState CurrentState { get; private set; }

    public void ChangeState(IState newState)
    {
        CurrentState?.Exit();
        CurrentState = newState;
        CurrentState?.Enter();
    }

    public void Update()
    {
        CurrentState?.Execute();
    }
}

public class PatrolState : IState
{
    private readonly string entityName;

    public PatrolState(string entityName) { this.entityName = entityName; }

    public void Enter() => Console.WriteLine($"{entityName} mulai berpatroli.");
    public void Execute() => Console.WriteLine($"{entityName} berjalan mengikuti rute patroli.");
    public void Exit() => Console.WriteLine($"{entityName} menghentikan patroli.");
}

public class AttackState : IState
{
    private readonly string entityName;

    public AttackState(string entityName) { this.entityName = entityName; }

    public void Enter() => Console.WriteLine($"{entityName} bersiap menyerang.");
    public void Execute() => Console.WriteLine($"{entityName} melakukan serangan ke target.");
    public void Exit() => Console.WriteLine($"{entityName} selesai menyerang.");
}
```

---

## 3. Command Pattern

Membungkus sebuah perintah atau aksi ke dalam objek `ICommand` terpisah.

### Contoh Pendekatan Sederhana

Eksekusi aksi langsung dilakukan di dalam pengkondisian input.

```csharp
using System;

public class PlayerInputBad
{
    public void HandleInput(string key)
    {
        if (key == "SPACE")
        {
            Console.WriteLine("Karakter melompat.");
        }
    }
}
```

### Contoh Penerapan Command Pattern

Perintah diisolasi ke dalam kelas `MoveCommand` yang mendukung fitur pembatalan aksi (*Undo*).

```csharp
using System;
using System.Collections.Generic;

public interface ICommand
{
    void Execute();
    void Undo();
}

public class MoveCommand : ICommand
{
    private readonly string entityName;
    private readonly float moveAmount;

    public MoveCommand(string entityName, float amount)
    {
        this.entityName = entityName;
        this.moveAmount = amount;
    }

    public void Execute() => Console.WriteLine($"{entityName} bergerak maju sebesar {moveAmount} unit.");
    public void Undo() => Console.WriteLine($"{entityName} mengundurkan posisi sebesar {moveAmount} unit.");
}

public class CommandInvoker
{
    private readonly Stack<ICommand> commandHistory = new Stack<ICommand>();

    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        commandHistory.Push(command);
    }

    public void UndoLastCommand()
    {
        if (commandHistory.Count > 0)
        {
            ICommand lastCmd = commandHistory.Pop();
            lastCmd.Undo();
        }
    }
}
```

---

## 4. Object Pooling Pattern

Mendaur ulang objek yang sering dibuat dan dihapus agar meminimalisir alokasi memori baru.

### Contoh Pendekatan Sederhana

Membuat objek baru (`new`) secara berulang setiap kali membutuhkan proyektil.

```csharp
using System;

public class GunBad
{
    public void Shoot()
    {
        Bullet b = new Bullet(); 
    }
}

public class Bullet { }
```

### Contoh Penerapan Object Pooling

Kumpulan objek peluru disimpan di dalam kelas pengelola `BulletPool` dan diaktifkan kembali saat dibutuhkan.

```csharp
using System;
using System.Collections.Generic;

public class Bullet
{
    public bool IsActive { get; set; }

    public void Reset()
    {
        IsActive = true;
    }
}

public class BulletPool
{
    private readonly Queue<Bullet> pool = new Queue<Bullet>();

    public BulletPool(int initialSize = 10)
    {
        for (int i = 0; i < initialSize; i++)
        {
            Bullet b = new Bullet { IsActive = false };
            pool.Enqueue(b);
        }
    }

    public Bullet GetBullet()
    {
        if (pool.Count > 0)
        {
            Bullet bullet = pool.Dequeue();
            bullet.Reset();
            Console.WriteLine("Menggunakan kembali peluru dari dalam pool.");
            return bullet;
        }

        Console.WriteLine("Pool habis, membuat peluru baru.");
        return new Bullet { IsActive = true };
    }

    public void ReturnBullet(Bullet bullet)
    {
        bullet.IsActive = false;
        pool.Enqueue(bullet);
        Console.WriteLine("Peluru dikembalikan ke dalam pool.");
    }
}
```

---

## 5. Observer Pattern

Memungkinkan suatu komponen mempublikasikan event ke komponen pengamat (*observer*) yang terdaftar tanpa perlu saling mengenal detail satu sama lain.

### Contoh Pendekatan Sederhana

Karakter menyimpan referensi langsung ke berbagai manager.

```csharp
using System;

public class PlayerCoupled
{
    public UIManager uiManager;
    public ScoreManager scoreManager;

    public void OnEnemyKilled()
    {
        uiManager?.UpdateKillUI();
        scoreManager?.AddKillPoint();
    }
}

public class UIManager { public void UpdateKillUI() { } }
public class ScoreManager { public void AddKillPoint() { } }
```

### Contoh Penerapan Observer Pattern

Penggunaan `GameEventBus` terpusat untuk mendistribusikan notifikasi event.

```csharp
using System;

public static class GameEventBus
{
    public static event Action OnEnemyKilled;

    public static void PublishEnemyKilled()
    {
        OnEnemyKilled?.Invoke();
    }
}

public class PlayerDecoupling
{
    public void KillEnemy()
    {
        Console.WriteLine("Musuh berhasil dikalahkan.");
        GameEventBus.PublishEnemyKilled();
    }
}

public class ScoreObserver
{
    public ScoreObserver()
    {
        GameEventBus.OnEnemyKilled += AddScore;
    }

    private void AddScore() => Console.WriteLine("Skor bertambah +100 poin.");
}
```

---

## 6. Factory Pattern

Mengisolasi logika pembuatan objek kompleks ke dalam kelas khusus (*Factory*).

### Contoh Pendekatan Sederhana

Instansiasi objek dilakukan langsung di dalam method spawner.

```csharp
using System;

public class SpawnerBad
{
    public void Spawn(string type)
    {
        if (type == "Goblin")
        {
            // Instansiasi langsung
        }
    }
}
```

### Contoh Penerapan Factory Pattern

Pemisahan logika pembuataan unit melalui kelas `GoblinFactory`.

```csharp
using System;

public abstract class Enemy
{
    public string Name { get; protected set; }
    public abstract void Attack();
}

public class Goblin : Enemy
{
    public Goblin() { Name = "Goblin"; }
    public override void Attack() => Console.WriteLine("Goblin menyerang dengan pisau.");
}

public abstract class EnemyFactory
{
    public abstract Enemy CreateEnemy();
}

public class GoblinFactory : EnemyFactory
{
    public override Enemy CreateEnemy()
    {
        Console.WriteLine("Pabrik membuat unit Goblin baru.");
        return new Goblin();
    }
}
```
