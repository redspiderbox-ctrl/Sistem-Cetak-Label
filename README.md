<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Cetak Label (POLYTRON-KUPANG)</title>
    
    <!-- Pustaka Eksternal -->
    <script src="https://cdn.sheetjs.com/xlsx-0.19.3/package/dist/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

    <style>
        :root {
            --primary: #2563eb;
            --primary-hover: #1d4ed8;
            --danger: #ef4444;
            --bg-desk: #cbd5e1;
            --paper: #ffffff;
            --text-main: #1e293b;
            --text-muted: #64748b;
            --border: #94a3b8;
            
            /* Konfigurasi A4 Landscape */
            --label-h: 14mm;
            --qty-side-w: 14mm; /* Sedikit lebih lebar untuk menyeimbangkan font besar */
            --gap-x: 3mm; 
            --gap-y: 4mm;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Roboto, Helvetica, sans-serif; }

        body { 
            background-color: var(--bg-desk); 
            color: var(--text-main); 
            display: flex; 
            justify-content: center; 
            min-height: 100vh; 
            padding: 20px;
        }

        /* === KERTAS A4 LANDSCAPE === */
        .a4-paper {
            width: 297mm; 
            height: 210mm; 
            background: var(--paper);
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
            padding: 10mm; 
            position: relative;
            display: flex;
            flex-direction: column;
            max-height: 95vh; 
            overflow-y: auto;
        }

        @media (max-width: 1200px) {
            .a4-paper { width: 100%; height: auto; min-height: 210mm; padding: 5mm; }
            body { padding: 0; }
        }

        /* === HEADER & TABS === */
        header { margin-bottom: 15px; border-bottom: 2px solid var(--bg-desk); padding-bottom: 10px; }
        h1 { font-size: 1.2rem; color: var(--text-main); }
        .subtitle { font-size: 0.85rem; color: var(--text-muted); }

        .tabs { display: flex; gap: 5px; margin-bottom: 15px; }
        .tab-btn {
            flex: 1; padding: 8px; background: #f1f5f9;
            border: 1px solid var(--border); border-bottom: none;
            border-radius: 6px 6px 0 0;
            cursor: pointer; font-weight: 600; color: var(--text-muted);
            font-size: 0.9rem; transition: all 0.2s;
        }
        .tab-btn.active { background: white; color: var(--primary); border-top: 3px solid var(--primary); margin-bottom: -1px; }

        /* === KONTROL & UPLOAD === */
        .control-panel {
            background: #f8fafc; border: 1px solid var(--border);
            padding: 10px; border-radius: 6px; margin-bottom: 15px;
            display: flex; gap: 10px; align-items: center; flex-wrap: wrap;
        }

        .upload-box { position: relative; overflow: hidden; display: inline-block; }
        .upload-btn {
            border: 2px dashed var(--border); background: white;
            color: var(--text-main); padding: 6px 12px;
            border-radius: 4px; font-size: 0.9rem; font-weight: 500;
            cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 6px;
        }
        .upload-btn:hover { border-color: var(--primary); background: #eff6ff; color: var(--primary); }
        .upload-box input[type=file] {
            font-size: 100px; position: absolute; left: 0; top: 0; opacity: 0; cursor: pointer;
        }

        .action-btn {
            padding: 6px 16px; border: none; border-radius: 4px; cursor: pointer; font-weight: 600; font-size: 0.85rem;
            display: flex; align-items: center; gap: 6px; transition: 0.2s;
        }
        .btn-primary { background: var(--primary); color: white; }
        .btn-primary:hover { background: var(--primary-hover); }
        .btn-primary:disabled { background: var(--text-muted); cursor: not-allowed; opacity: 0.7; }
        .btn-danger { background: white; border: 1px solid var(--border); color: var(--danger); }
        .btn-danger:hover { background: #fef2f2; border-color: var(--danger); }

        /* === GRID SYSTEM === */
        .labels-grid-fixed {
            display: grid;
            grid-template-columns: repeat(6, 1fr); 
            gap: var(--gap-y) var(--gap-x); 
            align-content: start;
        }

        .labels-grid-flex {
            display: flex;
            flex-wrap: wrap;
            gap: var(--gap-y) var(--gap-x);
            align-content: flex-start;
        }

        /* === LABEL CARD STYLES === */
        .label-card {
            background: white;
            border: 1px solid black; 
            border-radius: 2px;
            height: var(--label-h); 
            display: flex; 
            position: relative;
            overflow: hidden;
            font-size: 10px;
            box-shadow: 0 1px 2px rgba(0,0,0,0.05);
        }

        /* TAB 2: Flex Mode */
        .label-card.flex-mode {
            width: auto; 
            min-width: 50mm; /* Lebar minimum dinaikkan karena font besar */
            max-width: 150mm; 
            flex-shrink: 0;
        }

        /* TAB 1: Fixed Mode */
        .label-card.fixed-mode {
            width: 100%;
        }

        .label-main {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            padding: 2px;
            border-right: 1px solid #000;
            justify-content: center; /* Vertically center content */
            min-width: 20px;
        }

        .label-top {
            border-bottom: 1px solid #ccc;
            height: 50%; /* Bagian atas lebih besar untuk font besar */
            margin-bottom: 1px;
            display: flex;
            align-items: center;
            justify-content: center; /* Center Teks Part */
            width: 100%;
        }

        /* TAB 1 Override (Karena Tab 1 ukuran normal) */
        .labels-grid-fixed .label-top { height: 40%; }

        .part-number {
            font-weight: 900; 
            line-height: 1;
            color: #000;
            white-space: nowrap; 
            overflow: hidden; 
            text-overflow: ellipsis;
            width: 100%;
            text-align: center;
        }

        /* Override Font Size Tab 2 */
        .labels-grid-flex .part-number {
            font-size: 1.4rem; /* Font Sangat Besar */
        }
        
        /* Deskripsi Area */
        .desc-area {
            height: 50%; /* Sisa tinggi */
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            overflow: hidden;
            padding: 0 2px;
        }
        
        /* Override Deskripsi Tab 1 */
        .labels-grid-fixed .desc-area {
            height: 60%;
            align-items: flex-start;
            justify-content: center;
        }

        .desc-text {
            font-size: 0.7rem; 
            color: #333; 
            line-height: 1.1;
            width: 100%;
            word-wrap: break-word; /* Agar text turun ke bawah jika panjang */
        }
        
        .labels-grid-fixed .desc-text {
             white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
        }

        .qty-side {
            width: var(--qty-side-w);
            background: #f1f5f9; 
            border-left: 1px solid #000; 
            display: flex;
            justify-content: center;
            align-items: center; 
            flex-shrink: 0;
        }

        .qty-text {
            font-weight: 900;
            font-size: 1.2rem; /* Sesuaikan dengan part number */
            color: #000;
            line-height: 1;
            text-align: center;
        }

        .empty-msg {
            grid-column: 1 / -1;
            width: 100%;
            text-align: center; padding: 40px;
            color: var(--text-muted); font-style: italic; border: 2px dashed #e2e8f0;
        }

        /* Toast */
        #toast-container { position: fixed; bottom: 20px; right: 20px; z-index: 9999; display: flex; flex-direction: column; gap: 10px; }
        .toast { padding: 10px 16px; border-radius: 4px; color: white; box-shadow: 0 4px 6px rgba(0,0,0,0.1); font-size: 0.9rem; animation: slideIn 0.3s; }
        .toast.success { background-color: #10b981; }
        .toast.error { background-color: #ef4444; }
        .toast.info { background-color: #3b82f6; }
        @keyframes slideIn { from {transform: translateX(100%); opacity: 0;} to {transform: translateX(0); opacity: 1;} }

        .hidden { display: none !important; }
        .flex-spacer { flex-grow: 1; }
        .stats { font-size: 0.8rem; color: var(--text-muted); margin-left: auto; }
    </style>
</head>
<body>

    <div class="a4-paper">
        <header>
            <h1>Sistem Cetak Label (POLYTRON-KUPANG)</h1>
            <div class="subtitle">Sistem Akan Otomatis Membuat Label Komponen Dengan Presisi</div>
        </header>

       
       
       <!-- Navigasi Tab -->
        <div class="tabs">
            <button class="tab-btn active" id="btn-tab-1" onclick="switchTab(1)">
                Tab 1: Label Part (Fixed)
            </button>
            <button class="tab-btn" id="btn-tab-2" onclick="switchTab(2)">
                Tab 2: Label Stock (Flexible & Big)
            </button>
        </div>

        <!-- TAB 1 -->
        <main id="view-tab-1">
            <section class="control-panel">
                <div class="upload-box">
                    <button class="upload-btn">&#128194; Upload Excel (Tab 1)</button>
                    <input type="file" id="input-file-1" accept=".xlsx, .xls">
                </div>
                <div class="flex-spacer"></div>
                <div class="stats" id="stats-1">0 Label</div>
                <button class="action-btn btn-danger hidden" id="btn-reset-1" onclick="resetData(1)">&#10005; Reset</button>
                <button class="action-btn btn-primary" id="btn-download-1" disabled onclick="generatePDF(1)">&#128196; Download PDF</button>
            </section>

            <div class="labels-grid-fixed" id="grid-1">
                <div class="empty-msg">Upload Excel untuk Tab 1.</div>
            </div>
        </main>

        <!-- TAB 2 -->
        <main id="view-tab-2" class="hidden">
            <section class="control-panel">
                <div class="upload-box">
                    <button class="upload-btn">&#128194; Upload Excel (Tab 2)</button>
                    <input type="file" id="input-file-2" accept=".xlsx, .xls">
                </div>
                <div class="flex-spacer"></div>
                <div class="stats" id="stats-2">0 Label</div>
                <button class="action-btn btn-danger hidden" id="btn-reset-2" onclick="resetData(2)">&#10005; Reset</button>
                <button class="action-btn btn-primary" id="btn-download-2" disabled onclick="generatePDF(2)">&#128196; Download PDF</button>
            </section>

            <div class="labels-grid-flex" id="grid-2">
                <div class="empty-msg">Upload Excel untuk Tab 2.</div>
            </div>
        </main>

    </div>

    <div id="toast-container"></div>

<script>
    const state = {
        1: { raw: [], processed: [] }, 
        2: { raw: [], processed: [] }  
    };

    // KONFIGURASI PDF LANDSCAPE
    const CFG = {
        pageW: 297, 
        pageH: 210, 
        margin: 10,
        labelH: 14,
        gapX: 3, 
        gapY: 4,
        tab1_cols: 6, 
        qtySideW: 14, // Lebar kotak qty (mm)
        tab2_minW: 50, // Lebar minimum (mm) - disesuaikan dengan font besar
        tab2_maxW: 150 // Lebar maksimum (mm)
    };

    document.getElementById('input-file-1').addEventListener('change', (e) => handleFile(e, 1));
    document.getElementById('input-file-2').addEventListener('change', (e) => handleFile(e, 2));

    function switchTab(tabId) {
        document.getElementById('view-tab-1').classList.add('hidden');
        document.getElementById('view-tab-2').classList.add('hidden');
        document.getElementById('btn-tab-1').classList.remove('active');
        document.getElementById('btn-tab-2').classList.remove('active');

        document.getElementById(`view-tab-${tabId}`).classList.remove('hidden');
        document.getElementById(`btn-tab-${tabId}`).classList.add('active');
    }

    function showToast(msg, type = 'info') {
        const container = document.getElementById('toast-container');
        const toast = document.createElement('div');
        toast.className = `toast ${type}`;
        toast.innerText = msg;
        container.appendChild(toast);
        setTimeout(() => {
            toast.style.opacity = '0';
            setTimeout(() => toast.remove(), 300);
        }, 3000);
    }

    function handleFile(event, tabId) {
        const file = event.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = (e) => {
            try {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, { type: 'array' });
                const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
                const jsonData = XLSX.utils.sheet_to_json(firstSheet, { defval: "" });

                if (jsonData.length === 0) throw new Error("File Excel kosong.");

                processData(jsonData, tabId);
                showToast(`Data Tab ${tabId} dimuat.`, 'success');
                event.target.value = ''; 
            } catch (err) {
                console.error(err);
                showToast("Gagal membaca file Excel.", 'error');
            }
        };
        reader.readAsArrayBuffer(file);
    }

    function processData(json, tabId) {
        state[tabId].raw = json;
        state[tabId].processed = [];

        json.forEach((row, index) => {
            const findVal = (keywords) => {
                const key = Object.keys(row).find(k => keywords.includes(k.toLowerCase().trim()));
                return key ? String(row[key]).trim() : "";
            };

            const item = {
                part: findVal(['part', 'part number', 'part no', 'kode', 'code', 'id', 'item']) || 'N/A',
                desc: findVal(['deskripsi', 'description', 'desc', 'nama', 'name']) || '-',
                qty: parseInt(findVal(['qty', 'quantity', 'jumlah', 'stock', 'qty'])) || 1
            };

            if (tabId === 1) {
                for (let i = 0; i < item.qty; i++) {
                    state[tabId].processed.push(item);
                }
            } else {
                state[tabId].processed.push(item);
            }
        });

        renderGrid(tabId);
    }

    function resetData(tabId) {
        state[tabId] = { raw: [], processed: [] };
        document.getElementById(`input-file-${tabId}`).value = '';
        renderGrid(tabId);
        showToast(`Data Tab ${tabId} direset.`, 'info');
    }

    function renderGrid(tabId) {
        const container = document.getElementById(`grid-${tabId}`);
        const btnDownload = document.getElementById(`btn-download-${tabId}`);
        const btnReset = document.getElementById(`btn-reset-${tabId}`);
        const stats = document.getElementById(`stats-${tabId}`);
        const data = state[tabId].processed;

        container.innerHTML = '';
        stats.innerText = `${data.length} Label`;

        if (data.length === 0) {
            container.innerHTML = `<div class="empty-msg">Data kosong di Tab ${tabId}.</div>`;
            btnDownload.disabled = true;
            btnReset.classList.add('hidden');
            return;
        }

        btnDownload.disabled = false;
        btnReset.classList.remove('hidden');

        const previewLimit = 100;
        const dataToRender = data.slice(0, previewLimit);

        dataToRender.forEach(item => {
            const card = document.createElement('div');
            
            if (tabId === 2) {
                card.className = 'label-card flex-mode';
                // Struktur HTML Baru untuk Posisi Tengah & Flex
                card.innerHTML = `
                    <div class="label-main">
                        <div class="label-top">
                            <span class="part-number" title="${item.part}">${item.part}</span>
                        </div>
                        <div class="desc-area">
                            <span class="desc-text" title="${item.desc}">${item.desc}</span>
                        </div>
                    </div>
                    <div class="qty-side">
                        <span class="qty-text">${item.qty}</span>
                    </div>
                `;
            } else {
                card.className = 'label-card fixed-mode';
                card.innerHTML = `
                    <div class="label-content" style="width:100%; display:flex; flex-direction:column; padding:2px 4px;">
                        <div class="label-top" style="border-bottom:1px solid #ccc; height:40%; margin-bottom:2px; display:flex; align-items:center; justify-content:center;">
                            <span class="part-number" title="${item.part}">${item.part}</span>
                        </div>
                        <div class="desc-area" style="height:60%; display:flex; align-items:center; justify-content:center;">
                            <span class="desc-text" title="${item.desc}">${item.desc}</span>
                        </div>
                    </div>
                `;
            }
            container.appendChild(card);
        });

        if (data.length > previewLimit) {
            const info = document.createElement('div');
            info.style.width = "100%"; 
            info.style.gridColumn = "1 / -1";
            info.style.textAlign = "center";
            info.style.padding = "10px";
            info.style.color = "#64748b";
            info.style.fontSize = "0.8rem";
            info.innerHTML = `...dan ${data.length - previewLimit} label lainnya (preview terbatas).`;
            container.appendChild(info);
        }
    }

    // --- PDF GENERATION ---
    async function generatePDF(tabId) {
        const data = state[tabId].processed;
        if (!data || data.length === 0) return;

        const { jsPDF } = window.jspdf;
        const doc = new jsPDF('l', 'mm', 'a4');
        
        doc.setFont("helvetica");
        
        showToast("Sedang membuat PDF...", "info");

        setTimeout(() => {
            if (tabId === 1) {
                generateTab1PDF(doc, data);
            } else {
                generateTab2PDF(doc, data);
            }
            doc.save(`Label_Tab${tabId}_${new Date().getTime()}.pdf`);
            showToast("PDF berhasil diunduh!", "success");

        }, 100);
    }

    function generateTab1PDF(doc, data) {
        let cursor = { x: CFG.margin, y: CFG.margin };
        const colWidth = (CFG.pageW - (CFG.margin * 2) - (CFG.gapX * (CFG.tab1_cols - 1))) / CFG.tab1_cols;
        let colCount = 0;

        data.forEach((item) => {
            if (cursor.y + CFG.labelH > CFG.pageH - CFG.margin) {
                doc.addPage();
                cursor.y = CFG.margin;
                cursor.x = CFG.margin;
                colCount = 0;
            }

            if (colCount >= CFG.tab1_cols) {
                cursor.y += CFG.labelH + CFG.gapY;
                cursor.x = CFG.margin;
                colCount = 0;
                if (cursor.y + CFG.labelH > CFG.pageH - CFG.margin) {
                    doc.addPage();
                    cursor.y = CFG.margin;
                    cursor.x = CFG.margin;
                    colCount = 0;
                }
            }

            doc.setDrawColor(0); doc.setLineWidth(0.2);
            doc.rect(cursor.x, cursor.y, colWidth, CFG.labelH);

            // Part
            doc.setFont("helvetica", "bold"); doc.setFontSize(9); doc.setTextColor(0,0,0);
            const centerX = cursor.x + (colWidth / 2);
            doc.text(item.part, centerX, cursor.y + 4, { align: 'center', maxWidth: colWidth - 4 });

            doc.line(cursor.x, cursor.y + 5.5, cursor.x + colWidth, cursor.y + 5.5);

            // Desc
            doc.setFont("helvetica", "normal"); doc.setFontSize(6); doc.setTextColor(60,60,60);
            const splitDesc = doc.splitTextToSize(item.desc, colWidth - 4);
            doc.text(splitDesc, centerX, cursor.y + 9, { align: 'center' });

            cursor.x += colWidth + CFG.gapX;
            colCount++;
        });
    }

    function generateTab2PDF(doc, data) {
        // LOGIKA FLEXIBLE PDF (TAB 2) - TENGAH & FONTS BESAR
        let cursor = { x: CFG.margin, y: CFG.margin };

        data.forEach((item) => {
            // 1. Hitung Ukuran Font Besar
            doc.setFont("helvetica", "bold"); doc.setFontSize(14); // UKURAN FONT SANGAT BESAR
            const partWidth = doc.getTextWidth(item.part);
            
            // Padding lebih longgar untuk font besar
            const paddingX = 10; 
            let contentWidth = partWidth + paddingX;

            // Min/Max Width
            if (contentWidth < CFG.tab2_minW - CFG.qtySideW) {
                contentWidth = CFG.tab2_minW - CFG.qtySideW;
            }
            if (contentWidth > CFG.tab2_maxW - CFG.qtySideW) {
                contentWidth = CFG.tab2_maxW - CFG.qtySideW;
            }

            const totalWidth = contentWidth + CFG.qtySideW;

            // 2. Cek Posisi Baris/Halaman
            if (cursor.x + totalWidth > CFG.pageW - CFG.margin) {
                cursor.y += CFG.labelH + CFG.gapY;
                cursor.x = CFG.margin;
                
                if (cursor.y + CFG.labelH > CFG.pageH - CFG.margin) {
                    doc.addPage();
                    cursor.y = CFG.margin;
                    cursor.x = CFG.margin;
                }
            }

            // 3. GAMBAR KOTAK KIRI (KONTEN)
            doc.setDrawColor(0); doc.setLineWidth(0.2);
            doc.rect(cursor.x, cursor.y, contentWidth, CFG.labelH);

            // Garis Pemisah (Lebih rendah karena font besar butuh ruang)
            const dividerY = cursor.y + 7; 
            doc.line(cursor.x, dividerY, cursor.x + contentWidth, dividerY);

            // --- PART NUMBER (CENTER, BESAR) ---
            doc.setTextColor(0,0,0);
            const centerX = cursor.x + (contentWidth / 2);
            // Turunkan sedikit Y agar visual terlihat tengah di ruang atas
            doc.text(item.part, centerX, cursor.y + 5, { align: 'center', maxWidth: contentWidth - 2 });

            // --- DESKRIPSI (CENTER, FLEKSIBLE) ---
            doc.setFont("helvetica", "normal"); doc.setFontSize(5); // Font kecil agar muat banyak kalimat
            doc.setTextColor(60,60,60);
            const splitDesc = doc.splitTextToSize(item.desc, contentWidth - 4); // MaxWidth sesuai kotak
            // Hitung posisi tengah untuk deskripsi (Ruang bawah)
            // Kita letakkan sedikit di atas batas bawah
            const descY = cursor.y + 10;
            doc.text(splitDesc, centerX, descY, { align: 'center' });

            // 4. GAMBAR KOTAK KANAN (QTY)
            const qtyX = cursor.x + contentWidth;
            doc.setFillColor(241, 245, 249);
            doc.rect(qtyX, cursor.y, CFG.qtySideW, CFG.labelH, 'FD');

            // --- QTY TEXT (CENTER, BESAR) ---
            doc.setFont("helvetica", "bold"); doc.setFontSize(13); // Font besar juga
            doc.setTextColor(0,0,0);
            const qtyCenterX = qtyX + (CFG.qtySideW / 2);
            const qtyCenterY = cursor.y + (CFG.labelH / 2) + 1; // +1 untuk baseline adjustment
            doc.text(String(item.qty), qtyCenterX, qtyCenterY, { align: 'center' });

            // 5. Geser Cursor
            cursor.x += totalWidth + CFG.gapX;
        });
    }
</script>

</body>
</html>
