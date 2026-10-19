---
title: Dashboard KKN
date: 2026-09-22
tags: []
status: draft
dg-publish: "true"
---

<div class="glass-grid-container" style="background-color: #1e1e2e !important; color: #f0f0f0 !important; padding: 18px; border-radius: 14px; border: 1px solid #313244;">

  <!-- KARTU WIDGET STATISTIK & PETA TIRUAN -->
  <div class="kkn-map-dashboard-card" style="background-color: #181825 !important; color: #ffffff !important; padding: 16px; border-radius: 10px; margin-bottom: 20px; border: 1px solid rgba(255,255,255,0.08);">
    <div style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px; border-bottom: 1px solid rgba(255,255,255,0.06); padding-bottom: 12px; margin-bottom: 12px;">
      <div style="display: flex; align-items: center; gap: 10px;">
        <span style="font-size: 22px;">🗺️</span>
        <div>
          <h3 style="margin: 0; font-size: 13px; font-weight: 700; color: #f8fafc; text-transform: uppercase; letter-spacing: 0.6px;">Peta Sebaran Lokasi KKN Kabupaten Wonogiri</h3>
          <p style="margin: 2px 0 0 0; font-size: 10.5px; color: #a6adc8;">Ringkasan visual wilayah sebaran titik kegiatan KKN Mahasiswa.</p>
        </div>
      </div>
      <div style="display: flex; align-items: center; gap: 8px; flex-wrap: wrap;">
        <span style="font-size: 10px; font-weight: 600; padding: 3px 9px; border-radius: 20px; background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.08);">📍 25 Kecamatan</span>
        <span style="font-size: 10px; font-weight: 600; padding: 3px 9px; border-radius: 20px; background: rgba(245, 158, 11, 0.12); border: 1px solid rgba(245, 158, 11, 0.35); color: #fbbf24;">🟠 Proses</span>
        <span style="font-size: 10px; font-weight: 600; padding: 3px 9px; border-radius: 20px; background: rgba(16, 185, 129, 0.12); border: 1px solid rgba(16, 185, 129, 0.35); color: #34d399;">🟢 Selesai</span>
      </div>
    </div>
    
    <!-- Stage Peta Visual Statis yang Elegan -->
    <div style="position: relative; width: 100%; height: 320px; border-radius: 8px; background: #0b0f19; border: 1px solid rgba(255,255,255,0.1); display: flex; align-items: center; justify-content: center; flex-direction: column; gap: 8px; overflow: hidden;">
      <div style="font-size: 32px; animation: pulse 2s infinite;">🛰️</div>
      <div style="font-size: 12px; font-weight: 600; color: #e2e8f0;">Citra Wilayah Kabupaten Wonogiri & Waduk Gajah Mungkur</div>
      <div style="font-size: 10.5px; color: #94a3b8; text-align: center; max-width: 400px; padding: 0 10px;">
        Peta interaktif berbasis Leaflet aktif di dalam aplikasi Obsidian. Versi web statis ini menampilkan rekapitulasi data secara optimal.
      </div>
    </div>
  </div>

  <!-- TABEL REKAPITULASI DATA KKN (Gaya Flex-Grid) -->
  <div class="flex-grid-table" style="display: flex; flex-direction: column; width: 100%; overflow-x: auto;">
    
    <!-- Header Tabel -->
    <div style="display: grid; grid-template-columns: 85px 90px 95px 1fr 110px 150px 105px 2fr; align-items: center; border-bottom: 1px solid rgba(255,255,255,0.08); padding-bottom: 8px; font-size: 10px; font-weight: 500; color: #a6adc8; text-transform: uppercase; letter-spacing: 0.5px;">
      <div style="padding: 0 6px;">Kode</div>
      <div style="padding: 0 6px;">PTN</div>
      <div style="text-align: center; padding: 0 6px;">Status</div>
      <div style="padding: 0 6px;">PIC</div>
      <div style="padding: 0 6px;">No. HP</div>
      <div style="padding: 0 6px;">Kecamatan</div>
      <div style="padding: 0 6px;">Usulan</div>
      <div style="padding: 0 6px;">Lokasi / Desa</div>
    </div>

    <!-- Baris Contoh 1 -->
    <div style="display: grid; grid-template-columns: 85px 90px 95px 1fr 110px 150px 105px 2fr; align-items: center; border-bottom: 0.5px solid rgba(255,255,255,0.04); padding: 8px 0; font-size: 11.5px; transition: background 0.15s ease;">
      <div style="padding: 0 6px; font-weight: 600; color: #f0f0f0;">KKN-01</div>
      <div style="padding: 0 6px; font-weight: 500; color: #a6adc8;">UNS</div>
      <div style="padding: 0 6px; display: flex; justify-content: center;"><span style="color: #e67e22; background: rgba(243, 156, 18, 0.08); padding: 2px 8px; border-radius: 4px; font-size: 10px; font-weight: 600;">Proses</span></div>
      <div style="padding: 0 6px; color: #a6adc8;">Budi Santoso</div>
      <div style="padding: 0 6px; color: #a6adc8;">08123456789</div>
      <div style="padding: 0 6px;"><span style="background: #ff3b75; color: #fff; padding: 1px 6px; border-radius: 3px; font-size: 10px; font-weight: 600;">Baturetno</span></div>
      <div style="padding: 0 6px; color: #a6adc8; font-size: 10.5px;">2026-09-01</div>
      <div style="padding: 0 6px; color: #f0f0f0; font-size: 11px;">Desa Baturetno</div>
    </div>

    <!-- Baris Contoh 2 -->
    <div style="display: grid; grid-template-columns: 85px 90px 95px 1fr 110px 150px 105px 2fr; align-items: center; border-bottom: 0.5px solid rgba(255,255,255,0.04); padding: 8px 0; font-size: 11.5px; transition: background 0.15s ease;">
      <div style="padding: 0 6px; font-weight: 600; color: #f0f0f0;">KKN-02</div>
      <div style="padding: 0 6px; font-weight: 500; color: #a6adc8;">UGM</div>
      <div style="padding: 0 6px; display: flex; justify-content: center;"><span style="color: #27ae60; background: rgba(46, 204, 113, 0.08); padding: 2px 8px; border-radius: 4px; font-size: 10px; font-weight: 600;">Selesai</span></div>
      <div style="padding: 0 6px; color: #a6adc8;">Siti Aminah</div>
      <div style="padding: 0 6px; color: #a6adc8;">08987654321</div>
      <div style="padding: 0 6px;"><span style="background: #ff9f1a; color: #fff; padding: 1px 6px; border-radius: 3px; font-size: 10px; font-weight: 600;">Wonogiri</span></div>
      <div style="padding: 0 6px; color: #a6adc8; font-size: 10.5px;">2026-09-05</div>
      <div style="padding: 0 6px; color: #f0f0f0; font-size: 11px;">Kelurahan Wonogiri</div>
    </div>

  </div>

</div>

> **💡 Catatan Pengelola:** Halaman ini disinkronkan secara otomatis. Anda dapat memperbarui data lengkap melalui aplikasi Obsidian kapan saja.
