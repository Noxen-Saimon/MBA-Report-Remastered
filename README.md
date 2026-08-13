# MBA Report Remastered

Web app untuk membantu pencatatan laporan bacaan harian Mari Baca Alkitab Group 1.

Dibuat menggunakan SvelteKit + Svelte 5, TypeScript, dan Turso.

## Fitur

- Daftar anggota dan status bacaan
- Bacaan Alkitab otomatis berdasarkan tanggal
- Riwayat laporan
- Sistem host mingguan
- Login berdasarkan nama anggota
- Akses edit hanya untuk host yang sedang bertugas
- Copy laporan ke format WhatsApp
- Copy menu bacaan
- Light / dark mode
- Emoji harian berdasarkan tanggal

## Menjalankan Project

Install dependency:

```bash
npm install
````
---
Jalankan development server:

```bash
npm run dev
```

Build:

```bash
npm run build
```

Preview production build:

```bash
npm run preview
```

## Environment

Project menggunakan Turso sebagai database.

Buat file `.env` di root project:

```env
VITE_TURSO_URL=
VITE_TURSO_TOKEN=
```

Jangan masukkan token asli ke repository.

## Stack

* SvelteKit
* Svelte 5
* TypeScript
* Turso / libSQL
* Vite

## Catatan

Project ini dibuat untuk kebutuhan internal Mari Baca Alkitab Group 1 dan masih dalam pengembangan.

````

