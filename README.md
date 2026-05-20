<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulasi QRIS Pay</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen sm:p-4 bg-gray-100">

    <div class="w-full sm:max-w-sm bg-white sm:rounded-2xl shadow-xl flex flex-col h-screen sm:h-auto sm:min-h-[700px] relative overflow-hidden" id="app-container">
        
        <!-- ================= LAYAR 1: SCANNER QRIS ================= -->
        <div id="screen-scan" class="flex flex-col h-full p-6">
            <div class="text-center mb-6 pt-4">
                <h2 class="text-xl font-bold text-gray-800">Pindai QRIS</h2>
                <p class="text-sm text-gray-500 mt-1">Arahkan kamera HP ke kode QRIS</p>
            </div>
            
            <div id="reader" class="overflow-hidden rounded-xl bg-gray-50 border border-gray-200 w-full min-h-[250px]"></div>
            
            <div class="mt-auto pt-8">
                <p class="text-xs text-gray-400 text-center mb-2">Atau masukkan teks data QRIS manual</p>
                <textarea id="manual-qris" rows="2" class="w-full p-3 text-xs border rounded-xl focus:outline-none focus:ring-2 focus:ring-green-500 bg-gray-50" placeholder="Tempel kode QRIS di sini..."></textarea>
                <button onclick="processManualQRIS()" class="w-full mt-3 bg-gray-800 text-white text-sm py-3 rounded-xl font-semibold hover:bg-gray-900 transition">Proses Data</button>
            </div>
        </div>

        <!-- ================= LAYAR 2: INPUT NOMINAL ================= -->
        <div id="screen-nominal" class="hidden flex flex-col h-full justify-between p-6 bg-white">
            <div class="text-center pt-4 mb-6">
                <span class="bg-green-100 text-green-800 text-xs font-semibold px-3 py-1 rounded-full">QRIS Ditemukan</span>
                <h2 id="target-merchant" class="text-2xl font-bold text-gray-900 mt-4">Nama Tujuan</h2>
                <p class="text-sm text-gray-500 mt-1">Sumber Dana: Saldo Utama</p>
            </div>

            <div class="my-auto">
                <label class="block text-sm font-medium text-gray-600 text-center mb-3">Nominal Pembayaran</label>
                <div class="relative mt-1 rounded-md shadow-sm">
                    <div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-4">
                        <span class="text-gray-500 text-xl font-bold">Rp</span>
                    </div>
                    <input type="number" id="input-amount" class="block w-full rounded-2xl border-gray-300 pl-14 pr-4 text-3xl font-bold text-gray-900 focus:border-green-500 focus:ring-green-500 py-4 border bg-gray-50 focus:bg-white focus:outline-none transition" placeholder="0" min="1">
                </div>
            </div>

            <button onclick="executePayment()" class="w-full bg-green-500 text-white font-bold py-4 rounded-xl shadow-md hover:bg-green-600 transition text-lg mt-8">
                Bayar Sekarang
            </button>
        </div>

        <!-- ================= LAYAR 3: STRUK SUKSES ================= -->
        <div id="screen-receipt" class="hidden flex-col h-full bg-gray-100 relative">
            
            <div class="flex-1 overflow-y-auto">
                <div id="receipt-capture" class="p-4 pt-6 pb-6 bg-gray-100 flex flex-col items-center">
                    
                    <!-- Kartu Struk Putih -->
                    <div class="bg-white rounded-2xl shadow-sm overflow-hidden w-full">
                        
                        <!-- Header & Nominal (Satu wadah, murni rata tengah, TANPA BORDER BAWAH) -->
                        <div class="pt-8 pb-6 px-6 block text-center w-full">
                            
                            <!-- Ikon Ceklis -->
                            <div class="inline-flex items-center justify-center w-14 h-14 bg-green-100 rounded-full mb-4">
                                <i data-lucide="check" class="w-8 h-8 text-green-500 stroke-[3px]"></i>
                            </div>
                            
                            <!-- Teks Berhasil -->
                            <h2 class="text-xl font-bold text-gray-800 text-center block w-full m-0">Pembayaran Berhasil</h2>
                            <p class="text-sm text-gray-500 mt-1 text-center block w-full m-0">Transaksi Anda sukses diproses.</p>
                            
                            <div class="mt-6"></div>

                            <!-- Nominal -->
                            <p class="text-sm font-medium text-gray-500 mb-1 text-center block w-full m-0">Total Bayar</p>
                            <h1 id="receipt-amount" class="text-4xl font-bold text-gray-900 text-center block w-full m-0">Rp0</h1>
                            
                        </div>

                        <!-- SATU-SATUNYA GARIS: Efek sobekan struk putus-putus -->
                        <div class="relative flex items-center justify-center w-full h-6 overflow-hidden bg-white">
                            <div class="absolute -left-3 w-6 h-6 bg-gray-100 rounded-full"></div>
                            <div class="w-full mx-4 border-t-2 border-dashed border-gray-300 h-0"></div>
                            <div class="absolute -right-3 w-6 h-6 bg-gray-100 rounded-full"></div>
                        </div>

                        <!-- Rincian Transaksi -->
                        <div class="p-6 space-y-4 text-left">
                            <h3 class="text-sm font-bold text-gray-800 mb-2">Rincian Transaksi</h3>
                            
                            <div class="flex justify-between items-start gap-4">
                                <p class="text-sm text-gray-500 flex-shrink-0">Atas Nama</p>
                                <p id="receipt-merchant" class="text-sm font-semibold text-gray-900 text-right break-words">Nama Merchant</p>
                            </div>

                            <div class="flex justify-between items-start gap-4">
                                <p class="text-sm text-gray-500 flex-shrink-0">Ke Rekening</p>
                                <p class="text-sm font-semibold text-gray-900 text-right">QRIS</p>
                            </div>

                            <div class="flex justify-between items-start gap-4">
                                <p class="text-sm text-gray-500 flex-shrink-0">Waktu Transaksi</p>
                                <p id="receipt-time" class="text-sm font-medium text-gray-900 text-right">Waktu Realtime</p>
                            </div>

                            <div class="flex justify-between items-start gap-4">
                                <p class="text-sm text-gray-500 flex-shrink-0">Nomor Resi</p>
                                <p id="receipt-ref" class="text-sm font-medium text-gray-900 text-right tracking-wider">XXXXXX</p>
                            </div>
                            
                            <div class="flex justify-between items-start gap-4">
                                <p class="text-sm text-gray-500 flex-shrink-0">Metode</p>
                                <p class="text-sm font-medium text-gray-900 text-right">Saldo Utama</p>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Info Box -->
                    <div class="mt-4 bg-blue-50 rounded-xl p-3 flex gap-3 items-start border border-blue-100 w-full">
                        <i data-lucide="info" class="w-5 h-5 text-blue-500 flex-shrink-0 mt-0.5"></i>
                        <p class="text-xs text-blue-800 leading-relaxed">Simpan resi ini sebagai bukti pembayaran yang sah.</p>
                    </div>
                </div>
            </div>

            <!-- Tombol Aksi -->
            <div class="p-4 bg-white border-t border-gray-200 grid grid-cols-2 gap-3 mt-auto">
                <button onclick="resetApp()" class="flex items-center justify-center gap-2 bg-white text-gray-700 font-semibold py-3.5 rounded-xl shadow-sm hover:bg-gray-50 transition border border-gray-200">
                    <i data-lucide="scan" class="w-5 h-5"></i>
                    Scan Lagi
                </button>
                <button onclick="downloadHD()" id="btn-download" class="flex items-center justify-center gap-2 bg-green-500 text-white font-semibold py-3.5 rounded-xl shadow-md hover:bg-green-600 transition">
                    <i data-lucide="download" class="w-5 h-5"></i>
                    <span>Simpan</span>
                </button>
            </div>
        </div>

    </div>

    <script>
        let html5QrcodeScanner;
        let finalMerchantName = "Merchant Tidak Dikenal";

        window.addEventListener('DOMContentLoaded', () => {
            lucide.createIcons();
            startScanner();
        });

        function startScanner() {
            html5QrcodeScanner = new Html5Qrcode("reader");
            const config = { fps: 10, qrbox: { width: 250, height: 250 } };
            
            html5QrcodeScanner.start(
                { facingMode: "environment" }, 
                config, 
                (decodedText) => {
                    html5QrcodeScanner.stop().then(() => {
                        processQRISData(decodedText);
                    });
                },
                (errorMessage) => {}
            ).catch(err => {
                console.error("Kamera gagal diakses.", err);
                document.getElementById('reader').innerHTML = `<p class="p-4 text-xs text-red-500 text-center mt-10">Kamera tidak diizinkan atau gagal dimuat. Gunakan kolom teks di bawah.</p>`;
            });
        }

        function extractMerchantName(qrisText) {
            try {
                let i = 0;
                while (i < qrisText.length) {
                    let id = qrisText.substring(i, i + 2);
                    let len = parseInt(qrisText.substring(i + 2, i + 4));
                    let val = qrisText.substring(i + 4, i + 4 + len);
                    
                    if (id === '59') { 
                        return val.trim();
                    }
                    if (isNaN(len) || len <= 0) break; 
                    i += 4 + len;
                }
            } catch (e) {
                console.error("Gagal membaca standar QRIS");
            }
            return null;
        }

        function processManualQRIS() {
            const textData = document.getElementById('manual-qris').value;
            if(textData.trim() !== "") {
                if(html5QrcodeScanner && html5QrcodeScanner.isScanning) {
                    html5QrcodeScanner.stop().then(() => {
                        processQRISData(textData);
                    });
                } else {
                    processQRISData(textData);
                }
            }
        }

        function processQRISData(rawText) {
            let parsedName = extractMerchantName(rawText);
            finalMerchantName = parsedName ? parsedName : "Tujuan Tidak Diketahui";
            
            document.getElementById('screen-scan').classList.add('hidden');
            document.getElementById('screen-nominal').classList.remove('hidden');
            document.getElementById('target-merchant').innerText = finalMerchantName;
        }

        function executePayment() {
            const amountInput = document.getElementById('input-amount').value;
            if(!amountInput || amountInput <= 0) {
                alert("Masukkan nominal dengan benar!");
                return;
            }

            const formattedAmount = new Intl.NumberFormat('id-ID', {
                style: 'currency',
                currency: 'IDR',
                minimumFractionDigits: 0
            }).format(amountInput).replace("IDR", "Rp");

            const sekarang = new Date();
            const bulanIndo = ["Januari", "Februari", "Maret", "April", "Mei", "Juni", "Juli", "Agustus", "September", "Oktober", "November", "Desember"];
            const tanggalStr = `${sekarang.getDate()} ${bulanIndo[sekarang.getMonth()]} ${sekarang.getFullYear()}`;
            const jamStr = `${String(sekarang.getHours()).padStart(2, '0')}:${String(sekarang.getMinutes()).padStart(2, '0')}`;
            const waktuRealtime = `${tanggalStr}, ${jamStr} WIB`;

            const karakter = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ';
            let nomorRefAcak = 'IDM';
            for (let i = 0; i < 12; i++) {
                nomorRefAcak += karakter.charAt(Math.floor(Math.random() * karakter.length));
            }

            document.getElementById('receipt-amount').innerText = formattedAmount;
            document.getElementById('receipt-merchant').innerText = finalMerchantName;
            document.getElementById('receipt-time').innerText = waktuRealtime;
            document.getElementById('receipt-ref').innerText = nomorRefAcak;

            document.getElementById('screen-nominal').classList.add('hidden');
            document.getElementById('screen-receipt').classList.remove('hidden');
            document.getElementById('screen-receipt').classList.add('flex');
            
            lucide.createIcons();
        }

        function downloadHD() {
            const element = document.getElementById('receipt-capture');
            const btn = document.getElementById('btn-download');
            const btnSpan = btn.querySelector('span');
            const originalText = btnSpan.innerText;
            
            btnSpan.innerText = 'Menyimpan...';

            html2canvas(element, {
                scale: 3, 
                backgroundColor: '#f3f4f6', 
                useCORS: true 
            }).then(canvas => {
                const link = document.createElement('a');
                link.download = `Struk_Simulasi_${finalMerchantName.replace(/\s+/g, '_')}.png`;
                link.href = canvas.toDataURL('image/png');
                link.click();
                btnSpan.innerText = originalText;
            }).catch(err => {
                console.error("Gagal simpan struk", err);
                btnSpan.innerText = originalText;
            });
        }

        function resetApp() {
            document.getElementById('screen-receipt').classList.add('hidden');
            document.getElementById('screen-receipt').classList.remove('flex');
            document.getElementById('screen-nominal').classList.add('hidden');
            document.getElementById('screen-scan').classList.remove('hidden');
            document.getElementById('input-amount').value = '';
            document.getElementById('manual-qris').value = '';
            
            startScanner();
        }
    </script>
</body>
</html>
