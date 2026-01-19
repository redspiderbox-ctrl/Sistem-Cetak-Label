<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Cetak Label (Tab 2 Besar)</title>
    
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
            --qty-side-w: 10mm; /* Default kecil (Tab 3) */
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
        h1 { font-size: 1.1rem; color: var(--text-main); }
        .subtitle { font-size: 0.8rem; color: var(--text-muted); }

        .tabs { display: flex; gap: 5px; margin-bottom: 15px; }
        .tab-btn {
            flex: 1; padding: 8px 4px; background: #f1f5f9;
            border: 1px solid var(--border); border-bottom: none;
            border-radius: 6px 6px 0 0;
            cursor: pointer; font-weight: 600; color: var(--text-muted);
            font-size: 0.85rem; transition: all 0.2s; text-align: center;
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
            border-radius: 4px; font-size: 0.85rem; font-weight: 500;
            cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 6px;
        }
        .upload-btn:hover { border-color: var(--primary); background: #eff6ff; color: var(--primary); }
        .upload-box input[type=file] {
            font-size: 100px; position: absolute; left: 0; top: 0; opacity: 0; cursor: pointer;
        }

        .action-btn {
            padding: 6px 12px; border: none; border-radius: 4px; cursor: pointer; font-weight: 600; font-size: 0.85rem;
            display: flex; align-items: center; gap: 6px; transition: 0.2s;
        }
        .btn-primary { background: var(--primary); color: white; }
        .btn-primary:hover { background: var(--primary-hover); }
        .btn-primary:disabled { background: var(--text-muted); cursor: not-allowed; opacity: 0.7; }
        .btn-danger { background: white; border: 1px solid var(--border); color: var(--danger); }
        .btn-danger:hover { background: #fef2f2; border-color: var(--danger); }

        /* === GRID SYSTEM === */
        
        /* Tab 1: Fixed Grid (Row Major) */
        .labels-grid-fixed {
            display: grid;
            grid-template-columns: repeat(6, 1fr); 
            gap: var(--gap-y) var(--gap-x); 
            align-content: start;
        }

        /* Tab 2: Flex Wrap Row (Row Major - Kiri ke Kanan) */
        .labels-grid-flex-row {
            display: flex;
            flex-wrap: wrap;
            gap: var(--gap-y) var(--gap-x);
            align-content: flex-start;
        }

        /* Tab 3: Flex Wrap Column (COLUMN MAJOR - Atas ke Bawah) */
        .labels-grid-flex-col {
            display: flex;
            flex-direction: column; 
            flex-wrap: wrap; 
            height: 190mm; 
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

        /* Default Flex (Kecil - Untuk Tab 3) */
        .label-card.flex-mode {
            width: auto; 
            min-width: 30mm; 
            max-width: 120mm; 
            flex-shrink: 0;
        }

        /* SPECIAL: FLEX LARGE (Untuk Tab 2) */
        .label-card.flex-large {
            --qty-side-w: 14mm; /* Override lebar kotak qty jadi besar */
            width: auto; 
            min-width: 50mm; /* Lebar minimum lebih besar */
            max-width: 150mm; 
            flex-shrink: 0;
        }

        /* Mode Fixed (Tab 1) */
        .label-card.fixed-mode {
            width: 100%;
        }

        .label-main {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            padding: 2px;
            border-right: 1px solid #000;
            justify-content: center; 
            min-width: 20px;
        }

        .label-top {
            border-bottom: 1px solid #ccc;
            height: 40%; 
            margin-bottom: 1px;
            display: flex;
            align-items: center;
            justify-content: center; 
            width: 100%;
        }

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

        /* Font Kecil (Tab 3) */
        .flex-mode .part-number { font-size: 0.85rem; }
        
        /* Font Besar (Tab 2) */
        .flex-large .part-number { font-size: 1.2rem; }
        
        .desc-area {
            height: 60%; 
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            overflow: hidden;
            padding: 0 2px;
        }

        .desc-text {
            font-size: 0.65rem; 
            color: #333; 
            line-height: 1.1;
            width: 100%;
            word-wrap: break-word;
        }
        
        /* Font Desc Sedikit lebih besar di Tab 2 agar seimbang */
        .flex-large .desc-text { font-size: 0.8rem; }

        .qty-side {
            width: var(--qty-side-w); /* Menggunakan variable */
            background: #f1f5f9; 
            border-left: 1px solid #000; 
            display: flex;
            justify-content: center;
            align-items: center; 
            flex-shrink: 0;
        }

        .qty-text {
            font-weight: 900;
            font-size: 0.9rem; /* Kecil default */
            color: #000;
            line-height: 1;
            text-align: center;
        }

        /* Font Qty Besar (Tab 2) */
        .flex-large .qty-text { font-size: 1.1rem; }

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
            <h1>Sistem Cetak Label (Tab 2 Besar)</h1>
            <div class="subtitle">Tab 1: Fixed &bull; Tab 2: Flex (Besar) &bull; Tab 3: Flex (Kecil & Col)</div>
        </header>

        <!-- Navigasi Tab -->
        <div class="tabs">
            <button class="tab-btn active" id="btn-tab-1" onclick="switchTab(1)">Tab 1: Fixed</button>
            <button class="tab-btn" id="btn-tab-2" onclick="switchTab(2)">Tab 2: Flex</button>
            <button class="tab-btn" id="btn-tab-3" onclick="switchTab(3)">Tab 3: Col</button>
        </div>

        <!-- TAB 1: FIXED -->
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

        <!-- TAB 2: FLEX ROW (BESAR) -->
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

            <div class="labels-grid-flex-row" id="grid-2">
                <div class="empty-msg">Upload Excel untuk Tab 2.</div>
            </div>
        </main>

        <!-- TAB 3: FLEX COLUMN (KECIL) -->
        <main id="view-tab-3" class="hidden">
            <section class="control-panel">
                <div class="upload-box">
                    <button class="upload-btn">&#128194; Upload Excel (Tab 3)</button>
                    <input type="file" id="input-file-3" accept=".xlsx, .xls">
                </div>
                <div class="flex-spacer"></div>
                <div class="stats" id="stats-3">0 Label</div>
                <button class="action-btn btn-danger hidden" id="btn-reset-3" onclick="resetData(3)">&#10005; Reset</button>
                <button class="action-btn btn-primary" id="btn-download-3" disabled onclick="generatePDF(3)">&#128196; Download PDF</button>
            </section>

            <div class="labels-grid-flex-col" id="grid-3">
                <div class="empty-msg">Upload Excel untuk Tab 3.</div>
            </div>
        </main>

    </div>

    <div id="toast-container"></div>

<script>
    const state = {
        1: { raw: [], processed: [] }, 
        2: { raw: [], processed: [] }, 
        3: { raw: [], processed: [] }  
    };

    const CFG = {
        pageW: 297, 
        pageH: 210, 
        margin: 10,
        labelH: 14,
        gapX: 3, 
        gapY: 4,
        tab1_cols: 6, 
        
        // Config Kecil (Tab 3)
        qtySideW_Small: 10, 
        tab2_minW_Small: 30, 
        tab2_maxW_Small: 120,

        // Config Besar (Tab 2)
        qtySideW_Large: 14, // Lebar kotak Qty besar
        tab2_minW_Large: 50, // Lebar minimum kotak besar
        tab2_maxW_Large: 150 
    };

    document.getElementById('input-file-1').addEventListener('change', (e) => handleFile(e, 1));
    document.getElementById('input-file-2').addEventListener('change', (e) => handleFile(e, 2));
    document.getElementById('input-file-3').addEventListener('change', (e) => handleFile(e, 3));

    function switchTab(tabId) {
        document.getElementById('view-tab-1').classList.add('hidden');
        document.getElementById('view-tab-2').classList.add('hidden');
        document.getElementById('view-tab-3').classList.add('hidden');
        
        document.getElementById('btn-tab-1').classList.remove('active');
        document.getElementById('btn-tab-2').classList.remove('active');
        document.getElementById('btn-tab-3').classList.remove('active');

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
                // TAB 2: FLEX LARGE (BESAR)
                card.className = 'label-card flex-large';
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
            } else if (tabId === 3) {
                // TAB 3: FLEX MODE SMALL (KECIL)
                card.className = 'label-card flex-mode';
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
                // TAB 1: FIXED
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
            } else if (tabId === 2) {
                // TAB 2: FUNGSI BESAR
                generateTab2LargePDF(doc, data);
            } else {
                // TAB 3: FUNGSI KECIL & COLUMN MAJOR
                generateTab3PDF(doc, data);
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

            doc.setFont("helvetica", "bold"); doc.setFontSize(9); doc.setTextColor(0,0,0);
            const centerX = cursor.x + (colWidth / 2);
            doc.text(item.part, centerX, cursor.y + 4, { align: 'center', maxWidth: colWidth - 4 });

            doc.line(cursor.x, cursor.y + 5.5, cursor.x + colWidth, cursor.y + 5.5);

            doc.setFont("helvetica", "normal"); doc.setFontSize(6); doc.setTextColor(60,60,60);
            const splitDesc = doc.splitTextToSize(item.desc, colWidth - 4);
            doc.text(splitDesc, centerX, cursor.y + 9, { align: 'center' });

            cursor.x += colWidth + CFG.gapX;
            colCount++;
        });
    }

    function generateTab2LargePDF(doc, data) {
        // FUNGSI KHUSUS TAB 2 (FONTS & KOTAK BESAR)
        let cursor = { x: CFG.margin, y: CFG.margin };
        const currentQtySideW = CFG.qtySideW_Large;

        data.forEach((item) => {
            // Font Besar
            doc.setFont("helvetica", "bold"); doc.setFontSize(14); 
            const partWidth = doc.getTextWidth(item.part);
            
            // Padding Lebar
            const paddingX = 10; 
            let contentWidth = partWidth + paddingX;

            // Min/Max Width Besar
            if (contentWidth < CFG.tab2_minW_Large - currentQtySideW) {
                contentWidth = CFG.tab2_minW_Large - currentQtySideW;
            }
            if (contentWidth > CFG.tab2_maxW_Large - currentQtySideW) {
                contentWidth = CFG.tab2_maxW_Large - currentQtySideW;
            }

            const totalWidth = contentWidth + currentQtySideW;

            if (cursor.x + totalWidth > CFG.pageW - CFG.margin) {
                cursor.y += CFG.labelH + CFG.gapY;
                cursor.x = CFG.margin;
                
                if (cursor.y + CFG.labelH > CFG.pageH - CFG.margin) {
                    doc.addPage();
                    cursor.y = CFG.margin;
                    cursor.x = CFG.margin;
                }
            }

            // Draw Box
            doc.setDrawColor(0); doc.setLineWidth(0.2);
            doc.rect(cursor.x, cursor.y, contentWidth, CFG.labelH);

            const dividerY = cursor.y + 7; // Divider lebih rendah
            doc.line(cursor.x, dividerY, cursor.x + contentWidth, dividerY);

            // Text Part (Center)
            doc.setTextColor(0,0,0);
            const centerX = cursor.x + (contentWidth / 2);
            doc.text(item.part, centerX, cursor.y + 5, { align: 'center', maxWidth: contentWidth - 2 });

            // Text Desc (Flexible, font sedikit besar)
            doc.setFont("helvetica", "normal"); doc.setFontSize(7); 
            doc.setTextColor(60,60,60);
            const splitDesc = doc.splitTextToSize(item.desc, contentWidth - 4); 
            const descY = cursor.y + 10.5;
            doc.text(splitDesc, centerX, descY, { align: 'center' });

            // Qty Box (Lebar Besar)
            const qtyX = cursor.x + contentWidth;
            doc.setFillColor(241, 245, 249);
            doc.rect(qtyX, cursor.y, currentQtySideW, CFG.labelH, 'FD');

            // Qty Text (Besar)
            doc.setFont("helvetica", "bold"); doc.setFontSize(12); 
            doc.setTextColor(0,0,0);
            const qtyCenterX = qtyX + (currentQtySideW / 2);
            const qtyCenterY = cursor.y + (CFG.labelH / 2) + 1.5; 
            doc.text(String(item.qty), qtyCenterX, qtyCenterY, { align: 'center' });

            cursor.x += totalWidth + CFG.gapX;
        });
    }

    function generateTab3PDF(doc, data) {
        // FUNGSI TAB 3 (KECIL & COLUMN MAJOR)
        
        const availableHeight = CFG.pageH - (2 * CFG.margin);
        const itemTotalHeight = CFG.labelH + CFG.gapY;
        const itemsPerCol = Math.floor(availableHeight / itemTotalHeight);
        const columnsData = [];
        for (let i = 0; i < data.length; i += itemsPerCol) {
            columnsData.push(data.slice(i, i + itemsPerCol));
        }

        let cursorX = CFG.margin;
        
        columnsData.forEach((colItems, colIndex) => {
            let maxColWidth = 0;
            
            // Hitung lebar maks kolom (Kecil)
            colItems.forEach(item => {
                doc.setFont("helvetica", "bold"); doc.setFontSize(9);
                const partWidth = doc.getTextWidth(item.part);
                const paddingX = 6;
                let contentW = partWidth + paddingX;
                if (contentW < CFG.tab2_minW_Small - CFG.qtySideW_Small) contentW = CFG.tab2_minW_Small - CFG.qtySideW_Small;
                if (contentW > CFG.tab2_maxW_Small - CFG.qtySideW_Small) contentW = CFG.tab2_maxW_Small - CFG.qtySideW_Small;
                const totalW = contentW + CFG.qtySideW_Small;
                if (totalW > maxColWidth) maxColWidth = totalW;
            });

            if (cursorX + maxColWidth > CFG.pageW - CFG.margin) {
                doc.addPage();
                cursorX = CFG.margin;
            }

            let cursorY = CFG.margin;
            colItems.forEach((item) => {
                doc.setFont("helvetica", "bold"); doc.setFontSize(9);
                const partWidth = doc.getTextWidth(item.part);
                const paddingX = 6;
                let contentWidth = partWidth + paddingX;
                if (contentWidth < CFG.tab2_minW_Small - CFG.qtySideW_Small) contentWidth = CFG.tab2_minW_Small - CFG.qtySideW_Small;
                if (contentWidth > CFG.tab2_maxW_Small - CFG.qtySideW_Small) contentWidth = CFG.tab2_maxW_Small - CFG.qtySideW_Small;
                const totalWidth = contentWidth + CFG.qtySideW_Small;

                // Draw Box
                doc.setDrawColor(0); doc.setLineWidth(0.2);
                doc.rect(cursorX, cursorY, contentWidth, CFG.labelH);

                const dividerY = cursorY + 5.5; 
                doc.line(cursorX, dividerY, cursorX + contentWidth, dividerY);

                // Text Part (Center)
                doc.setTextColor(0,0,0);
                const centerX = cursorX + (contentWidth / 2);
                doc.text(item.part, centerX, cursorY + 4, { align: 'center', maxWidth: contentWidth - 2 });

                // Text Desc (Center, Small)
                doc.setFont("helvetica", "normal"); doc.setFontSize(6); 
                doc.setTextColor(60,60,60);
                const splitDesc = doc.splitTextToSize(item.desc, contentWidth - 4); 
                doc.text(splitDesc, centerX, cursorY + 9, { align: 'center' });

                // Qty Box (Small)
                const qtyX = cursorX + contentWidth;
                doc.setFillColor(241, 245, 249);
                doc.rect(qtyX, cursorY, CFG.qtySideW_Small, CFG.labelH, 'FD');

                doc.setFont("helvetica", "bold"); doc.setFontSize(9); 
                doc.setTextColor(0,0,0);
                const qtyCenterX = qtyX + (CFG.qtySideW_Small / 2);
                const qtyCenterY = cursorY + (CFG.labelH / 2) + 1; 
                doc.text(String(item.qty), qtyCenterX, qtyCenterY, { align: 'center' });

                cursorY += CFG.labelH + CFG.gapY;
            });

            cursorX += maxColWidth + CFG.gapX;
        });
    }
</script>

</body>
</html>
