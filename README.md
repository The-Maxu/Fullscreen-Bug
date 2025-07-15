# Fullscreen-Bug (just a generic crash?)
Fullscreen-Bug
Load the page in full screen.
Press refresh and exit full screen. A crash occurs on firmware 12.00.
VIDEO:
https://x.com/The_Maxu/status/1941978454724866525

# Bug Report: Crash when exiting fullscreen during heap-spray on page load

## Summary

When exiting fullscreen mode while a page is loading and performing a massive heap-spray of `ArrayBuffer` objects in JavaScript, WebKit PS4 v12.00 crashes. The bug only occurs if the transition from fullscreen to normal happens before the load and heap-spray are complete.

---

## Steps to Reproduce

1. Open the page [Fullscreen-out-bug](https://the-maxu.github.io/Fullscreen-Bug/) en WebKit PS4.
2. Enter fullscreen mode (`document.documentElement.requestFullscreen()` or via the browser UI).
3. While the page is still loading and the heap-spray is running, exit fullscreen mode.
4. The browser crashes.

---

## Expected Behavior

Exiting fullscreen during page load should not cause a crash, regardless of the amount of JS heap memory used.

---

## Actual Behavior

The browser crashes immediately upon exiting fullscreen if the page load and heap-spray have not finished.

---

## Technical Notes

- The bug **does not occur if the heap-spray completes before exiting fullscreen**.
- The crash appears to be related to memory pressure (JS heap saturation) and the release of the document/DOM while event dispatching (`fullscreenchange`, etc).
- It may be a **use-after-free** or race condition in the lifecycle of nodes/documents during event dispatch and GC.
- Relevant code in WebKit: `FrameLoader.cpp`, `EventDispatcher.cpp`, `GCController.cpp`.

---

## System

- **Browser:**  WebKit
- **Version:** Production, no access to logs or debug
- **OS:** PS4 v12.00

---


# Bug Report: Crash al salir de pantalla completa durante heap-spray en carga

## Resumen

Al salir de pantalla completa (fullscreen) mientras se carga una página que realiza un heap-spray masivo de `ArrayBuffer` en JavaScript, WebKit en PS4 v12.00 sufre un crash. El bug ocurre únicamente cuando la transición de fullscreen a modo normal ocurre antes de que finalice la carga y el heap-spray.

---

## Pasos para reproducir

1. Abrir la página [Fullscreen-out-bug](https://the-maxu.github.io/Fullscreen-Bug/) en WebKit PS4.
2. Entrar en modo pantalla completa (`document.documentElement.requestFullscreen()` o mediante el navegador).
3. Mientras la página sigue cargando y el heap-spray está en curso, salir de pantalla completa.
4. El navegador crashea.

---

## Comportamiento esperado

Salir de pantalla completa durante la carga no debería provocar un crash, independientemente de la cantidad de memoria JS utilizada.

---

## Comportamiento actual

El navegador crashea inmediatamente al salir de fullscreen si la carga y el heap-spray no han finalizado.

---

## Observaciones técnicas

- El bug **no ocurre si el heap-spray termina antes de salir de fullscreen**.
- El crash parece estar relacionado con la presión de memoria (JS heap saturado) y la liberación del documento/DOM en medio del despacho de eventos (`fullscreenchange`, etc).
- Es posible que se trate de un **use-after-free** o condición de carrera en el ciclo de vida de nodos/documentos durante el despacho de eventos y el GC.
- Código relevante en WebKit: `FrameLoader.cpp`, `EventDispatcher.cpp`, `GCController.cpp`.

---

## Sistema

- **Navegador:** WebKit
- **Versión:** Producción, sin acceso a logs o debug
- **SO:** PS4 v12.00

---
