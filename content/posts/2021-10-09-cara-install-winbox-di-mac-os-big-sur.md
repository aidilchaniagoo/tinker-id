---
author: Aidil Putra
date: '2021-10-09'
description: >-
    Secara default winbox hanya bisa di jalankan di sistem operasi Windows.
    Namun kita tetap bisa menggunakan nya pada macOS dengan menggunakan wine.
slug: cara-install-winbox-pada-macos
tags:
- macos
- big-sur
- wine
title: Cara install Winbox pada macOS
type: post
---

**winbox** adalah sebuah aplikasi yang memungkinkan kita untuk mengadministrasi perangkat `mikrotik` menggunakan GUI secara cepat dan sederhana.

Namun saat artikel ini di tulis **winbox** masih belum mendukung sistem operasi macOS. Agar dapat berjalan pada sistem operasi macOS kita harus menginstall wine terlebih dahulu, kemudian baru kita dapat menjalankan **winbox** di atas wine.

Dibawah ini adalah cara install winbox pada macOS.

## Instalasi Wine
Kita akan menginstall wine menggunakan paket manager `homebrew`. Jika `homebrew` belum tersedia lakukan peritah berikut menginstall paket `homebrew`
```console
~$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

kemudian install `wine` dan paket pendukung
```console
~$ brew install --cask wine-stable
~$ brew install winetricks 
~$ winetricks corefonts mshtml
```

## Menjalankan Winbox melalui terminal
Setelah `wine` dan paket pendukungnya telah terinstall maka kita harus mendownload `Winbox` ke lokal directory kita agar dapat di jalankan oleh wine.
```console
~$ mkdir -p /Users/userx/Scripts/app/
~$ wget https://mt.lv/winbox64 -P /Users/userx/Scripts/app/
~$ wine64 /Users/userx/Scripts/app/winbox64
```

## Menjalankan Winbox melalui Spotlight search field
Agar winbox dapat di panggil menggunakan Spotlight search field (Command + Space) maka kita harus membuat `app` nya melalui:
`automator` > `File` > `New` > `Aplication` > `Run Shell Script`

Kemudian pada kotak dialog `Run Shell Script` isi dengan perintah berikut.
```console
export LC_CTYPE="en_US.UTF-8"
export WINEDLLOVERRIDES="mscoree,mshtml="
export WINEDEBUG="fixme-esync"

wine64 /Users/userx/Scripts/app/winbox64
```

Jika sudah maka klik `save`. Simpan dengan nama `Winbox` dan lokasi `Application`.
