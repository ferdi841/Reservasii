<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>HB Barbershop</title>

  <!-- Firebase SDK -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
    import { getDatabase, ref, get, set } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js";

    const firebaseConfig = {
      apiKey: "AIzaSyCfpoT8OIhOmPgM14wmVEWBNNw5Odmstzs",
      authDomain: "hb-barbershop-f37b7.firebaseapp.com",
      databaseURL: "https://hb-barbershop-f37b7-default-rtdb.firebaseio.com",
      projectId: "hb-barbershop-f37b7",
      storageBucket: "hb-barbershop-f37b7.appspot.com",
      messagingSenderId: "497791870137",
      appId: "1:497791870137:web:c17ab29689f65c2baee768",
      measurementId: "G-LK5MM0LHC0"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);
    window.appData = { db };
  </script>

  <style>
    body {
      background-image: url('IMG_2174.png');
      background-size: cover;
      background-position: center;
      color: white;
      font-family: sans-serif;
      padding: 2rem;
      text-align: center;
    }
    form {
      background-color: rgba(0, 0, 0, 0.7);
      padding: 1.5rem;
      border-radius: 10px;
      max-width: 400px;
      margin: auto;
    }
    input, select, button {
      width: 100%;
      padding: 10px;
      margin: 0.5rem 0;
      border: none;
      border-radius: 5px;
    }
    button {
      background-color: #ffcc00;
      color: black;
      font-weight: bold;
    }
    .countdown {
      margin-top: 1rem;
      font-size: 20px;
      color: #ffcc00;
    }
    img.qr {
      width: 200px;
      margin: 1rem auto;
    }
    .riwayat {
      background: rgba(0, 0, 0, 0.7);
      padding: 1rem;
      margin-top: 1rem;
      border-radius: 10px;
      max-width: 500px;
      margin-left: auto;
      margin-right: auto;
      text-align: left;
    }
  </style>
</head>
<body>

  <h1>Reservasi HB Barbershop</h1>
  <p>Jam Operasional: 10:00 - 22:00</p>

  <form id="bookingForm">
    <input type="text" id="nama" placeholder="Nama Anda" required />
    <input type="date" id="tgl" required />
    <select id="slot" required>
      <option value="">Pilih Jam</option>
    </select>

    <select id="metode" required>
      <option value="">Metode Pembayaran</option>
      <option value="QRIS DANA">QRIS DANA</option>
      <option value="Bayar di Tempat">Bayar di Tempat</option>
    </select>

    <button type="submit">Booking Sekarang</button>
    <div class="countdown" id="timer"></div>
  </form>

  <div id="qris" style="display:none;">
    <h3>Silakan Scan QRIS DANA:</h3>
    <img src="IMG_2176.jpeg" alt="QRIS DANA" class="qr"/>
  </div>

  <div class="riwayat">
    <h3>Riwayat Reservasi Hari Ini:</h3>
    <div id="riwayatList"></div>
  </div>

  <script type="module">
    const { db } = window.appData;
    const form = document.getElementById('bookingForm');
    const qris = document.getElementById('qris');
    const riwayatList = document.getElementById('riwayatList');
    const timerEl = document.getElementById('timer');
    const slotEl = document.getElementById('slot');
    const tanggalEl = document.getElementById('tgl');

    const slots = [
      "10:00 - 12:00", "12:00 - 13:00", "13:00 - 14:00", "14:00 - 15:00",
      "15:00 - 16:00", "16:00 - 17:00", "17:00 - 18:00",
      "19:00 - 20:00", "20:00 - 21:00", "21:00 - 22:00"
    ];

    function loadSlots(tgl) {
      slotEl.innerHTML = "<option value=''>Muat slot...</option>";
      import("https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js").then(({ ref, get }) => {
        get(ref(db, `reservasi/${tgl}`)).then(snapshot => {
          const booked = snapshot.val() || {};
          slotEl.innerHTML = "<option value=''>Pilih Jam</option>";
          slots.forEach(s => {
            const opt = document.createElement("option");
            opt.value = s;
            opt.textContent = booked[s] ? s + " (TERISI)" : s;
            if (booked[s]) opt.disabled = true;
            slotEl.appendChild(opt);
          });
          showRiwayat(tgl, booked);
        });
      });
    }

    form.addEventListener("submit", function(e) {
      e.preventDefault();
      const nama = document.getElementById('nama').value;
      const tgl = document.getElementById('tgl').value;
      const slot = document.getElementById('slot').value;
      const metode = document.getElementById('metode').value;

      if (!nama || !tgl || !slot || !metode) return alert("Lengkapi semua data!");

      const path = `reservasi/${tgl}/${slot}`;
      import("https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js").then(({ ref, get, set }) => {
        get(ref(db, path)).then(snapshot => {
          if (snapshot.exists()) {
            alert("Slot ini sudah dibooking.");
          } else {
            set(ref(db, path), { nama, metode, waktu: Date.now() }).then(() => {
              alert("Reservasi berhasil!");
              if (metode === 'QRIS DANA') qris.style.display = 'block';
              loadSlots(tgl);
              startTimer();
              const waUrl = `https://wa.me/6289518371444?text=${encodeURIComponent(
                `Halo HB Barbershop,%0ASaya *${nama}* melakukan reservasi pada:%0A🗓️ Tanggal: ${tgl}%0A⏰ Jam: ${slot}%0A💳 Pembayaran: ${metode}%0A%0A*Jika capster belum membalas dalam 10 menit, berarti masih antrian. Harap tunggu, terima kasih!* 🙏`
              )}`;
              setTimeout(() => window.open(waUrl, '_blank'), 800);
            });
          }
        });
      });
    });

    function startTimer() {
      const waktuMulai = Date.now();
      localStorage.setItem("bookingTime", waktuMulai);
      runTimer(waktuMulai);
    }

    function runTimer(waktuMulai) {
      clearInterval(window.timer);
      window.timer = setInterval(() => {
        const now = Date.now();
        const diff = 3600 - Math.floor((now - waktuMulai) / 1000);
        if (diff <= 0) {
          timerEl.textContent = "⏳ Waktu habis.";
          clearInterval(window.timer);
          localStorage.removeItem("bookingTime");
        } else {
          const m = Math.floor(diff / 60);
          const s = diff % 60;
          timerEl.textContent = `⏳ Waktu tersisa: ${m}m ${s < 10 ? '0' : ''}${s}s`;
        }
      }, 1000);
    }

    function showRiwayat(tgl, data) {
      riwayatList.innerHTML = "";
      for (const jam in data) {
        const r = data[jam];
        riwayatList.innerHTML += `<p>✅ ${jam} - ${r.nama} (${r.metode})</p>`;
      }
    }

    const today = new Date().toISOString().split('T')[0];
    tanggalEl.value = today;
    loadSlots(today);
    tanggalEl.addEventListener('change', e => loadSlots(e.target.value));

    const saved = localStorage.getItem("bookingTime");
    if (saved) {
      runTimer(Number(saved));
    }
  </script>
</body>
</html>
