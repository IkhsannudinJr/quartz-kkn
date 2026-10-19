---
title: Dashboard KKN
date: 2026-09-22
tags: []
status: draft
dg-publish: "true"
---
```dataviewjs
// 1. Ambil data halaman target
const pages = dv.pages("#proyek/kkn2026").sort(p => p.tgl_usulan, 'desc');

// 2. Opsi Referensi Komponen
const statusOptions = ["Proses", "Selesai", "Usulan"];
const kecamatanOptions = [
    "Baturetno","Batuwarno","Bulukerto","Eromoko","Girimarto", 
    "Giritontro","Giriwoyo","Jatipurno", "Jatiroto", "Jatisrono",
    "Karangtengah","Kismantoro", "Manyaran","Ngadirojo","Nguntoronadi",
    "Paranggupito","Pracimantoro","Puhpelem", "Purwantoro","Selogiri",
    "Sidoharjo","Slogohimo","Tirtomoyo","Wonogiri","Wuryantoro"
];

// Helper parsing data YAML aman (String/Number/Null)
function parseYamlString(val) {
    if (val === undefined || val === null) return "";
    return String(val);
}

// Helper parsing Array Kecamatan
function parseYamlArray(val) {
    if (!val) return [];
    if (Array.isArray(val)) return val.map(v => String(v));
    if (typeof val === "object" && val.values) return Array.from(val.values).map(v => String(v));
    if (typeof val === "string") return val.split(",").map(k => k.trim());
    return [];
}

// Helper validasi tanggal anti-glitch
function formatIsoDate(dateVal) {
    if (!dateVal) return "";
    if (typeof dateVal === "object" && dateVal.toISODate) return dateVal.toISODate();
    if (dateVal.toFormat) return dateVal.toFormat("yyyy-MM-dd");
    if (typeof dateVal === "string") return dateVal.substring(0, 10);
    return "";
}

// Loader Helper Leaflet Map Engine
async function getLeafletEngine() {
    if (window.L && typeof window.L.map === "function") return window.L;
    
    // Inject Leaflet CSS
    if (!document.getElementById("kkn-leaflet-cdn-css")) {
        const link = document.createElement("link");
        link.id = "kkn-leaflet-cdn-css";
        link.rel = "stylesheet";
        link.href = "https://unpkg.com/leaflet@1.9.4/dist/leaflet.css";
        document.head.appendChild(link);
    }
    
    // Inject Leaflet JS
    if (!window.L || typeof window.L.map !== "function") {
        await new Promise((resolve, reject) => {
            const existing = document.getElementById("kkn-leaflet-cdn-js");
            if (existing) {
                if (window.L) return resolve();
                existing.addEventListener("load", resolve);
                existing.addEventListener("error", reject);
                return;
            }
            const script = document.createElement("script");
            script.id = "kkn-leaflet-cdn-js";
            script.src = "https://unpkg.com/leaflet@1.9.4/dist/leaflet.js";
            script.onload = () => resolve();
            script.onerror = () => reject(new Error("Gagal memuat pustaka peta Leaflet"));
            document.head.appendChild(script);
        });
    }
    return window.L;
}

// Koordinat Geografis Presisi 25 Kecamatan Kabupaten Wonogiri
const kecPositions = {
    "selogiri": { lat: -7.7845, lng: 110.9032, name: "Selogiri" },
    "wonogiri": { lat: -7.8142, lng: 110.9254, name: "Wonogiri" },
    "manyaran": { lat: -7.8542, lng: 110.7645, name: "Manyaran" },
    "wuryantoro": { lat: -7.9042, lng: 110.8753, name: "Wuryantoro" },
    "eromoko": { lat: -7.9863, lng: 110.8652, name: "Eromoko" },
    "pracimantoro": { lat: -8.0694, lng: 110.8228, name: "Pracimantoro" },
    "giritontro": { lat: -8.1367, lng: 110.9012, name: "Giritontro" },
    "giriwoyo": { lat: -8.0772, lng: 110.9765, name: "Giriwoyo" },
    "baturetno": { lat: -8.0068, lng: 110.9851, name: "Baturetno" },
    "batuwarno": { lat: -7.9798, lng: 111.0924, name: "Batuwarno" },
    "nguntoronadi": { lat: -7.9103, lng: 111.0375, name: "Nguntoronadi" },
    "ngadirojo": { lat: -7.8286, lng: 111.0028, name: "Ngadirojo" },
    "sidoharjo": { lat: -7.8173, lng: 111.0768, name: "Sidoharjo" },
    "girimarto": { lat: -7.7314, lng: 111.0941, name: "Girimarto" },
    "jatipurno": { lat: -7.7208, lng: 111.1685, name: "Jatipurno" },
    "jatisrono": { lat: -7.7942, lng: 111.1432, name: "Jatisrono" },
    "jatiroto": { lat: -7.8552, lng: 111.1718, name: "Jatiroto" },
    "slogohimo": { lat: -7.7792, lng: 111.2064, name: "Slogohimo" },
    "bulukerto": { lat: -7.7186, lng: 111.2589, name: "Bulukerto" },
    "puhpelem": { lat: -7.7554, lng: 111.2789, name: "Puhpelem" },
    "purwantoro": { lat: -7.8152, lng: 111.2825, name: "Purwantoro" },
    "kismantoro": { lat: -7.8687, lng: 111.2721, name: "Kismantoro" },
    "tirtomoyo": { lat: -7.9482, lng: 111.1378, name: "Tirtomoyo" },
    "karangtengah": { lat: -8.0381, lng: 111.1554, name: "Karangtengah" },
    "paranggupito": { lat: -8.2045, lng: 110.8354, name: "Paranggupito" }
};

// Builder Komponen Peta Interaktif KKN Satelit Asli
// Builder Komponen Peta Interaktif KKN Satelit Asli
function createMapDashboard(pagesData) {
    const card = document.createElement("div");
    card.className = "kkn-map-dashboard-card";
    card.style.cssText = "background-color: #181825 !important; color: #ffffff !important; padding: 12px; border-radius: 10px; margin-bottom: 20px;";

    // 1. Kumpulkan data KKN per kecamatan
    const mapKecGroups = {};
    let prosesCount = 0;
    let selesaiCount = 0;
    let totalLocations = 0;

    pagesData.forEach(p => {
        const ptn = parseYamlString(p.ptn || p.file.name);
        const kode = parseYamlString(p.kode);
        const status = parseYamlString(p.status) || "Usulan";
        const pic = parseYamlString(p.pic);
        const kontak = parseYamlString(p.kontak);
        const lokasi = parseYamlString(p.lokasi);
        const tglBuka = formatIsoDate(p.tgl_pembukaan);
        const tglTutup = formatIsoDate(p.tgl_penutupan);
        const kecList = parseYamlArray(p.kecamatan);

        if (status === "Proses") prosesCount++;
        else if (status === "Selesai") selesaiCount++;

        kecList.forEach(k => {
            const key = k.toLowerCase().replace(/\s+/g, '');
            if (kecPositions[key]) {
                if (!mapKecGroups[key]) {
                    mapKecGroups[key] = {
                        kecName: kecPositions[key].name,
                        pos: kecPositions[key],
                        items: []
                    };
                }
                mapKecGroups[key].items.push({
                    ptn, kode, status, pic, kontak, lokasi, tglBuka, tglTutup,
                    filePath: p.file.path
                });
                totalLocations++;
            }
        });
    });

    const activeDistrictsCount = Object.keys(mapKecGroups).length;

    // 2. Header Kartu Peta
    const header = document.createElement("div");
    header.className = "kkn-map-header";
    header.innerHTML = `
        <div class="kkn-map-title-group">
            <span class="kkn-map-icon">🗺️</span>
            <div>
                <h3 class="kkn-map-title">Peta Sebaran Lokasi KKN Kabupaten Wonogiri</h3>
                <p class="kkn-map-subtitle">Zoom in/out untuk melihat detail desa & kecamatan. Klik PIN untuk menyorot data KKN.</p>
            </div>
        </div>
        <div class="kkn-map-actions">
            <span class="kkn-stat-badge">📍 ${activeDistrictsCount} Kecamatan (${totalLocations} Titik)</span>
            <span class="kkn-stat-badge proses">🟠 ${prosesCount} Proses</span>
            <span class="kkn-stat-badge selesai">🟢 ${selesaiCount} Selesai</span>
            <button class="kkn-map-btn kkn-map-waduk-btn" type="button" title="Fokus & perbesar tampilan Waduk Gajah Mungkur">🌊 Waduk Gajah Mungkur</button>
            <button class="kkn-map-btn kkn-map-reset-btn" type="button" title="Pusatkan kembali ke seluruh Kabupaten Wonogiri">🎯 Reset Peta</button>
            <button class="kkn-map-mode-btn" type="button" title="Ganti tampilan lapisan peta">🛰️ Satelit Asli</button>
            <button class="kkn-map-toggle-btn" type="button">▲ Sembunyikan Peta</button>
        </div>
    `;
    card.appendChild(header);

    // 3. Stage Container Peta
    const stage = document.createElement("div");
    stage.className = "kkn-map-stage";
    stage.innerHTML = `<div style="display:flex;align-items:center;justify-content:center;height:100%;color:var(--text-muted);font-size:12px;gap:8px;">
        <span>🛰️</span><span>Memuat Citra Satelit Resolusi Tinggi Wonogiri & Waduk Gajah Mungkur...</span>
    </div>`;
    card.appendChild(stage);

    // 4. Inisialisasi Peta Leaflet
    let leafletMap = null;
    let currentTileLayer = null;

    const layersConfig = [
        {
            label: "🛰️ Satelit Asli",
            title: "Citra satelit asli Google Hybrid (Foto bumi resolusi tinggi + nama Desa, Kecamatan, dan Jalan)",
            url: "https://{s}.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
            subdomains: ["mt0", "mt1", "mt2", "mt3"],
            maxZoom: 20,
            attribution: "&copy; Google Satellite Imagery"
        },
        {
            label: "🗺️ GMaps Jalan",
            title: "Peta jalan Google Maps (Jalan raya, batas wilayah kecamatan & desa)",
            url: "https://{s}.google.com/vt/lyrs=m&x={x}&y={y}&z={z}",
            subdomains: ["mt0", "mt1", "mt2", "mt3"],
            maxZoom: 20,
            attribution: "&copy; Google Maps"
        },
        {
            label: "🌐 OpenStreetMap",
            title: "Peta komunitas OpenStreetMap lengkap batas wilayah desa & fasilitas",
            url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
            subdomains: ["a", "b", "c"],
            maxZoom: 19,
            attribution: "&copy; OpenStreetMap contributors"
        },
        {
            label: "🏞️ Satelit Murni",
            title: "Foto citra satelit murni Esri World Imagery tanpa teks/label",
            url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
            subdomains: [],
            maxZoom: 19,
            attribution: "&copy; Esri World Imagery"
        }
    ];

    let currentLayerIdx = 0;

    (async () => {
        try {
            const L = await getLeafletEngine();
            stage.innerHTML = ""; // Bersihkan placeholder loading

            // Bersihkan instance map lama jika ada
            if (stage._kknMapInstance) {
                try { stage._kknMapInstance.remove(); } catch(e){}
            }

            // Inisialisasi Leaflet Map
            leafletMap = L.map(stage, {
                center: [-7.915, 111.020],
                zoom: 10,
                minZoom: 9,
                maxZoom: 19,
                zoomControl: true,
                scrollWheelZoom: true
            });
            stage._kknMapInstance = leafletMap;

            // Pasang Tile Layer Awal (Satelit Asli Hybrid)
            function setMapLayer(idx) {
                currentLayerIdx = idx;
                const cfg = layersConfig[idx];
                if (currentTileLayer) {
                    leafletMap.removeLayer(currentTileLayer);
                }
                currentTileLayer = L.tileLayer(cfg.url, {
                    subdomains: cfg.subdomains,
                    maxZoom: cfg.maxZoom,
                    attribution: cfg.attribution
                }).addTo(leafletMap);

                const modeBtn = header.querySelector(".kkn-map-mode-btn");
                if (modeBtn) {
                    modeBtn.innerHTML = cfg.label;
                    modeBtn.title = cfg.title + " (Klik untuk ganti mode)";
                }
            }

            setMapLayer(0); // Default: Satelit Asli Hybrid

            // Kalibrasi ukuran peta setelah render
            setTimeout(() => { if (leafletMap) leafletMap.invalidateSize(); }, 250);
            setTimeout(() => { if (leafletMap) leafletMap.invalidateSize(); }, 750);
            setTimeout(() => {
  if (map) { map.invalidateSize(); }
}, 500);

            // Render PIN KKN Interaktif (Desain Presisi & Estetik)
            Object.entries(mapKecGroups).forEach(([key, group]) => {
                const hasProses = group.items.some(i => i.status === "Proses");
                const hasSelesai = group.items.some(i => i.status === "Selesai");
                const statusCls = hasProses ? "proses" : (hasSelesai ? "selesai" : "usulan");

                const ptnList = Array.from(new Set(group.items.map(i => i.ptn)));
                let ptnLabel = ptnList.join(", ");
                if (ptnLabel.length > 14) ptnLabel = ptnList[0] + (ptnList.length > 1 ? ` +${ptnList.length - 1}` : "");

                let alignCls = "";
                if (group.pos.lat < -8.02) alignCls += " popover-align-top";
                if (group.pos.lng > 111.18) alignCls += " popover-align-right";
                else if (group.pos.lng < 110.82) alignCls += " popover-align-left";

                let itemsHtml = "";
                group.items.forEach(item => {
                    const stCls = item.status === "Proses" ? "proses" : (item.status === "Selesai" ? "selesai" : "usulan");
                    
                    // Ekstrak lokasi desa yang spesifik untuk kecamatan ini jika formatnya per kecamatan
                    let displayLokasi = item.lokasi || "";
                    if (displayLokasi && displayLokasi.toLowerCase().includes(group.kecName.toLowerCase())) {
                        const chunks = displayLokasi.split(/[,;\r\n]+/);
                        const matched = chunks.filter(c => c.toLowerCase().includes(group.kecName.toLowerCase()));
                        if (matched.length > 0) {
                            displayLokasi = matched.map(m => m.replace(new RegExp(`\\b${group.kecName}\\b[\\s:(-]*`, 'i'), '').replace(/[)]$/, '').trim()).join(", ") || matched.join(", ");
                        }
                    }

                    itemsHtml += `
                        <div class="kkn-popover-item">
                            <div class="kkn-popover-item-head">
                                <span class="kkn-popover-ptn">🏛️ ${item.ptn}</span>
                                <span class="kkn-popover-badge ${stCls}">${item.status}</span>
                            </div>
                            <div class="kkn-popover-meta">
                                <div class="kkn-meta-row">
                                    <span class="kkn-meta-label">Kode:</span>
                                    <span class="kkn-meta-val highlight">${item.kode || '-'}</span>
                                </div>
                                ${displayLokasi ? `
                                <div class="kkn-meta-row">
                                    <span class="kkn-meta-label">Desa:</span>
                                    <span class="kkn-meta-val">${displayLokasi}</span>
                                </div>` : ''}
                                <div class="kkn-meta-row">
                                    <span class="kkn-meta-label">PIC:</span>
                                    <span class="kkn-meta-val">${item.pic || '-'}</span>
                                </div>
                                <div class="kkn-meta-row">
                                    <span class="kkn-meta-label">Kontak:</span>
                                    <span class="kkn-meta-val">${item.kontak || '-'}</span>
                                </div>
                                ${(item.tglBuka || item.tglTutup) ? `
                                <div class="kkn-meta-row">
                                    <span class="kkn-meta-label">Periode:</span>
                                    <span class="kkn-meta-val" style="font-size:9.5px;color:var(--text-muted);">${item.tglBuka || '?'} s/d ${item.tglTutup || '?'}</span>
                                </div>` : ''}
                            </div>
                        </div>
                    `;
                });

                const pinWrapperHtml = `
                    <div class="kkn-pin-wrapper ${statusCls}">
                        <!-- Titik Koordinat Presisi di Tanah -->
                        <div class="kkn-pin-ground">
                            <span class="kkn-pin-ground-pulse"></span>
                            <span class="kkn-pin-ground-core"></span>
                        </div>
                        
                        <!-- Floating Glass Pill Marker -->
                        <div class="kkn-pin-pill">
                            <span class="kkn-pill-dot"></span>
                            <span class="kkn-pill-title">${ptnLabel}</span>
                            ${group.items.length > 1 ? `<span class="kkn-pill-count">${group.items.length}</span>` : ""}
                        </div>

                 <!-- Popover Kartu Identitas (Sudah Diberi Style Inline) -->
<div class="kkn-pin-popover ${alignCls}" style="background-color: #1e1e2e !important; color: #ffffff !important; border: 1px solid #45475a !important; padding: 10px; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.5);">
    <div style="border-bottom:1px solid rgba(255,255,255,0.15);padding-bottom:5px;margin-bottom:8px;display:flex;justify-content:space-between;align-items:center;">
        <span style="font-size:11px;font-weight:700;color:#60a5fa;">📍 KEC. ${group.kecName.toUpperCase()}</span>
        <span style="font-size:9.5px;color:#a6adc8;">${group.items.length} Kegiatan KKN</span>
    </div>
    ${itemsHtml}
    <div class="kkn-popover-hint" style="color:#b4befe !important; font-size:10px; margin-top:6px;">💡 Klik PIN untuk menyorot baris di tabel</div>
</div>
                `;

                const pinIcon = L.divIcon({
                    className: 'kkn-leaflet-marker-clean',
                    html: pinWrapperHtml,
                    iconSize: [0, 0],
                    iconAnchor: [0, 0]
                });

                const marker = L.marker([group.pos.lat, group.pos.lng], { icon: pinIcon, zIndexOffset: 500 }).addTo(leafletMap);

                marker.on("mouseover", () => marker.setZIndexOffset(10000));
                marker.on("mouseout", () => marker.setZIndexOffset(500));

                marker.on("click", () => {
                    const rows = Array.from(document.querySelectorAll(".body-row"));
                    const matchingRow = rows.find(r => {
                        const cellId = r.querySelector(".cell-id");
                        return group.items.some(i => i.kode && cellId && cellId.innerText.trim() === i.kode.trim());
                    });
                    if (matchingRow) {
                        matchingRow.scrollIntoView({ behavior: "smooth", block: "center" });
                        matchingRow.classList.add("row-highlight-pulse");
                        setTimeout(() => matchingRow.classList.remove("row-highlight-pulse"), 2500);
                    }
                });
            });

            // Action: Ganti Mode Peta (Layer Cycle)
            const modeBtn = header.querySelector(".kkn-map-mode-btn");
            if (modeBtn) {
                modeBtn.addEventListener("click", () => {
                    const nextIdx = (currentLayerIdx + 1) % layersConfig.length;
                    setMapLayer(nextIdx);
                });
            }

            // Action: Fokus Waduk Gajah Mungkur
            const wadukBtn = header.querySelector(".kkn-map-waduk-btn");
            if (wadukBtn) {
                wadukBtn.addEventListener("click", () => {
                    leafletMap.flyTo([-7.892, 110.916], 13, { duration: 1.5 });
                });
            }

            // Action: Reset Peta ke Seluruh Wonogiri
            const resetBtn = header.querySelector(".kkn-map-reset-btn");
            if (resetBtn) {
                resetBtn.addEventListener("click", () => {
                    leafletMap.flyTo([-7.915, 111.020], 10, { duration: 1.2 });
                });
            }

        } catch (err) {
            console.error("Leaflet initialization error:", err);
            stage.innerHTML = `
                <div style="padding:20px;text-align:center;color:var(--text-muted);font-size:12px;">
                    <p style="color:#ef4444;font-weight:600;">⚠️ Gagal memuat peta satelit interaktif.</p>
                    <p>Pastikan perangkat terhubung ke internet untuk memuat citra satelit Google & Leaflet.</p>
                </div>
            `;
        }
    })();

    // 5. Tombol Toggle Sembunyikan / Tampilkan Peta
    const toggleBtn = header.querySelector(".kkn-map-toggle-btn");
    toggleBtn.addEventListener("click", () => {
        const isCollapsed = stage.classList.toggle("collapsed");
        toggleBtn.innerHTML = isCollapsed ? "▼ Tampilkan Peta" : "▲ Sembunyikan Peta";
        if (!isCollapsed && leafletMap) {
            setTimeout(() => leafletMap.invalidateSize(), 200);
        }
    });

    return card;
}

// Container Utama Dashboard Grid
// Container Utama Dashboard Grid
const mainContainer = dv.el("div", "", { cls: "glass-grid-container" });
mainContainer.style.cssText = "background-color: #1e1e2e !important; color: #f0f0f0 !important; padding: 15px; border-radius: 12px; border: 1px solid #313244;";

// Tambahkan Widget Peta Interaktif di atas tabel
const mapDashboardCard = createMapDashboard(pages);
mainContainer.appendChild(mapDashboardCard);

// Jalankan pembentukan Grid Tabel
const table = document.createElement("div");
table.className = "flex-grid-table";

// --- HEADER GRID ---
const headerRow = document.createElement("div");
headerRow.className = "grid-row header-row";
const headers = ["Kode", "PTN", "Status", "PIC", "No. HP", "Kecamatan", "Usulan", "Buka", "Tutup", "Tinjauan", "Lokasi"];
headers.forEach(h => {
    const th = document.createElement("div");
    const sanitizeClass = "th-" + h.toLowerCase().replace(/[^a-z0-9]/g, '');
    th.className = `grid-cell th-cell ${sanitizeClass}`;
    th.innerText = h;
    headerRow.appendChild(th);
});
table.appendChild(headerRow);

// --- BODY GRID ---
pages.forEach(p => {
    const row = document.createElement("div");
    row.className = "grid-row body-row";

    // 1. Sel ID (Kode)
    const cellKode = document.createElement("div");
    cellKode.className = "grid-cell td-cell cell-id";
    cellKode.innerText = parseYamlString(p.kode) || "-";
    row.appendChild(cellKode);

    // 2. Sel PTN
    const cellPTN = document.createElement("div");
    cellPTN.className = "grid-cell td-cell cell-ptn";
    cellPTN.innerText = parseYamlString(p.ptn) || "-";
    row.appendChild(cellPTN);

    // 3. Sel Dropdown Status
    const cellStatus = document.createElement("div");
    cellStatus.className = "grid-cell td-cell cell-status";
    const statusSelect = document.createElement("select");
    statusSelect.className = "glass-select-inline";
    const currentStatus = parseYamlString(p.status);
    setSelectStyle(statusSelect, currentStatus);
    
    statusOptions.forEach(opt => {
        const option = new Option(opt, opt);
        if (currentStatus === opt) option.selected = true;
        statusSelect.add(option);
    });
    statusSelect.addEventListener("change", async (e) => {
        await updateFrontmatter(p.file.path, "status", e.target.value);
        setSelectStyle(statusSelect, e.target.value);
    });
    cellStatus.appendChild(statusSelect);
    row.appendChild(cellStatus);

    // 4. Sel PIC
    const cellPic = document.createElement("div");
    cellPic.className = "grid-cell td-cell cell-pic";
    cellPic.innerText = parseYamlString(p.pic) || "-";
    row.appendChild(cellPic);

    // 5. Sel No HP
    const cellHP = document.createElement("div");
    cellHP.className = "grid-cell td-cell cell-hp";
    cellHP.innerText = parseYamlString(p.kontak) || "-";
    row.appendChild(cellHP);

    // 6. Sel Multi-Select Kecamatan
    const cellKec = document.createElement("div");
    cellKec.className = "grid-cell td-cell cell-kecamatan";
    
    const capsuleContainer = document.createElement("div");
    capsuleContainer.className = "inline-capsule-container";
    const badgeRow = document.createElement("div");
    badgeRow.className = "badge-row-flow";
    
    let activeKec = parseYamlArray(p.kecamatan);
    
    const displayCapsules = () => {
        badgeRow.innerHTML = "";
        activeKec.forEach(kec => {
            if (!kec) return;
            const capsule = document.createElement("span");
            const sanitizeClass = "kec-" + kec.toLowerCase().replace(/\s+/g, '');
            capsule.className = `seamless-kec-capsule ${sanitizeClass}`;
            capsule.innerHTML = `${kec} <span class="capsule-remove">×</span>`;
            capsule.querySelector(".capsule-remove").addEventListener("click", async (e) => {
                e.stopPropagation();
                activeKec = activeKec.filter(k => k !== kec);
                displayCapsules();
                await updateFrontmatter(p.file.path, "kecamatan", activeKec);
            });
            badgeRow.appendChild(capsule);
        });
    };
    
    const addBtn = document.createElement("button");
    addBtn.className = "add-capsule-btn";
    addBtn.innerText = "+";
    
    const popupMenu = document.createElement("div");
    popupMenu.className = "capsule-popup-menu hidden";
    const searchInput = document.createElement("input");
    searchInput.type = "text";
    searchInput.className = "popup-search-input";
    searchInput.placeholder = "Cari...";
    popupMenu.appendChild(searchInput);
    
    const listContainer = document.createElement("div");
    listContainer.className = "popup-list-container";
    popupMenu.appendChild(listContainer);
    
    kecamatanOptions.forEach(opt => {
        const label = document.createElement("label");
        label.className = "popup-checkbox-label";
        label.dataset.name = opt.toLowerCase();
        const checkbox = document.createElement("input");
        checkbox.type = "checkbox";
        checkbox.value = opt;
        checkbox.checked = activeKec.includes(opt);
        checkbox.addEventListener("change", async () => {
            if (checkbox.checked) { if (!activeKec.includes(opt)) activeKec.push(opt); }
            else { activeKec = activeKec.filter(k => k !== opt); }
            displayCapsules();
            await updateFrontmatter(p.file.path, "kecamatan", activeKec);
        });
        label.appendChild(checkbox);
        label.appendChild(document.createTextNode(" " + opt));
        listContainer.appendChild(label);
    });

    searchInput.addEventListener("input", (e) => {
        const query = e.target.value.toLowerCase().trim();
        listContainer.querySelectorAll(".popup-checkbox-label").forEach(lbl => {
            if (lbl.dataset.name.includes(query)) lbl.classList.remove("hidden-item");
            else lbl.classList.add("hidden-item");
        });
    });

    addBtn.addEventListener("click", (e) => {
        e.stopPropagation();
        const isOpen = !popupMenu.classList.contains("hidden");
        document.querySelectorAll(".capsule-popup-menu").forEach(menu => menu.classList.add("hidden"));
        if (!isOpen) {
            document.body.appendChild(popupMenu);
            popupMenu.classList.remove("hidden");
            const rect = addBtn.getBoundingClientRect();
            popupMenu.style.top = `${rect.bottom + window.scrollY + 4}px`;
            popupMenu.style.left = `${rect.left + window.scrollX - 100}px`;
            setTimeout(() => searchInput.focus(), 50);
        }
    });

    displayCapsules();
    capsuleContainer.appendChild(badgeRow);
    capsuleContainer.appendChild(addBtn);
    cellKec.appendChild(capsuleContainer);
    row.appendChild(cellKec);

    // 7. Tanggal Usulan
    const cellUsulan = document.createElement("div");
    cellUsulan.className = "grid-cell td-cell cell-date";
    const inUsulan = document.createElement("input");
    inUsulan.type = "date";
    inUsulan.className = "glass-date-input";
    inUsulan.value = formatIsoDate(p.tgl_usulan);
    inUsulan.addEventListener("change", async (e) => {
        if(!e.target.value) return;
        await updateFrontmatter(p.file.path, "tgl_usulan", e.target.value);
    });
    cellUsulan.appendChild(inUsulan);
    row.appendChild(cellUsulan);

    // 8. Tanggal Pembukaan
    const cellBuka = document.createElement("div");
    cellBuka.className = "grid-cell td-cell cell-date";
    const inBuka = document.createElement("input");
    inBuka.type = "date";
    inBuka.className = "glass-date-input";
    inBuka.value = formatIsoDate(p.tgl_pembukaan);
    inBuka.addEventListener("change", async (e) => {
        if(!e.target.value) return;
        await updateFrontmatter(p.file.path, "tgl_pembukaan", e.target.value);
    });
    cellBuka.appendChild(inBuka);
    row.appendChild(cellBuka);

    // 9. Tanggal Penutupan
    const cellTutup = document.createElement("div");
    cellTutup.className = "grid-cell td-cell cell-date";
    const inTutup = document.createElement("input");
    inTutup.type = "date";
    inTutup.className = "glass-date-input";
    inTutup.value = formatIsoDate(p.tgl_penutupan);
    inTutup.addEventListener("change", async (e) => {
        if(!e.target.value) return;
        await updateFrontmatter(p.file.path, "tgl_penutupan", e.target.value);
    });
    cellTutup.appendChild(inTutup);
    row.appendChild(cellTutup);

    // 10. Sel Tinjauan
    const cellTinjauan = document.createElement("div");
    cellTinjauan.className = "grid-cell td-cell cell-tinjauan";
    const inTinjauan = document.createElement("input");
    inTinjauan.type = "text";
    inTinjauan.className = "glass-text-input-inline";
    inTinjauan.value = parseYamlString(p.tinjauan);
    inTinjauan.placeholder = "...";
    inTinjauan.addEventListener("change", async (e) => {
        await updateFrontmatter(p.file.path, "tinjauan", e.target.value);
    });
    cellTinjauan.appendChild(inTinjauan);
    row.appendChild(cellTinjauan);

    // 11. Lokasi Desa
    const cellLokasi = document.createElement("div");
    cellLokasi.className = "grid-cell td-cell cell-lokasi";
    cellLokasi.innerText = parseYamlString(p.lokasi) || "-";
    row.appendChild(cellLokasi);


    table.appendChild(row);
});

mainContainer.appendChild(table);

// Global Click Popup Cleaner
document.addEventListener("click", () => {
    document.querySelectorAll(".capsule-popup-menu").forEach(menu => menu.classList.add("hidden"));
});

// Frontmatter Processor Engine
async function updateFrontmatter(filePath, key, value) {
    try {
        const file = app.vault.getAbstractFileByPath(filePath);
        if (file) {
            await app.fileManager.processFrontMatter(file, (frontmatter) => {
                frontmatter[key] = value;
            });
            new Notice(`✅ Grid Terupdate!`);
        }
    } catch (err) {
        console.error(err);
    }
}

function setSelectStyle(el, status) {
    el.classList.remove("inline-status-proses", "inline-status-selesai", "inline-status-belum");
    if (status === "Proses") el.classList.add("inline-status-proses");
    else if (status === "Selesai") el.classList.add("inline-status-selesai");
    else el.classList.add("inline-status-belum");
}


```