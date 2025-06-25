
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

      <button type="button" onclick="startBooking()">Mulai
