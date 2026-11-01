# 🪙 Coin Frenzy

> ⚡ A blazing-fast 2D arcade reflex game built with **Unity 6** and the **Universal Render Pipeline (URP)**!
> Catch falling golden coins 🪙, dodge deadly bombs 💣, and rack up the highest score you can — all in just **60 seconds**! 🕐
> Features epic sound design, animated UI, a rainbow-tier ranking system 🌈, and persistent high-score tracking. **Are you ready for the frenzy?** 🔥

---

## 🛠️ Technical Specifications

| 🔧 Property | 📋 Details |
|---|---|
| 🎮 Engine | Unity 6 (`6000.6.7f1`) |
| 🖼️ Render Pipeline | Universal Render Pipeline (URP 2D — `17.6.0`) |
| 💻 Scripting Backend | C# (`.NET Standard 2.1`) |
| 🕹️ Input System | Unity Input Manager (Legacy & Action Mappings) |
| 🖥️ Target Platforms | PC / macOS / Linux / WebGL |
| ⚙️ Physics Model | 2D Physics (Trigger Colliders & Kinematic Translation) |

---

## ✨ Key Features

- ⚡ **Fast Arcade Mechanics** — Snappy horizontal movement and satisfying falling collectible gameplay that keeps you on your toes!
- ⚖️ **Risk vs. Reward Scoring** — Coins give you points 💰, but bombs punish you hard 💥. Score is clamped — it can never go negative!
- 🎲 **Dynamic Spawner** — Randomized X positions, configurable spawn rates, and tweakable bomb-to-coin ratios for endless replayability.
- 🔊 **Zero-Latency Audio Engine** — Custom-built audio system with in-memory waveform trimming and pre-allocated voice pooling for instantaneous, delay-free SFX. No lag, ever!
- 🎨 **Animated HUD & Game Over Sequence** — Smooth state transitions, screen shake 📳, pulsating text, animated score count-up, and real-time rainbow 🌈 HSV rank displays.
- 🏆 **Persistent High Score System** — Your best run is saved locally via `PlayerPrefs` and carries over across all sessions.
- 🧩 **Clean Component Design** — Decoupled scripts following Single Responsibility and Singleton patterns for easy maintenance and expansion.

---

## 🎮 Gameplay & Controls

### 🎯 Objective
Catch as many **golden coins** 🪙 as possible before the **60-second countdown** ⏳ hits zero. Watch out for **bombs** 💣 — they'll slash your score!

### 🕹️ Controls

| 🎮 Action | ⌨️ Primary Key | 🖱️ Secondary Input |
|---|---|---|
| ⬅️ Move Left | `Left Arrow` / `A` | Horizontal Axis |
| ➡️ Move Right | `Right Arrow` / `D` | Horizontal Axis |
| ▶️ Start Game | `Spacebar` | Left Mouse Button |
| 🔄 Restart Game | `R` | — |

---

## 🏅 Scoring & Ranking System

| 💥 Event | 📊 Effect |
|---|---|
| 🪙 Coin Collected | **+1 Point** |
| 💣 Bomb Hit | **−3 Points** *(can't go below 0!)* |
| ⏱️ Time Limit | **60 Seconds** |

After the match ends, your final score earns you a **prestige rank** 🎖️:

| 🏆 Score Threshold | 🎖️ Rank | 💬 Description |
|---|---|---|
| 🌈 40+ Points | **LEGEND!!!** | Master-tier performance with animated rainbow styling! |
| 🔵 25 – 39 Points | **PRO** | Advanced catcher efficiency — impressive! |
| 🟡 10 – 24 Points | **NOT BAD** | Solid beginner performance, keep it up! |
| 🔴 0 – 9 Points | **TRY AGAIN** | Hit more coins, dodge more bombs — you've got this! |

---

## 🏗️ Architecture & System Design

> 🧠 Coin Frenzy is built around lightweight, decoupled components for maximum performance, scalability, and code clarity.

```mermaid
flowchart TD
    subgraph 🎛️ Core Managers
        GM[🎮 GameManager] -->|State / Score / Timer| UI[🖥️ Animated OnGUI HUD]
        AM[🔊 AudioManager] -->|SFX & BGM Playback| AudioDevices[🔈 Audio Listener & Pool]
    end

    subgraph 🌀 Spawner System
        SP[📦 Spawner] -->|Instantiates| Coin[🪙 coin.prefab]
        SP -->|Instantiates| Bomb[💣 bomb.prefab]
    end

    subgraph 🧍 Player Layer
        Player[🥣 Catcher] -->|Moves horizontally| ScreenBounds[📐 Boundary Clamp]
    end

    subgraph ⚡ Interaction
        Coin -->|Triggers On Catch| GM
        Coin -->|Triggers On Catch| AM
        Bomb -->|Triggers On Hit| GM
    end
```

### 📜 Script Architecture

1. 🎮 **`GameManager.cs`**
   - Core game loop controller handling `Awake`, `Start`, `Running`, and `GameOver` states.
   - Manages the match timer (`timeLeft`), live score, and high score persistence via `PlayerPrefs`.
   - Drives GUI rendering, screen shake matrix transforms, and animated score tally sequences.

2. 🔊 **`AudioManager.cs`**
   - Centralized audio hub managing background music 🎵 and all sound effects 🔔.
   - Features `PrepareCoinAudio()` — a smart in-memory waveform scanner that auto-trims leading silence from audio clips on `Awake()`, **eliminating playback latency**.
   - Multi-source pool with **8 pre-allocated `AudioSource` instances** using `PlayOneShot` for rapid, overlapping coin chimes with zero audio cutoff.

3. 🥣 **`catcher.cs`**
   - Player movement script using smooth horizontal axis translation.
   - Enforces configurable boundary limits via `Mathf.Clamp` to keep the catcher within the visible camera area.

4. 📦 **`spawner.cs`**
   - Procedural item generator that spawns prefabs at randomized X coordinates along the top screen edge.
   - Configurable `spawnInterval` and `bombChance` for easy gameplay tuning.

5. 🪙 **`coinDrop.cs`**
   - Translates coin prefabs downward at a constant velocity.
   - Detects trigger collisions with the catcher, increments score, triggers audio, and self-destructs.
   - Automatically cleans up coins that fall below the lower screen threshold (`y < -6f`).

6. 💣 **`bomb.cs`**
   - Translates bomb hazards downward.
   - Detects collisions with the catcher, applies score penalty, and handles object cleanup.

---

## 🔊 Audio Engine Optimization

> 🎵 Sound latency ruins arcade games. Coin Frenzy solves this with a **custom low-latency audio engine** built into `AudioManager.cs`.

- ✂️ **Automated Silence Trimming** — On startup, raw audio samples are inspected using `AudioClip.GetData()`. The engine finds the exact sample where amplitude exceeds `0.02f` and constructs a clean in-memory PCM `AudioClip` — so sounds start **instantly**.
- ⚡ **Zero-Seek Triggering** — All sounds play from sample 0 via `AudioSource.PlayOneShot()`, completely eliminating runtime seek operations.
- 🔈 **Pre-allocated Channel Pool** — 8 independent audio source channels prevent cutoffs during high-density coin collection streaks. Collect 10 coins at once? No problem!

---

## 📁 Project Structure

```
🗂️ Coin Frenzy (ISD ONLINE WORKSHOP)/
├── 📂 Assets/
│   ├── 🔊 Audio/
│   │   ├── bgmusic.mp3               🎵 Looping arcade electronic soundtrack
│   │   └── Coin Collect SFX.mp3     🔔 High-fidelity coin collection chime
│   ├── 🧩 Prefabs/
│   │   ├── Spawner.prefab            📦 Configured procedural item spawner
│   │   ├── coin.prefab               🪙 Coin prefab with collider and coinDrop script
│   │   └── bomb.prefab               💣 Bomb hazard prefab with collider and penalty script
│   ├── 🎬 Scenes/
│   │   ├── isd workshop.unity        🎮 Main production game scene
│   │   └── isd online.unity          🔧 Workshop iteration scene
│   ├── 📜 Scripts/
│   │   ├── AudioManager.cs           🔊 Audio manager and low-latency audio engine
│   │   ├── GameManager.cs            🎮 Game state, HUD, timer, and high score manager
│   │   ├── bomb.cs                   💣 Bomb translation and hazard trigger
│   │   ├── catcher.cs                🥣 Player horizontal controller
│   │   ├── coinDrop.cs               🪙 Coin translation and collection trigger
│   │   └── spawner.cs                📦 Randomized item and hazard spawner
│   ├── ⚙️ Settings/                  URP 2D renderer assets and input settings
│   └── 🖼️ Sprites/                  2D visuals (Neon background, coins, bombs, bowl)
├── 📦 Packages/                      Unity package manifests (URP, 2D suite, Input System)
├── ⚙️ ProjectSettings/               Project configuration, tags, physics, and build settings
└── 📄 README.md                      📖 Project documentation
```

---

## 🎛️ Inspector Configuration Parameters

> 🔧 All components expose developer-friendly Inspector parameters for quick and painless gameplay balancing!

### 🎮 GameManager (`GameManager.cs`)
| ⚙️ Parameter | 🔢 Default | 📋 Description |
|---|---|---|
| `timeLimit` | `60f` | ⏱️ Duration of a game round in seconds |

### 📦 Spawner (`spawner.cs`)
| ⚙️ Parameter | 🔢 Default | 📋 Description |
|---|---|---|
| `coinPrefab` | `coin` | 🪙 Prefab reference for falling coin collectible |
| `bombPrefab` | `bomb` | 💣 Prefab reference for falling hazard |
| `spawnInterval` | `0.8f` | ⏳ Time in seconds between item spawns |
| `xRange` | `8.0f` | ↔️ Horizontal span (−xRange to +xRange) for item drops |
| `spawnY` | `6.0f` | ⬆️ Vertical spawn height |
| `bombChance` | `0.25f` | 🎲 Probability (0.0 – 1.0) of spawning a bomb instead of a coin |

### 🥣 Player Catcher (`catcher.cs`)
| ⚙️ Parameter | 🔢 Default | 📋 Description |
|---|---|---|
| `speed` | `10.0f` | ➡️ Horizontal movement speed |
| `limit` | `8.0f` | 📐 Maximum absolute X position (screen boundaries) |

### 💣 Hazard (`bomb.cs`)
| ⚙️ Parameter | 🔢 Default | 📋 Description |
|---|---|---|
| `fallSpeed` | `4.0f` | ⬇️ Downward movement speed |
| `scorePenalty` | `3` | 💥 Points deducted upon contact |

### 🪙 Collectible (`coinDrop.cs`)
| ⚙️ Parameter | 🔢 Default | 📋 Description |
|---|---|---|
| `fallSpeed` | `4.0f` | ⬇️ Downward movement speed |

### 🔊 Audio Manager (`AudioManager.cs`)
| ⚙️ Parameter | 🔢 Default | 📋 Description |
|---|---|---|
| `bgMusic` | `bgmusic` | 🎵 Background music clip |
| `coinSound` | `Coin Collect SFX` | 🔔 Coin collection sound effect clip |
| `musicStartTime` | `16.0f` | ⏭️ Start/loop timestamp in seconds for background track drop |
| `musicVolume` | `0.5f` | 🔉 Volume level for background music (0.0 – 1.0) |
| `coinVolume` | `1.0f` | 🔊 Volume level for coin SFX (0.0 – 1.0) |

---

## 🚀 Installation & Setup

### 📋 Prerequisites
Before you start the frenzy, make sure you have: 
- ✅ **Unity Hub** installed on your machine.
- ✅ **Unity 6 Editor** (version `6000.6.7f1` or a compatible version).

### 📂 Open the Project in Unity
1. 🖥️ Launch **Unity Hub**.
2. ➕ Click **"Add"** and select the `ISD ONLINE WORKSHOP` project directory.
3. 🎮 Open the project with **Unity 6** and wait for it to import.

### ▶️ Run the Game
1. 📁 Navigate to `Assets/Scenes/` in the **Project** window.
2. 🎬 Open `isd workshop.unity`.
3. ▶️ Press the **Play** button in the Unity Editor toolbar.
4. 🕹️ Press **Spacebar** or **click** to begin the frenzy!

---

## 🏗️ Build Instructions

### 🖥️ Standalone (Windows / macOS / Linux)
1. 📂 Open **File > Build Profiles** (or **Build Settings**).
2. ✅ Ensure `Assets/Scenes/isd workshop.unity` is listed in **Scenes In Build**.
3. 🖥️ Select your target platform (**macOS**, **Windows**, or **Linux**).
4. 🔨 Click **Build** or **Build and Run** — and enjoy Coin Frenzy on your desktop!

### 🌐 WebGL (Play in Browser!)
1. 🌐 In **Build Profiles**, switch the target platform to **WebGL**.
2. ⚙️ In **Project Settings > Player > Resolution and Presentation**, configure resolution settings *(recommended: `1920×1080` or responsive canvas)*.
3. 🔨 Click **Build** and export to your desired output directory. Then host it and share the frenzy! 🎉

---

## 📄 License

> 🎓 This project was developed for **educational and workshop purposes** as part of the **ISD Online Workshop** curriculum.
> All assets and code are **free to use and modify** for learning, prototyping, and portfolio demonstrations.
> Go build something awesome! 🚀✨
