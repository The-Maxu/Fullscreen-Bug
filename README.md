# Fullscreen-Bug (just a generic crash?)
Fullscreen-Bug (Crash when exiting fullscreen during page load with heavy ArrayBuffer allocation (PS4 WebKit v12.00))
Load the page in full screen.
Press refresh and exit full screen. A crash occurs on firmware 12.00.
VIDEO:
https://x.com/The_Maxu/status/1941978454724866525

# Bug Report: Crash when exiting fullscreen during heap-spray on page load

## Summary

When exiting fullscreen mode while a page is still loading and performing a large allocation of ArrayBuffer objects in JavaScript, the PS4 WebKit browser on firmware 12.00 crashes.
The crash only occurs if the transition from fullscreen to normal mode happens before the page load and memory allocation complete.

---

## Steps to Reproduce

1. Open the page [Fullscreen-out-bug](https://the-maxu.github.io/Fullscreen-Bug/) en WebKit PS4.
2. Enter fullscreen mode (`document.documentElement.requestFullscreen()` or via the browser UI).
3. While the page is still loading and the memory allocations are in progress, exit fullscreen mode.
4. The browser crashes.

---

## Expected Behavior

Exiting fullscreen during page load should not cause a crash, regardless of the amount of JS heap memory used.

---

## Actual Behavior

The browser crashes immediately when exiting fullscreen if the page load and memory allocation have not finished.

---

## Technical Notes

- The bug **does not occur if the heap-spray completes before exiting fullscreen**.
- The crash occurs in native code inside the browser process and results in a full termination of the Shell UI process.
- The issue is timing-dependent and reproducible only when fullscreen is exited while the page is still loading.
- The behavior suggests an interaction between page load completion callbacks, and teardown of the fullscreen or scene state.
- Heavy memory allocation delays page load completion, increasing the likelihood that fullscreen exit occurs while internal WebView-related objects are being invalidated.
- This indicates a possible race condition in the lifecycle management of the WebView or page context during fullscreen transitions.

---

## Crash Evidence
The system log shows that the crash occurs during thumbnail generation after page load completion, while the scene is being unloaded:
```
  OnFinishedLoad : 3567 ms
  Unload enqueue: OptionMenu
  Unload enqueue: OptionMenuContainer
  Ignored unload request because unload is already enqueued
```
Shortly after, the browser attempts to capture an image of the WebView:
```  
  MainScene.CreateThumbnail
  WebView.CaptureImage
  Saving to file: /user/home/.../frequency_thumb/XXXX.jpg
```
The process then crashes in native code:
``` 
  Got a SIGSEGV while executing native code
  thread name: SceShellUIMain
  proc name: SceShellUI
```
Register state at the time of the crash shows invalid or null pointers:
```
  rsi = 0x00000000
  rdx = 0x00000000
```
This sequence indicates that the crash happens after a page load completion callback, during a scene transition and while attempting to use WebView-related resources.

---

## System

- **Browser:**  WebKit (PS4 built-in WebKit browser)
- **Version:** Production, no access to logs or debug
- **OS:** PS4 v12.00

---


# Bug Report: Crash al salir de pantalla completa durante heap-spray en carga

## Resumen

Al salir del modo pantalla completa (fullscreen) mientras una página aún se está cargando y realiza una asignación masiva de objetos ArrayBuffer en JavaScript, el navegador WebKit de PS4 en firmware 12.00 sufre un crash.
El fallo solo ocurre si la transición de fullscreen a modo normal se produce antes de que finalicen la carga y la asignación de memoria.

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

El navegador crashea inmediatamente al salir de fullscreen si la carga de la página y la asignación de memoria no han finalizado.

---

## Observaciones técnicas

- El bug **no ocurre si el heap-spray termina antes de salir de fullscreen**.
- El crash ocurre en código nativo del navegador y provoca la terminación del proceso SceShellUI.
- El fallo depende del momento exacto en que se sale de fullscreen durante la carga.
- El comportamiento sugiere una interacción entre el callback de finalización de carga de la página, y la destrucción de la escena asociada a la WebView.
- La presión de memoria provocada por la asignación masiva de ArrayBuffer retrasa la finalización de la carga, aumentando la probabilidad de que la salida de fullscreen coincida con un estado intermedio del ciclo de vida de la WebView.
- Esto sugiere una posible condición de carrera en la gestión del ciclo de vida de la WebView o del contexto de la página durante transiciones de estado.

---

## Evidencia del crash
El log del sistema muestra que el crash ocurre durante la generación de un thumbnail tras finalizar la carga de la página, mientras la escena está siendo descargada:
```
  OnFinishedLoad : 3567 ms
  Unload enqueue: OptionMenu
  Unload enqueue: OptionMenuContainer
  Ignored unload request because unload is already enqueued
```
A continuación, el sistema intenta capturar una imagen de la WebView:
```
  MainScene.CreateThumbnail
  WebView.CaptureImage
  Saving to file: /user/home/.../frequency_thumb/XXXX.jpg
```
Y el proceso falla en código nativo:
```  
  Got a SIGSEGV while executing native code
  thread name: SceShellUIMain
  proc name: SceShellUI
```
Los registros muestran punteros inválidos o nulos en el momento del crash:
```
  rsi = 0x00000000
  rdx = 0x00000000
```
Esta secuencia indica que el fallo ocurre tras un callback de finalización de carga, durante una transición de escena, al intentar usar recursos asociados a la WebView.

---

## Sistema

- **Navegador:** WebKit (WebKit integrado en PS4)
- **Versión:** Producción, sin acceso a logs o debug
- **SO:** PS4 v12.00

---
