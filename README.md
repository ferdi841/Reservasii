<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
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

    button {
      background: #ffcc00;
      border: none;
      padding: 10px 20px;
      font-size: 16px;
      margin-top: 10px;
      cursor: pointer;
      border-radius: 5px;
    }

    input {
      padding: 10px;
      margin: 8px 0;
      width: 90%;
      max-width: 300px;
      border-radius: 5px;
      border: none;
    }

    a {
      color: #ffcc00;
      text-decoration: none;
    }
  </style>
</head>
<body>

  <header>
    <img src="IMG_2174.png" alt="HB Logo">
    <h1>HB Barbershop</h1>
    <p>Potongan Rapi, Gaya Pasti</p>
  </header>

  <div class="content">
    <h2>Reservasi</h2>
    <form id="bookingForm">
      <input type="text" id="nama" placeholder="Nama Lengkap" required><br>
      <input type="date" id="tanggal" required><br>
      <input type="time" id="waktu" required><br>
      <button type="button" onclick="startBooking()">Mulai Reservasi</button>
    </form>
    <div class="countdown" id="timer"></div>
  </div>

  <div class="content">
    <h3>Pembayaran & Kontak</h3>
    <p>📱 Dana: 089518371444</p>
    <p>💬 WhatsApp: <a href="https://wa.me/6289518371444" target="_blank">Klik untuk Chat</a></p>
  </div>

  <script>
    function startBooking() {
      const nama = document.getElementById('nama').value;
      const tanggal = document.getElementById('tanggal').value;
      const waktu = document.getElementById('waktu').value;

      if (!nama || !tanggal || !waktu) {
        alert("Isi semua data terlebih dahulu.");
        return;
      }

      // Dapatkan waktu selesai (1 jam dari sekarang)
      const sekarang = new Date();
      const waktuSelesai = new Date(sekarang.getTime() + 60 * 60 * 1000);
      const jamSelesai = waktuSelesai.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });

      // Mulai countdown di web
      let timeLeft = 60 * 60;
      const countdownElement = document.getElementById('timer');

      function updateTimer() {
        const minutes = Math.floor(timeLeft / 60);
        const seconds = timeLeft % 60;
        countdownElement.textContent = `Waktu tersisa: ${minutes}m ${seconds < 10 ? '0' : ''}${seconds}s`;

        if (timeLeft <= 0) {
          clearInterval(timerInterval);
          countdownElement.textContent = "Waktu habis!";
        }
        timeLeft--;
      }

      updateTimer();
      const timerInterval = setInterval(updateTimer, 1000);

      // Buat pesan WhatsApp dengan jam selesai
      const pesan = `Halo HB Barbershop,%0ASaya *${nama}* ingin reservasi pada:%0A📅 Tanggal: ${tanggal}%0A⏰ Jam: ${waktu}%0A%0A⏳ Waktu reservasi berlaku hingga jam *${jamSelesai}
      cek sisa waktu anda di website *%0A%0AWebsite: [https://ferdi841.github.io/Reservasii/]`;
      const waUrl = `https://wa.me/6289518371444?text=${pesan}`;
      window.open(waUrl, '_blank');
    }
  </script>

</body>
</html>
