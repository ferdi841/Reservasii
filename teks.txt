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
    input, label {
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
    <h2>Reservasi</h2>
    <form id="bookingForm">
      <input type="text" id="nama" placeholder="Nama Lengkap" required /><br />
      <input type="date" id="tanggal" required /><br />
      <input type="time" id="waktu" required /><br />

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

    // Ambil data dari localStorage jika ada
    let semuaReservasi = JSON.parse(localStorage.getItem("reservasi")) || [];
    let timerEndTime = localStorage.getItem("timerEnd");

    // Fungsi untuk menampilkan riwayat
    function tampilkanRiwayat() {
      if (semuaReservasi.length === 0) {
        historyBox.innerHTML = "<p>Belum ada reservasi.</p>";
        return;
      }
      historyBox.innerHTML = "";
      semuaReservasi.forEach(r => {
        historyBox.innerHTML += `<p>✅ *${r.nama}* – ${r.tanggal} pukul ${r.waktu} – <em>${r.metode}</em> (berakhir ${r.berakhir})</p>`;
      });
    }

    // Fungsi countdown permanen
    function mulaiCountdown() {
      if (!timerEndTime) return;

      function updateTimer() {
        const now = new Date().getTime();
        const distance = timerEndTime - now;

        if (distance <= 0) {
          countdownElement.textContent = "Waktu habis!";
          clearInterval(timerInterval);
          localStorage.removeItem("timerEnd");
          return;
        }

        const minutes = Math.floor(distance / (1000 * 60));
        const seconds = Math.floor((distance % (1000 * 60)) / 1000);
        countdownElement.textContent = `Waktu tersisa: ${minutes}m ${seconds < 10 ? '0' : ''}${seconds}s`;
      }

      updateTimer();
      const timerInterval = setInterval(updateTimer, 1000);
    }

    // Jalankan countdown jika tersimpan
    if (timerEndTime) {
      timerEndTime = parseInt(timerEndTime);
      mulaiCountdown();
    }

    tampilkanRiwayat();

    function startBooking() {
      const nama = document.getElementById('nama').value.trim();
      const tanggal = document.getElementById('tanggal').value;
      const waktu = document.getElementById('waktu').value;
      const metode = document.querySelector('input[name="metode"]:checked').value;

      if (!nama || !tanggal || !waktu) {
        alert("Isi semua data terlebih dahulu.");
        return;
      }

      const duplikat = semuaReservasi.find(r => r.tanggal === tanggal && r.waktu === waktu);
      if (duplikat) {
        alert("Waktu tersebut sudah dipesan. Pilih waktu lain.");
        return;
      }

      const sekarang = new Date();
      const waktuSelesai = new Date(sekarang.getTime() + 60 * 60 * 1000);
      const jamSelesai = waktuSelesai.toLocaleTimeString('id-ID', {
        hour: '2-digit',
        minute: '2-digit'
      });

      // Simpan ke localStorage
      const reservasiBaru = { nama, tanggal, waktu, metode, berakhir: jamSelesai };
      semuaReservasi.push(reservasiBaru);
      localStorage.setItem("reservasi", JSON.stringify(semuaReservasi));

      timerEndTime = waktuSelesai.getTime();
      localStorage.setItem("timerEnd", timerEndTime);

      tampilkanRiwayat();
      mulaiCountdown();

      // Kirim ke WA
      const pesan = `Halo HB Barbershop,%0ASaya *${nama}* ingin reservasi pada:%0A📅 Tanggal: ${tanggal}%0A⏰ Jam: ${waktu}%0A💳 Metode Pembayaran: ${metode}%0A%0A⏳ Berlaku hingga jam *${jamSelesai}*`;
      const waUrl = `https://wa.me/6289518371444?text=${pesan}`;
      window.open(waUrl, '_blank');
    }
  </script>
</body>
</html>
