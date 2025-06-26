<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>HB Barbershop</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: url('IMG_2174.png') no-repeat center center/cover;
      color: white;
      text-align: center;
    }
    header {
      padding: 30px;
      background-color: rgba(0, 0, 0, 0.6);
    }
    header img {
      width: 150px;
    }
    .content {
      background-color: rgba(0, 0, 0, 0.6);
      padding: 20px;
      margin: 20px auto;
      max-width: 600px;
      border-radius: 12px;
    }
    .countdown {
      font-size: 24px;
      margin-top: 15px;
      color: #ffcc00;
    }
    .history {
      background-color: #222;
      margin-top: 20px;
      padding: 10px;
      border-radius: 8px;
      font-size: 14px;
      max-height: 200px;
      overflow-y: auto;
      text-align: left;
    }
    button {
      background: #ffcc00;
      border: none;
      padding: 10px 20px;
      font-size: 16px;
      margin-top: 10px;
      cursor: pointer;
      border-radius: 5px;
    }
    input, select {
      padding: 10px;
      margin: 8px 0;
      width: 90%;
      max-width: 300px;
      border-radius: 5px;
      border: none;
      display: block;
      margin-left: auto;
      margin-right: auto;
      color: black;
    }
    label {
      color: white;
    }
    .qr-image {
      width: 200px;
      margin-top: 15px;
    }
  </style>
</head>
<body>
  <header>
    <img src="IMG_2174.png" alt="HB Logo" />
    <h1>HB Barbershop</h1>
    <p>Potongan Rapi, Gaya Pasti</p>
  </header>

  <div class="content">
    <h2>Reservasi (Slot Waktu)</h2>
    <form id="bookingForm">
      <input type="text" id="nama" placeholder="Nama Lengkap" required /><br />
      <input type="date" id="tanggal" required /><br />
      <select id="slotWaktu" required></select><br />
      <label><input type="radio" name="metode" value="QRIS DANA" checked /> QRIS DANA</label>
      <label><input type="radio" name="metode" value="Bayar di Tempat" /> Bayar di Tempat</label><br/>
      <button type="submit">Mulai Reservasi</button>
    </form>
    <div class="countdown" id="timer"></div>
  </div>

  <div class="content">
    <h3>QR Pembayaran (QRIS DANA)</h3>
    <img src="IMG_2176.jpeg" alt="QR Dana" class="qr-image" />
    <p>Silakan pindai QR untuk pembayaran via DANA</p>
  </div>

  <div class="content">
    <h3>Riwayat Reservasi</h3>
    <div class="history" id="historyBox">
      <p>Belum ada reservasi.</p>
    </div>
  </div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-app.js";
    import { getAnalytics } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-analytics.js";
    import {
      getDatabase,
      ref,
      set,
      get,
      onValue
    } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-database.js";

    const firebaseConfig = {
      apiKey: "AIzaSyCfpoT8OIhOmPgM14wmVEWBNNw5Odmstzs",
      authDomain: "hb-barbershop-f37b7.firebaseapp.com",
      projectId: "hb-barbershop-f37b7",
      storageBucket: "hb-barbershop-f37b7.appspot.com",
      messagingSenderId: "497791870137",
      appId: "1:497791870137:web:c17ab29689f65c2baee768",
      measurementId: "G-LK5MM0LHC0"
    };

    const app = initializeApp(firebaseConfig);
    const analytics = getAnalytics(app);
    const db = getDatabase(app);

    const slotList = [
      "10:00 - 12:00",
      "12:00 - 13:00",
      "13:00 - 14:00",
      "14:00 - 15:00",
      "15:00 - 16:00",
      "16:00 - 17:00",
      "17:00 - 18:00",
      "19:00 - 20:00",
      "20:00 - 21:00",
      "21:00 - 22:00"
    ];

    const slotSelect = document.getElementById("slotWaktu");
    const form = document.getElementById("bookingForm");
    const timerEl = document.getElementById("timer");
    const riwayatBox = document.getElementById("historyBox");
    const tglEl = document.getElementById("tanggal");

    // Isi tanggal hari ini
    const today = new Date().toISOString().split("T")[0];
    tglEl.value = today;

    function updateSlotOptions() {
      const selectedDate = tglEl.value;
      slotSelect.innerHTML = "";
      slotList.forEach(slot => {
        const key = `${selectedDate}_${slot.replace(/[^0-9]/g, '')}`;
        const slotRef = ref(db, 'reservasi/' + key);
        get(slotRef).then(snapshot => {
          const option = document.createElement("option");
          option.value = slot;
          if (snapshot.exists()) {
            option.textContent = `${slot} (TERISI)`;
            option.disabled = true;
          } else {
            option.textContent = slot;
          }
          slotSelect.appendChild(option);
        });
      });
    }

    tglEl.addEventListener('change', updateSlotOptions);

    function tampilkanRiwayat() {
      const reservasiRef = ref(db, 'reservasi');
      onValue(reservasiRef, (snapshot) => {
        const data = snapshot.val();
        if (!data) {
          riwayatBox.innerHTML = "<p>Belum ada reservasi.</p>";
          return;
        }
        riwayatBox.innerHTML = "";
        Object.values(data).forEach(r => {
          const jamAkhir = new Date(r.berakhir).toLocaleTimeString("id-ID", {hour: '2-digit', minute: '2-digit'});
          riwayatBox.innerHTML += `<p>✅ <strong>${r.nama}</strong> – ${r.tanggal} jam ${r.slot} – <em>${r.metode}</em> (berakhir ${jamAkhir})</p>`;
        });
      });
    }

    function mulaiTimer(endTime) {
      clearInterval(window.timerInterval);
      function update() {
        const now = new Date().getTime();
        const diff = endTime - now;
        if (diff <= 0) {
          timerEl.textContent = "Waktu habis!";
          clearInterval(window.timerInterval);
        } else {
          const m = Math.floor(diff / 60000);
          const s = Math.floor((diff % 60000) / 1000);
          timerEl.textContent = `Waktu tersisa: ${m}m ${s < 10 ? '0' : ''}${s}s`;
        }
      }
      update();
      window.timerInterval = setInterval(update, 1000);
    }

    form.addEventListener("submit", function(e) {
      e.preventDefault();
      const nama = document.getElementById("nama").value.trim();
      const tanggal = tglEl.value;
      const slot = slotSelect.value;
      const metode = document.querySelector('input[name="metode"]:checked').value;

      if (!nama || !tanggal || !slot) {
        alert("Lengkapi semua data.");
        return;
      }

      const now = new Date();
      const waktuSelesai = new Date(now.getTime() + 60 * 60 * 1000);
      const jamSelesai = waktuSelesai.toISOString();
      const key = `${tanggal}_${slot.replace(/[^0-9]/g, '')}`;

      set(ref(db, 'reservasi/' + key), {
        nama,
        tanggal,
        slot,
        metode,
        berakhir: jamSelesai
      }).then(() => {
        mulaiTimer(waktuSelesai.getTime());
        updateSlotOptions();
        tampilkanRiwayat();

        const waMsg = `Halo HB Barbershop,%0ASaya *${nama}* ingin reservasi:%0A📅 Tanggal: ${tanggal}%0A⏰ Jam: ${slot}%0A💳 Metode: ${metode}%0A⏳ Berlaku hingga *${waktuSelesai.toLocaleTimeString('id-ID', {hour: '2-digit', minute: '2-digit'})}*%0A*Jika capster belum membalas dalam 10 menit, berarti masih antrian 🙏*`;
        const waUrl = `https://wa.me/6289518371444?text=${waMsg}`;
        window.open(waUrl, '_blank');
      });
    });

    updateSlotOptions();
    tampilkanRiwayat();
  </script>
</body>
</html>
