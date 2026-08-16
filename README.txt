WEB PUSH HOST — SISTEM ABSENSI GURU
====================================

TUJUAN
------
PushHost adalah halaman top-level HTTPS untuk aktivasi Web Push.
Google Apps Script tetap menjadi backend/database. OneSignal menjadi
transport Web Push.

FILE
----
activate.html
  Halaman aktivasi yang dibuka guru dari link Admin.

config.js
  Hanya berisi OneSignal App ID (publik) dan URL Web App GAS.
  JANGAN menaruh OneSignal REST API Key di sini.

OneSignalSDKWorker.js
  Service Worker resmi OneSignal. File ini harus berada pada origin
  PushHost yang sama dengan activate.html dan dapat diakses langsung.

ARSITEKTUR
----------
Admin GAS
  -> membuat token aktivasi
  -> link diarahkan ke PushHost/activate.html

Guru
  -> buka link
  -> PushHost memvalidasi token melalui endpoint publik terbatas GAS
  -> OneSignal meminta permission pada top-level origin
  -> Guru klik IZINKAN
  -> OneSignal membuat Web Push Subscription
  -> OneSignal.login() memakai External ID:
       MADRASAH_ID:KODE_GURU
  -> PushHost memberi tahu GAS bahwa perangkat sudah terdaftar
  -> GAS menandai perangkat aktif

PERSYARATAN ONESIGNAL
---------------------
1. Buat Web Push App di OneSignal.
2. Site URL harus sama persis dengan origin PushHost.
3. HTTPS wajib.
4. Upload OneSignalSDKWorker.js ke ROOT PushHost.
5. Di OneSignal Web Settings, Service Worker file:
     OneSignalSDKWorker.js
   Path:
     /
   Scope:
     /
6. App ID dimasukkan ke PushHost/config.js.

PENTING
-------
- API Key OneSignal TIDAK diletakkan di PushHost.
- API Key server nanti disimpan di Apps Script Script Properties:
    ONESIGNAL_API_KEY
- PushHost adalah static site. Tidak membutuhkan server-side code.
- Browser yang sudah menyimpan permission denied harus di-reset untuk
  ORIGIN PUSHHOST tersebut. Jangan reset origin GAS lagi.

SETELAH HOSTING
---------------
Isi PushHost/config.js:

ONESIGNAL_APP_ID: "App ID OneSignal"
GAS_WEB_APP_URL: "URL Web App GAS /exec"

Kemudian pada GAS Config.js isi:

PUSH_HOST_URL: "https://origin-pushhost-anda"

Tanpa slash di akhir.

REDEPLOY GAS
------------
Setelah Config.js + Kode.js diganti:
Deploy -> Manage deployments -> Edit -> New version -> Deploy.

TES
---
1. Admin login ke GAS.
2. Admin -> Notifikasi Guru.
3. Buat link aktivasi.
4. Link harus menuju:
     https://PUSHHOST/activate.html?activate=...&m=...
5. Buka link pada Chrome normal, bukan Incognito.
6. Pastikan origin PushHost belum denied.
7. Klik IZINKAN NOTIFIKASI.
8. Chrome harus menampilkan prompt permission.
9. Klik IZINKAN.
10. Tunggu sampai muncul:
      NOTIFIKASI AKTIF
11. Kembali ke Admin -> Notifikasi Guru.
12. Guru tersebut harus menunjukkan 1 perangkat aktif.

CATATAN
-------
Patch ini menyelesaikan LAYER AKTIVASI Web Push.
Pengiriman otomatis ABSENSI -> OneSignal API masih merupakan tahap
berikutnya. Backend harus memakai ONESIGNAL_API_KEY dan target External ID
yang sama:
    MADRASAH_ID:KODE_GURU

Referensi resmi:
https://documentation.onesignal.com/docs/en/web-push-custom-code-setup
https://documentation.onesignal.com/docs/en/onesignal-service-worker
