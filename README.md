# 🖌️ 16x16 Paint in Assembly

This project is a simple **paint application** created in a **simulated 16-bit Assembly environment**. It allows you to draw colorful 16x16 pixel blocks on a canvas, switch between colors, and move the drawing cursor using your keyboard. The app runs in a simulated low-level graphical environment and was developed for the **Systems 1 - Hardware** class.

## 🎮 Features

- 16x16 pixel "paintbrush"
- Keyboard controls for movement and painting
- 5 available colors: Blue, Red, Green, Yellow, Orange
- Minimal UI with menu and instructions
- Exit functionality with a thank-you screen

## 🎨 Controls

| Action           | Key     |
| ---------------- | ------- |
| Paint Block      | `Space` |
| Move Up          | `W`     |
| Move Down        | `S`     |
| Move Left        | `A`     |
| Move Right       | `D`     |
| Change to Blue   | `B`     |
| Change to Red    | `R`     |
| Change to Green  | `G`     |
| Change to Yellow | `Y`     |
| Change to Orange | `O`     |
| Quit             | `Q`     |

## 🧠 How It Works

- **Text and Bitmap Modes**: The app starts in text mode for displaying the menu and instructions, then switches to bitmap mode for painting.
- **Input Polling**: It polls keyboard input to handle movement, color changes, and painting.
- **Graphics Simulation**: Uses I/O ports and memory-mapped video addresses to draw pixels and characters.

## 📸 Screenshots

### Main Menu

![Menu1](/pictures/1.png)

### Instructions

![Menu2](/pictures/2.png)

### Paint Mode

![Menu3](/pictures/3.png)

## ⚙️ Requirements

- Assembly simulator that supports:
  - Bitmap & text display modes
  - I/O port control (e.g., `OUT`, `IN`)
  - Basic stack operations and memory access
- Tested on and made for (not sure if this will work on other versions): [16-Bit Assembly Simulator (link)](https://e.famnit.upr.si/pluginfile.php/696726/mod_resource/content/1/index.html)

## 📘 Educational Use

This project was created as an **educational exercise** to demonstrate low-level hardware interaction, memory handling, and basic user interface creation in Assembly. It provides a visual and interactive way to explore how computers operate at a hardware level.

---

Made with `MOV`, `OUT`, and a lot of patience 😊
