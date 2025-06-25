<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>HB Barbershop</title>

  <!-- Firebase SDK -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
    import { getDatabase, ref, get, set, onValue } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js";

    // ⚙️ Masukkan firebaseConfig kamu di sini:
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

    window.appData = { db, firebaseConfig };
  </script>

  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: url('IMG_2174.png') no-repeat center center/cover;
      color: white;
      text-align: center;
    }
    .content { background: rgba(0,0,0,0.6); margin:20px auto; padding:20px; max-width:600px; border-radius:12px; }
    input, select { width:90%; padding:10px; margin:8px auto; border-radius:5px; }
    button { padding:10px 20px; background:#ffcc00; border:none; border-radius:5px; cursor:pointer; }
    .countdown { font-size:24px; color:#ffcc00; margin-top:15px; }
    .history { background:#222; padding:10px; border-radius:8px; font-size:14px; text-align:left; max-height:200px; overflow-y:auto; }
    .qr-image { width:200px; margin-top:15px; }
  </style>
</head>
<body>

  <div class="content">
    <h2>Reservasi HB Barbershop</h2>
    <input type="text" id="nama" placeholder="Nama Lengkap" />
    <input type="date" id="tanggal" />
    <select id="slot"><option value="">Pilih slot waktu</option></select>
    <select id="metode">
      <option value="QRIS DANA">QRIS DANA</option>
      <option value="Bayar di Tempat">Bayar di Tempat</option>
    </select>
    <button onclick="booking()">Booking Sekarang</button>
    <div class="countdown" id="timer"></div>
  </div>

  <div class="content">
    <h3>QR Pembayaran (QRIS DANA)</h3>
    <img src="IMG_2176.jpeg" class="qr-image" alt="QR DANA">
  </div>

  <div class="content">
    <h3>Riwayat Reservasi</h3>
    <div class="history" id="riwayat"></div>
  </div>

<script type="module">
  import { ref, get, set, onValue } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js";
  const { db } = window.appData;

  const slots = [
    "10:00 - 12:00", "12:00 - 13:00", "13:00 - 14:00", "14:00 - 15:00",
    "15:00 - 16:00", "16:00 - 17:00", "17:00 - 18:00", "19:00 - 20:00",
    "20:00 - 21:00", "21:00 - 22:00"
  ];
  const slotSel = document.getElementById('slot');
  const tanggalEl = document.getElementById('tanggal');
  const riwayatEl = document.getElementById('riwayat');
  const timerEl = document.getElementById('timer');

  tanggalEl.addEventListener('change', loadSlots);
  loadSlots();

  let timerInterval = null;

  function loadSlots() {
    const tgl = tanggalEl.value;
    slotSel.innerHTML = `<option value="">Muat...</option>`;
    get(ref(db, `reservasi/${tgl}`)).then(snap => {
      const booked = snap.val() || {};
      slotSel.innerHTML = '<option value="">Pilih slot waktu</option>';
      slots.forEach(slot => {
        const opt = document.createElement('option');
        opt.value = slot;
        opt.textContent = booked[slot] ? `${slot} (TERISI)` : slot;
        if (booked[slot]) opt.disabled = true;
        slotSel.appendChild(opt);
      });
      renderRiwayat(tgl);
    });
  }

  function booking() {
    const nama = document.getElementById('nama').value.trim();
    const tgl = tanggalEl.value;
    const slot = slotSel.value;
    const metode = document.getElementById('metode').value;
    if (!nama || !tgl || !slot) return alert('Lengkapi data.');

    const path = `reservasi/${tgl}/${slot}`;
    get(ref(db, path)).then(snap => {
      if (snap.exists()) return alert('Slot sudah dibooking');
      set(ref(db, path), { nama, metode, waktu: Date.now() }).then(() => {
        alert('Booking berhasil! cek WhatsApp.');
        loadSlots(); renderRiwayat(tgl);
        window.open(`https://wa.me/6289518371444?text=${encodeURIComponent(
          `Halo HB Barbershop,%0ASaya *${nama}* reservasi tanggal ${tgl} jam ${slot} via ${metode}.`
        )}`, '_blank');
      });
    });
    startTimer();
  }

  function renderRiwayat(tgl) {
    riwayatEl.innerHTML = '';
    get(ref(db, `reservasi/${tgl}`)).then(snap => {
      const data = snap.val() || {};
      const keys = Object.keys(data);
      if (!keys.length) riwayatEl.innerHTML = '<p>Belum ada reservasi.</p>';
      keys.forEach(slot => {
        const r = data[slot];
        riwayatEl.innerHTML += `<p>✅ ${slot} — ${r.nama} (${r.metode})</p>`;
      });
    });
  }

  function startTimer() {
    if (timerInterval) clearInterval(timerInterval);
    let waktuLeft = 3600;
    timerEl.textContent = '';
    timerInterval = setInterval(() => {
      const m = Math.floor(waktuLeft / 60);
      const s = waktuLeft % 60;
      timerEl.textContent = `Waktu tersisa: ${m}m ${s<10?'0':''}${s}s`;
      if (--waktuLeft < 0) clearInterval(timerInterval);
    }, 1000);
  }
</script>

</body>
</html>
