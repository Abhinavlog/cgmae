# cgmae
# Manga Drift 🏎️💨

**Manga Drift** is a high-speed, stylized pseudo-3D racing and drifting game built entirely with Flutter and Dart. It features a unique "Manga/Ink" aesthetic, deterministic physics, procedural generation, and a custom software 3D renderer that requires **zero external game engines or image assets**.

---

## 📖 Overview

Manga Drift is designed to be a lightweight, cross-platform arcade racer. It uses a custom-built 3D projection engine to render 3D boxes and cylinders onto a 2D Flutter `Canvas`. 

The game operates in two visual modes:
*   **Offline (Manga B/W):** A high-contrast, black-and-white ink style with dynamic speed lines.
*   **Online (Color Palettes):** When connected to the leaderboard server, the game randomly selects vibrant retro-synthwave and neon color palettes.

---

## ✨ Features

*   **Custom 3D Renderer:** Renders 3D boxes and cylinders with depth sorting, back-face culling, and dynamic shading without using `flutter_cube` or OpenGL.
*   **Three Unique Vehicles:** 
    *   **Ferrari:** A wide, low-slung red supercar.
    *   **Mazda:** A white, long-hooded GT racer.
    *   **F1:** A silver Mercedes-style open-wheel formula car with Petronas accents.
*   **Deterministic Physics:** Runs on a fixed 120Hz physics timestep. This guarantees that replays and leaderboard entries are perfectly accurate.
*   **Three Game Modes:**
    *   **Drift:** Chain drifts together to build a combo multiplier.
    *   **Speed:** Reach the highest top speed possible.
    *   **No Brake:** Reach a target speed and drive as far as possible without touching the brake.
*   **Procedural Generation:** Pillars and ground ticks are generated using a deterministic hash function based on the level seed.
*   **Leaderboard & Replays:** Built-in support for local mock leaderboards or HTTP backend integration.

---

## 🏗️ Architecture & Technical Details

The game is built on three core pillars: **The Game Loop**, **The Math/Camera**, and **The Custom Painter**.

### 1. The Game Loop & Physics (`Game` class)
The game uses a hybrid update system. The Flutter `Ticker` provides a variable delta time (`dt`), but the physics engine accumulates this time and processes it in fixed `1/120` second steps (`kStep`). 
*   **Input:** The game reads raw keyboard input (`HardwareKeyboard`) every frame.
*   **Bicycle Model Steering:** Steering is calculated using a softened bicycle model. Grip is dynamically altered based on speed, throttle, and whether the handbrake is pulled, allowing for realistic power-over drift mechanics.
*   **Replay Recorder:** Every physics tick, an `Input` bitmask is recorded. When a run finishes, this string of bits is encoded and saved.

### 2. The Camera (`_Cam` class)
The game uses a **pinhole camera with no pitch**. This means vertical lines always remain vertical on the screen (a technique similar to two-point perspective). 
*   Instead of tilting the camera down, the horizon line is simply shifted up the screen.
*   The camera dynamically follows the *velocity* of the car rather than the heading, which allows the player to see the side of the car during a drift.

### 3. The Renderer (`GamePainter` class)
Instead of drawing flat sprites, the game builds 3D geometry and projects it.
*   **`_boxFaces()`:** Takes 3D dimensions, calculates the 8 vertices, culls the back faces (based on the camera position), and returns a list of visible faces with appropriate light/dark shading.
*   **`_cylinderFaces()`:** Generates a 10-segment cylinder for the wheels. It handles radial culling for the sides and axial culling for the end-caps, applying shading based on the surface normal.
*   **Depth Sorting:** Every object (pillars, smoke, car parts) is assigned a Z-depth. Before rendering, the list is sorted from farthest to nearest (Painter's Algorithm) to ensure proper overlapping.

---

## 🚗 Vehicle Geometry

Because the game uses a box-and-cylinder renderer, cars are built using a "Low-Poly / Lego" approach. 
*   The **wheels** are cylinders (`_cylinderFaces`).
*   The **chassis, cabins, and spoilers** are boxes (`_boxFaces`) stacked in 3D space.
*   Cars are drawn in a specific order: Far wheels ➔ Chassis ➔ Cabin ➔ Near wheels. This ensures the wheels correctly overlap the bodywork depending on the camera angle.

---

## 📂 Project Structure

To run the game, your `lib/` folder should contain:
*   `main.dart` – The entry point, game loop, physics engine, and custom renderer.
*   `levels.dart` – Level definitions, stars, unlocks, and the Level Select UI.
*   `leaderboard.dart` – Leaderboard logic, HTTP/Mock backends, and the Rankings UI.

---

## 🚀 Getting Started

### Prerequisites
*   Flutter SDK (3.x or higher)
*   A keyboard (required for input)

### Installation
1. Clone the repository.
2. Run `flutter pub get` to fetch dependencies.
3. Run the game on your desktop platform of choice:
   ```bash
   flutter run -d windows
   # OR
   flutter run -d macos
   # OR
   flutter run -d linux
