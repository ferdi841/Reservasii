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
      width: auto;
      display: inline-block;
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

      <label>
        <input type="radio" name="metode" value="QRIS DANA" checked />
        QRIS DANA
      </label>
      <label>
        <input type="radio" name="metode" value="Bayar di Tempat" />
        Bayar di Tempat
      </label><br/>

      <button type="button" onclick="startBooking()">Mulai Reservasi</button>
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

  <script>
    const historyBox = document.getElementById('historyBox');
    const countdownElement = document.getElementById('timer');
    const selectSlot = document.getElementById('slotWaktu');

    const semuaReservasi = JSON.parse(localStorage.getItem("reservasi")) || [];
    let timerData = JSON.parse(localStorage.getItem("timerData")) || null;

    const slots = [
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

    function tampilkanRiwayat() {
      if (semuaReservasi.length === 0) {
        historyBox.innerHTML = "<p>Belum ada reservasi.</p>";
        return;
      }
      historyBox.innerHTML = "";
      semuaReservasi.forEach(r => {
        historyBox.innerHTML += `<p>✅ *${r.nama}* – ${r.tanggal} jam ${r.slot} – <em>${r.metode}</em> (berakhir ${r.berakhir})</p>`;
      });
    }

    function updateSlotWaktu() {
      const selectedDate = document.getElementById('tanggal').value;
      selectSlot.innerHTML = "";
      slots.forEach(slot => {
        const dipakai = semuaReservasi.some(r => r.tanggal === selectedDate && r.slot === slot);
        const aktif = timerData && timerData.slot === slot && timerData.tanggal === selectedDate && new Date().getTime() < timerData.end;
        const disabled = dipakai || aktif;
        const option = document.createElement("option");
        option.value = slot;
        option.textContent = disabled ? `${slot} (TERISI)` : slot;
        if (disabled) option.disabled = true;
        selectSlot.appendChild(option);
      });
    }

    function mulaiCountdown(endTime) {
      function updateTimer() {
        const now = new Date().getTime();
        const distance = endTime - now;

        if (distance <= 0) {
          countdownElement.textContent = "Waktu habis!";
          clearInterval(timerInterval);
          localStorage.removeItem("timerData");
          return;
        }

        const minutes = Math.floor(distance / (1000 * 60));
        const seconds = Math.floor((distance % (1000 * 60)) / 1000);
        countdownElement.textContent = `Waktu tersisa: ${minutes}m ${seconds < 10 ? '0' : ''}${seconds}s`;
      }

      updateTimer();
      const timerInterval = setInterval(updateTimer, 1000);
    }

    if (timerData && new Date().getTime() < timerData.end) {
      mulaiCountdown(timerData.end);
    }

    tampilkanRiwayat();

    document.getElementById('tanggal').addEventListener('change', updateSlotWaktu);

    function startBooking() {
      const nama = document.getElementById('nama').value.trim();
      const tanggal = document.getElementById('tanggal').value;
      const slot = document.getElementById('slotWaktu').value;
      const metode = document.querySelector('input[name="metode"]:checked').value;

      if (!nama || !tanggal || !slot) {
        alert("Isi semua data terlebih dahulu.");
        return;
      }

      const now = new Date();
      const waktuSelesai = new Date(now.getTime() + 60 * 60 * 1000);
      const jamSelesai = waktuSelesai.toLocaleTimeString('id-ID', {
        hour: '2-digit',
        minute: '2-digit'
      });

      const dataBaru = { nama, tanggal, slot, metode, berakhir: jamSelesai };
      semuaReservasi.push(dataBaru);
      localStorage.setItem("reservasi", JSON.stringify(semuaReservasi));

      timerData = { tanggal, slot, end: waktuSelesai.getTime() };
      localStorage.setItem("timerData", JSON.stringify(timerData));

      tampilkanRiwayat();
      mulaiCountdown(timerData.end);
      updateSlotWaktu();

      const pesan = `Halo HB Barbershop,%0ASaya *${nama}* ingin reservasi:%0A📅 Tanggal: ${tanggal}%0A⏰ Jam: ${slot}%0A💳 Metode: ${metode}%0A⏳ Berlaku hingga *${jamSelesai}*`;
      const waUrl = `https://wa.me/6289518371444?text=${pesan}`;
      window.open(waUrl, '_blank');
    }
  </script>
</body>
</html>

