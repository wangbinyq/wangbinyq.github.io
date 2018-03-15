---
title: electron-api
tags:
---

## Process
1. Main Process and Render Proccess (`BrowserWindow` module)
2. Main process is responsible for creating and managing `BrowserWindow` and application events.
2. In The Render Process we can get `BrowserWindow` module and other electron module by `electron.remote`
3. `ipcMain, ipcRender` module allows you to send and receive synchronous(`sendSync`) and asynchronous(`send`) messages between the main and render processes.

## Window
3. There is a option(`{show: false}`) for `BrowserWindow` can hide the window, which can be used for background task.
4. we use `BrowserWindow.loadURL` to load html for our works. 
4. We can listen various events on window like `focus, blur, close` etc. There are also `unresponsive, responsive` event on `BrowserWindow` and `crashed` event on `BrowserWindow.webContents`
5. window can be frameless, without toolbars, title bars, status bars, borders (`{frame: false}`).
6. WebContents: we can send event to window

## Menu
- Two kinds of menus: application menu, context menu
- Two modules: `electron.Menu, electron.MenuItem` (alse in remote)

## Native User Interface
- `shell`: open external links and the file manager
- `dialog`: system dialogs
- `Tray`

## System
