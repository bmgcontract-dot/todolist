<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Guest - Visitor Management</title>
    <!-- Tailwind CSS -->
    <script src="[https://cdn.tailwindcss.com](https://cdn.tailwindcss.com)"></script>
    <!-- SweetAlert2 -->
    <script src="[https://cdn.jsdelivr.net/npm/sweetalert2@11](https://cdn.jsdelivr.net/npm/sweetalert2@11)"></script>
    <!-- QR Code Generator -->
    <script src="[https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js](https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js)"></script>
    <!-- HTML5 QR Scanner -->
    <script src="[https://unpkg.com/html5-qrcode](https://unpkg.com/html5-qrcode)" type="text/javascript"></script>
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brand: { 50: '#f0fdf4', 100: '#dcfce7', 500: '#22c55e', 600: '#16a34a', 700: '#15803d' }
                    }
                }
            }
        }
    </script>
    
    <style>
        @import url('[https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600&display=swap](https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600&display=swap)');
        body { font-family: 'Prompt', sans-serif; background-color: #f0fdf4; }
        
        /* สไตล์สำหรับตอนปริ้น Slip ความร้อน */
        @media print {
            body * { visibility: hidden; }
            #slip-container, #slip-container * { visibility: visible; }
            #slip-container { position: absolute; left: 0; top: 0; width: 100%; max-width: 80mm; padding: 10px; box-shadow: none; }
            .no-print { display: none !important; }
        }
    </style>
</head>
<body class="text-gray-800">

    <!-- Navbar -->
    <nav class="bg-brand-600 text-white p-4 shadow-md flex justify-between items-center no-print">
        <h1 class="text-xl font-bold">🏢 My Guest</h1>
        <div class="text-sm" id="nav-mode-text">ระบบ รปภ.</div>
    </nav>

    <div class="max-w-md mx-auto p-4" id="app-container">
        
        <!-- ============================================== -->
        <!-- VIEW 1: หน้าบันทึกผู้มาติดต่อ (Check-in) -->
        <!-- ============================================== -->
        <div id="view-checkin" class="bg-white rounded-xl shadow-lg p-6 mb-6">
            <h2 class="text-lg font-semibold text-brand-700 mb-4 border-b pb-2">บันทึกรถเข้า (Check-in)</h2>
            <form id="checkin-form">
                
                <!-- Contact Type -->
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">สถานที่ติดต่อ</label>
                    <div class="grid grid-cols-2 gap-2">
                        <label class="border rounded p-2 text-sm flex items-center cursor-pointer hover:bg-brand-50">
                            <input type="radio" name="contactType" value="บ้านเลขที่/ห้องชุด" class="mr-2" onchange="toggleRoomInput()" required> ห้องชุด
                        </label>
                        <label class="border rounded p-2 text-sm flex items-center cursor-pointer hover:bg-brand-50">
                            <input type="radio" name="contactType" value="สำนักงานนิติฯ" class="mr-2" onchange="toggleRoomInput()"> นิติฯ
                        </label>
                        <label class="border rounded p-2 text-sm flex items-center cursor-pointer hover:bg-brand-50">
                            <input type="radio" name="contactType" value="เยี่ยมชมโครงการ" class="mr-2" onchange="toggleRoomInput()"> ชมโครงการ
                        </label>
                        <label class="border rounded p-2 text-sm flex items-center cursor-pointer hover:bg-brand-50">
                            <input type="radio" name="contactType" value="อื่นๆ" class="mr-2" onchange="toggleRoomInput()"> อื่นๆ
                        </label>
                    </div>
                </div>

                <!-- Room Autocomplete -->
                <div id="room-input-group" class="mb-4 hidden">
                    <label class="block text-sm font-medium text-gray-700 mb-1">ค้นหาห้อง/บ้านเลขที่</label>
                    <input list="room-list" id="targetRoom" class="w-full border rounded-lg p-2 focus:ring-brand-500 focus:border-brand-500" placeholder="พิมพ์เลขห้อง...">
                    <datalist id="room-list"></datalist>
                </div>

                <!-- Purpose -->
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-1">วัตถุประสงค์</label>
                    <select id="purpose" class="w-full border rounded-lg p-2" required>
                        <option value="">-- เลือกวัตถุประสงค์ --</option>
                        <option value="ญาติ/เพื่อน">ญาติ/เพื่อน</option>
                        <option value="นัดหมายไว้">นัดหมายไว้</option>
                        <option value="ส่งของ/อาหาร">ส่งของ/อาหาร</option>
                        <option value="ตกแต่ง/ช่าง">ตกแต่งต่อเติม/ช่าง</option>
                        <option value="อื่นๆ">อื่นๆ</option>
                    </select>
                </div>

                <!-- Camera / License Plate -->
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-1">ถ่ายรูปทะเบียนรถ</label>
                    <div class="flex items-center justify-center w-full">
                        <label class="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed rounded-lg cursor-pointer bg-gray-50 hover:bg-gray-100 border-gray-300">
                            <div class="flex flex-col items-center justify-center pt-5 pb-6">
                                <svg class="w-8 h-8 mb-4 text-gray-500" aria-hidden="true" xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" fill="none" viewBox="0 0 20 16"><path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 13h3a3 3 0 0 0 0-6h-.025A5.56 5.56 0 0 0 16 6.5 5.5 5.5 0 0 0 5.207 5.021C5.137 5.017 5.071 5 5 5a4 4 0 0 0 0 8h2.167M10 15V6m0 0L8 8m2-2 2 2"/></svg>
                                <p class="text-sm text-gray-500" id="file-name-display"><span class="font-semibold">แตะเพื่อเปิดกล้อง</span></p>
                            </div>
                            <input id="camera-input" type="file" accept="image/*" capture="environment" class="hidden" onchange="previewImage(event)"/>
                        </label>
                    </div>
                    <img id="image-preview" class="hidden mt-2 w-full rounded-lg object-cover h-40">
                </div>

                <button type="button" onclick="submitCheckin()" class="w-full bg-brand-600 hover:bg-brand-700 text-white font-bold py-3 rounded-lg shadow-md transition duration-300">
                    💾 บันทึกข้อมูล
                </button>
            </form>
            
            <hr class="my-6">
            <button onclick="switchView('view-checkout')" class="w-full border-2 border-brand-600 text-brand-600 font-bold py-2 rounded-lg hover:bg-brand-50">
                สลับไปหน้า สแกนรถออก (Check-out)
            </button>
        </div>

        <!-- ============================================== -->
        <!-- VIEW 2: หน้า Slip แสดงผลเพื่อ Print -->
        <!-- ============================================== -->
        <div id="view-slip" class="hidden">
            <div id="slip-container" class="bg-white rounded-xl shadow-lg p-6 text-center mx-auto" style="max-width: 350px;">
                <h2 class="text-xl font-bold text-gray-800">🏢 My Guest</h2>
                <p class="text-sm text-gray-500 mb-4 border-b pb-2">บัตรผู้มาติดต่อ (Visitor Pass)</p>
                
                <div class="text-left text-sm space-y-2 mb-4">
                    <p><strong>ID:</strong> <span id="slip-id"></span></p>
                    <p><strong>วันที่:</strong> <span id="slip-date"></span></p>
                    <p><strong>ติดต่อ:</strong> <span id="slip-room"></span></p>
                    <p><strong>ธุระ:</strong> <span id="slip-purpose"></span></p>
                </div>

                <div class="flex justify-center mb-4">
                    <div id="qrcode-display" class="p-2 border rounded-lg bg-white"></div>
                </div>
                
                <p class="text-xs text-gray-500 mb-4 border-t pt-2">* โปรดนำสลิปนี้ให้เจ้าของบ้านประทับตรา หรือสแกน QR Code ก่อนออกจากโครงการ</p>

                <div class="flex gap-2 no-print">
                    <button onclick="window.print()" class="flex-1 bg-gray-800 text-white py-2 rounded-lg font-bold">🖨️ พิมพ์</button>
                    <button onclick="switchView('view-checkin')" class="flex-1 bg-brand-600 text-white py-2 rounded-lg font-bold">กลับหน้าแรก</button>
                </div>
            </div>
        </div>

        <!-- ============================================== -->
        <!-- VIEW 3: หน้าบันทึกรถออก (Check-out) -->
        <!-- ============================================== -->
        <div id="view-checkout" class="hidden bg-white rounded-xl shadow-lg p-6">
            <h2 class="text-lg font-semibold text-brand-700 mb-4 border-b pb-2">บันทึกรถออก (Check-out)</h2>
            
            <div id="reader" class="w-full mb-4"></div>
            
            <div class="flex gap-2 mb-4">
                <input type="text" id="search-id" placeholder="สแกน QR หรือพิมพ์ ID..." class="w-full border rounded-lg p-2">
                <button onclick="searchVisitor()" class="bg-gray-800 text-white px-4 rounded-lg">ค้นหา</button>
            </div>

            <div id="checkout-result" class="hidden border rounded-lg p-4 bg-gray-50">
                <h3 class="font-bold mb-2">ข้อมูลผู้มาติดต่อ: <span id="co-id"></span></h3>
                <p class="text-sm">สถานะ: <span id="co-status" class="font-bold"></span></p>
                <button onclick="confirmCheckout()" class="mt-4 w-full bg-red-500 text-white py-2 rounded-lg font-bold">บันทึกรถออก</button>
            </div>

            <button onclick="switchView('view-checkin')" class="w-full mt-4 text-brand-600 underline text-sm text-center block">กลับหน้าบันทึกรถเข้า</button>
        </div>

        <!-- ============================================== -->
        <!-- VIEW 4: หน้าลูกบ้านสแกนประทับตรา (Resident Action) -->
        <!-- ============================================== -->
        <div id="view-resident" class="hidden bg-white rounded-xl shadow-lg p-6 text-center">
            <h2 class="text-xl font-bold text-brand-700 mb-2">ระบบประทับตรา (E-Stamp)</h2>
            <p class="text-gray-600 mb-6">สำหรับเจ้าของห้อง</p>
            
            <div class="bg-brand-50 p-4 rounded-lg mb-6 text-left text-sm space-y-2">
                <p><strong>ผู้ติดต่อ ID:</strong> <span id="res-id"></span></p>
                <p><strong>เวลาเข้า:</strong> <span id="res-time"></span></p>
                <p><strong>ติดต่อห้อง:</strong> <span id="res-room"></span></p>
            </div>

            <div class="grid gap-3">
                <button onclick="residentStamp('Stamped (อนุมัติ)')" class="bg-brand-600 text-white py-3 rounded-lg font-bold text-lg shadow-md">✅ ประทับตราอนุมัติ</button>
                <button onclick="residentStamp('Rejected (ปฏิเสธ)')" class="bg-red-500 text-white py-3 rounded-lg font-bold text-lg shadow-md">❌ ปฏิเสธการเข้าพบ</button>
            </div>
        </div>

    </div>

    <!-- Script ควบคุม Logic หน้าเว็บ -->
    <script>
        let currentBase64Image = null;
        let scanActionId = null; 
        
        // ดึงค่า URL Parameter จาก GAS Template
        // <?!= JSON.stringify(urlParams || {}) ?> จะถูกแทนที่ด้วยข้อมูลโดย GAS
        const urlParams = <?!= JSON.stringify(urlParams || {}) ?>;
        const webAppUrl = "<?!= webAppUrl ?>";

        // ทำงานเมื่อโหลดหน้าเว็บเสร็จ
        document.addEventListener('DOMContentLoaded', () => {
            loadResidents();
            
            // เช็คว่าเปิดมาด้วยโหมดไหน (ลูกบ้านสแกน QR หรือ รปภ.ใช้งานปกติ)
            if (urlParams.action === 'scan' && urlParams.id) {
                scanActionId = urlParams.id;
                setupResidentView(scanActionId);
            } else {
                switchView('view-checkin');
            }
        });

        function switchView(viewId) {
            ['view-checkin', 'view-slip', 'view-checkout', 'view-resident'].forEach(id => {
                document.getElementById(id).classList.add('hidden');
            });
            document.getElementById(viewId).classList.remove('hidden');
        }

        // โหลดข้อมูลห้องใส่ Datalist
        function loadResidents() {
            google.script.run.withSuccessHandler(rooms => {
                let datalist = document.getElementById('room-list');
                rooms.forEach(room => {
                    let option = document.createElement('option');
                    option.value = room;
                    datalist.appendChild(option);
                });
            }).getResidentsList();
        }

        function toggleRoomInput() {
            let type = document.querySelector('input[name="contactType"]:checked').value;
            let roomGroup = document.getElementById('room-input-group');
            let roomInput = document.getElementById('targetRoom');
            
            if (type === 'บ้านเลขที่/ห้องชุด') {
                roomGroup.classList.remove('hidden');
                roomInput.required = true;
            } else {
                roomGroup.classList.add('hidden');
                roomInput.required = false;
                roomInput.value = '';
            }
        }

        function previewImage(event) {
            const file = event.target.files[0];
            if (file) {
                document.getElementById('file-name-display').innerText = file.name;
                const reader = new FileReader();
                reader.onload = function(e) {
                    currentBase64Image = e.target.result;
                    let imgPreview = document.getElementById('image-preview');
                    imgPreview.src = currentBase64Image;
                    imgPreview.classList.remove('hidden');
                }
                reader.readAsDataURL(file);
            }
        }

        // ==========================================
        // ส่วนของ รปภ. (Check-in)
        // ==========================================
        function submitCheckin() {
            let form = document.getElementById('checkin-form');
            if(!form.checkValidity()) {
                form.reportValidity();
                return;
            }

            let type = document.querySelector('input[name="contactType"]:checked')?.value;
            let room = document.getElementById('targetRoom').value;
            let purpose = document.getElementById('purpose').value;

            if(!currentBase64Image) {
                Swal.fire('แจ้งเตือน', 'กรุณาถ่ายรูปทะเบียนรถ', 'warning');
                return;
            }

            Swal.fire({ title: 'กำลังบันทึกข้อมูล...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});

            let payload = { contactType: type, targetRoom: room, purpose: purpose, imageBase64: currentBase64Image };

            google.script.run.withSuccessHandler(response => {
                Swal.close();
                if(response.success) {
                    generateSlip(response.id, response.data);
                }
            }).saveVisitor(payload);
        }

        function generateSlip(id, data) {
            // data format: [id, date, timeIn, timeOut, type, room, img, purpose, status]
            document.getElementById('slip-id').innerText = id;
            document.getElementById('slip-date').innerText = `${data[1]} ${data[2]}`;
            document.getElementById('slip-room').innerText = data[4] + (data[5] ? ` (${data[5]})` : '');
            document.getElementById('slip-purpose').innerText = data[7];
            
            document.getElementById('qrcode-display').innerHTML = "";
            
            // สร้าง QR Code เป็น Link ให้ลูกบ้านสแกน หรือ รปภ. สแกน
            let qrUrl = `${webAppUrl}?action=scan&id=${id}`;
            new QRCode(document.getElementById("qrcode-display"), {
                text: qrUrl,
                width: 128, height: 128,
                colorDark : "#000000", colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });

            switchView('view-slip');
            
            // Reset Form
            document.getElementById('checkin-form').reset();
            currentBase64Image = null;
            document.getElementById('image-preview').classList.add('hidden');
            document.getElementById('file-name-display').innerHTML = "<span class='font-semibold'>แตะเพื่อเปิดกล้อง</span>";
            toggleRoomInput();
        }

        // ==========================================
        // ส่วนของ รปภ. (Check-out)
        // ==========================================
        let html5QrcodeScanner;
        function initScanner() {
            if(!html5QrcodeScanner) {
                html5QrcodeScanner = new Html5QrcodeScanner("reader", { fps: 10, qrbox: {width: 250, height: 250} }, false);
                html5QrcodeScanner.render((decodedText, decodedResult) => {
                    // ดึง ID ออกจาก URL ที่สแกนได้
                    let urlParams = new URLSearchParams(decodedText.split('?')[1]);
                    let scannedId = urlParams.get('id') || decodedText;
                    document.getElementById('search-id').value = scannedId;
                    searchVisitor();
                    html5QrcodeScanner.clear();
                });
            }
        }

        // ดักจับการเปิดหน้า Checkout เพื่อเปิดกล้อง
        const originalSwitchView = switchView;
        switchView = function(viewId) {
            originalSwitchView(viewId);
            if(viewId === 'view-checkout') setTimeout(initScanner, 500);
            else if (html5QrcodeScanner) { html5QrcodeScanner.clear(); html5QrcodeScanner = null; }
        }

        function searchVisitor() {
            let id = document.getElementById('search-id').value.trim();
            if(!id) return;
            
            Swal.fire({ title: 'กำลังค้นหา...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});
            
            google.script.run.withSuccessHandler(visitor => {
                Swal.close();
                if(visitor) {
                    document.getElementById('co-id').innerText = visitor.ID;
                    let statusEl = document.getElementById('co-status');
                    statusEl.innerText = visitor.Status;
                    
                    if(visitor.Status.includes('อนุมัติ')) statusEl.className = "font-bold text-green-600";
                    else if(visitor.Status.includes('ปฏิเสธ')) statusEl.className = "font-bold text-red-600";
                    else statusEl.className = "font-bold text-yellow-600";

                    document.getElementById('checkout-result').classList.remove('hidden');
                } else {
                    Swal.fire('ไม่พบข้อมูล', 'ไม่พบรหัสผู้มาติดต่อนี้', 'error');
                }
            }).getVisitorById(id);
        }

        function confirmCheckout() {
            let id = document.getElementById('co-id').innerText;
            Swal.fire({ title: 'กำลังบันทึกรถออก...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});
            
            google.script.run.withSuccessHandler(res => {
                if(res.success) {
                    Swal.fire('สำเร็จ', res.message, 'success');
                    document.getElementById('checkout-result').classList.add('hidden');
                    document.getElementById('search-id').value = '';
                }
            }).updateVisitorStatus(id, null, true);
        }

        // ==========================================
        // ส่วนของ ลูกบ้าน (QR Scan Resident Action)
        // ==========================================
        function setupResidentView(id) {
            document.getElementById('nav-mode-text').innerText = "ระบบประทับตรา";
            switchView('view-resident');
            Swal.fire({ title: 'กำลังดึงข้อมูล...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});
            
            google.script.run.withSuccessHandler(visitor => {
                Swal.close();
                if(visitor) {
                    document.getElementById('res-id').innerText = visitor.ID;
                    document.getElementById('res-time').innerText = visitor.Date + " " + visitor.TimeIn;
                    document.getElementById('res-room').innerText = visitor.TargetRoom;
                    
                    if(visitor.Status !== 'Pending') {
                        Swal.fire('แจ้งเตือน', `รายการนี้ถูก ${visitor.Status} ไปแล้ว`, 'info');
                    }
                } else {
                    Swal.fire('ข้อผิดพลาด', 'ไม่พบข้อมูลผู้มาติดต่อ', 'error');
                }
            }).getVisitorById(id);
        }

        function residentStamp(status) {
            Swal.fire({ title: 'กำลังบันทึก...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});
            google.script.run.withSuccessHandler(res => {
                if(res.success) {
                    Swal.fire('สำเร็จ', `ดำเนินการ ${status} เรียบร้อยแล้ว`, 'success');
                }
            }).updateVisitorStatus(scanActionId, status, false);
        }
    </script>
</body>
</html>
