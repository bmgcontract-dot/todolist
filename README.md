<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MY Guest - Security Console</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <!-- QRCode.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- HTML5 QR Code Scanner -->
    <script src="https://unpkg.com/html5-qrcode"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        pastel: {
                            50: '#f0fdf4',
                            100: '#dcfce7',
                            500: '#22c55e',
                            600: '#16a34a',
                            700: '#15803d'
                        }
                    }
                }
            }
        }
    </script>

    <style>
        body { font-family: 'Sarabun', sans-serif; background-color: #f0fdf4; }
        .view-section { display: none; }
        .view-section.active { display: block; }
        
        /* สไตล์สำหรับการพิมพ์สลิปด้วยเครื่องปริ้นความร้อน */
        @media print {
            body * { visibility: hidden; }
            #view-slip, #view-slip * { visibility: visible; }
            #view-slip { 
                position: absolute; 
                left: 0; top: 0; 
                width: 58mm; /* ขนาดกระดาษความร้อนมาตรฐาน */
                padding: 0;
                margin: 0;
                background: white;
            }
            .no-print { display: none !important; }
            .print-border { border: 1px solid #000; padding: 10px; }
        }
    </style>
</head>
<body class="text-gray-800 h-screen flex flex-col">

    <!-- Navbar (ซ่อนตอน Print) -->
    <nav class="bg-pastel-600 text-white p-4 shadow-md no-print flex justify-between items-center">
        <h1 class="text-xl font-bold flex items-center">
            <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"></path></svg>
            MY Guest
        </h1>
        <div class="flex space-x-2">
            <button onclick="switchView('checkin')" class="bg-white text-pastel-600 px-3 py-1 rounded text-sm font-medium hover:bg-pastel-100">บันทึกเข้า</button>
            <button onclick="switchView('checkout')" class="bg-pastel-700 text-white px-3 py-1 rounded text-sm font-medium hover:bg-pastel-500">บันทึกออก</button>
            <button onclick="switchView('register')" class="bg-pastel-700 text-white px-3 py-1 rounded text-sm font-medium hover:bg-pastel-500">QR ลูกบ้าน</button>
        </div>
    </nav>

    <main class="flex-1 overflow-y-auto p-4 md:p-6 pb-20">
        
        <!-- ============================================== -->
        <!-- VIEW 1: บันทึกผู้มาติดต่อ (Check-in) -->
        <!-- ============================================== -->
        <div id="view-checkin" class="view-section active max-w-lg mx-auto bg-white rounded-xl shadow-lg p-6 border-t-4 border-pastel-500">
            <h2 class="text-2xl font-bold text-center mb-6 text-pastel-700">บันทึกผู้มาติดต่อ</h2>
            
            <form id="checkinForm" onsubmit="submitForm(event)">
                <!-- ผู้ที่มาติดต่อ -->
                <div class="mb-4">
                    <label class="block text-sm font-semibold mb-2">สถานที่ติดต่อ</label>
                    <div class="grid grid-cols-2 gap-2">
                        <label class="border rounded p-3 flex items-center cursor-pointer hover:bg-pastel-50">
                            <input type="radio" name="contactType" value="Room" class="mr-2" onchange="toggleRoomInput()" required> บ้าน/ห้องชุด
                        </label>
                        <label class="border rounded p-3 flex items-center cursor-pointer hover:bg-pastel-50">
                            <input type="radio" name="contactType" value="Office" class="mr-2" onchange="toggleRoomInput()"> นิติบุคคล
                        </label>
                        <label class="border rounded p-3 flex items-center cursor-pointer hover:bg-pastel-50">
                            <input type="radio" name="contactType" value="Tour" class="mr-2" onchange="toggleRoomInput()"> เยี่ยมชมโครงการ
                        </label>
                        <label class="border rounded p-3 flex items-center cursor-pointer hover:bg-pastel-50">
                            <input type="radio" name="contactType" value="Other" class="mr-2" onchange="toggleRoomInput()"> อื่นๆ
                        </label>
                    </div>
                </div>

                <!-- Input บ้านเลขที่ (แสดงเมื่อเลือก บ้าน/ห้องชุด) -->
                <div id="roomInputContainer" class="mb-4 hidden">
                    <label class="block text-sm font-semibold mb-2">ค้นหาบ้านเลขที่ / ห้อง</label>
                    <input type="text" id="targetRoom" list="residentList" class="w-full p-3 border rounded focus:ring focus:ring-pastel-200 outline-none" placeholder="พิมพ์เลขห้อง...">
                    <datalist id="residentList"></datalist>
                </div>

                <!-- วัตถุประสงค์ -->
                <div class="mb-4">
                    <label class="block text-sm font-semibold mb-2">วัตถุประสงค์</label>
                    <select id="purpose" class="w-full p-3 border rounded focus:ring focus:ring-pastel-200 outline-none" required onchange="toggleOtherPurpose()">
                        <option value="">-- เลือกวัตถุประสงค์ --</option>
                        <option value="ญาติ/เพื่อน">ญาติ/เพื่อน</option>
                        <option value="นัดหมายล่วงหน้า">นัดหมายล่วงหน้า</option>
                        <option value="ส่งอาหาร/พัสดุ">ส่งอาหาร/พัสดุ (Rider)</option>
                        <option value="ตกแต่งต่อเติม">ตกแต่ง/ต่อเติม/ซ่อมแซม</option>
                        <option value="Other">อื่นๆ</option>
                    </select>
                    <input type="text" id="otherPurpose" class="w-full p-3 border rounded mt-2 hidden" placeholder="ระบุวัตถุประสงค์...">
                </div>

                <!-- ถ่ายรูปทะเบียนรถ -->
                <div class="mb-6">
                    <label class="block text-sm font-semibold mb-2">ถ่ายรูปทะเบียนรถ / บัตร</label>
                    <div class="flex items-center justify-center w-full">
                        <label class="flex flex-col w-full h-32 border-2 border-dashed border-pastel-400 hover:bg-pastel-50 hover:border-pastel-500 rounded cursor-pointer transition">
                            <div class="flex flex-col items-center justify-center pt-7">
                                <svg class="w-8 h-8 text-pastel-500 mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                                <p class="text-sm text-gray-500" id="photoStatus">แตะเพื่อถ่ายรูป</p>
                            </div>
                            <!-- capture="environment" บังคับเปิดกล้องหลังมือถือ -->
                            <input type="file" id="cameraInput" accept="image/*" capture="environment" class="hidden" onchange="handlePhoto(this)">
                        </label>
                    </div>
                </div>

                <button type="submit" id="btnSubmit" class="w-full bg-pastel-600 hover:bg-pastel-700 text-white font-bold py-3 px-4 rounded-lg shadow-md transition flex justify-center items-center">
                    บันทึกข้อมูลเข้า
                </button>
            </form>
        </div>

        <!-- ============================================== -->
        <!-- VIEW 2: สลิปผู้มาติดต่อ (Slip & Print) -->
        <!-- ============================================== -->
        <div id="view-slip" class="view-section max-w-sm mx-auto bg-white rounded-xl shadow-lg p-6 border-t-4 border-pastel-500 text-center">
            <div id="slip-content" class="print-border">
                <h2 class="text-2xl font-bold text-gray-800 mb-1">MY Guest</h2>
                <p class="text-sm text-gray-500 mb-4 border-b pb-2">บัตรผู้มาติดต่อ (Visitor Pass)</p>
                
                <div class="text-left text-sm mb-4 space-y-2">
                    <p><strong>ID:</strong> <span id="slip-id">V-xxx</span></p>
                    <p><strong>วันที่:</strong> <span id="slip-date"></span></p>
                    <p><strong>เวลาเข้า:</strong> <span id="slip-time"></span></p>
                    <p><strong>ติดต่อ:</strong> <span id="slip-room"></span></p>
                    <p><strong>เหตุผล:</strong> <span id="slip-purpose"></span></p>
                </div>

                <div class="flex justify-center mb-4">
                    <div id="qrcode" class="p-2 bg-white border rounded"></div>
                </div>
                
                <p class="text-xs text-gray-500 mt-2">โปรดให้เจ้าของห้องสแกน QR Code<br>หรือประทับตราก่อนออกจากโครงการ</p>
            </div>

            <div class="mt-6 flex space-x-2 no-print">
                <button onclick="window.print()" class="flex-1 bg-gray-800 hover:bg-gray-900 text-white font-bold py-2 px-4 rounded">
                    🖨️ พิมพ์สลิป
                </button>
                <button onclick="switchView('checkin'); resetForm();" class="flex-1 bg-pastel-600 hover:bg-pastel-700 text-white font-bold py-2 px-4 rounded">
                    เสร็จสิ้น
                </button>
            </div>
        </div>

        <!-- ============================================== -->
        <!-- VIEW 3: ตรวจสอบรถออก (Check-out) -->
        <!-- ============================================== -->
        <div id="view-checkout" class="view-section max-w-lg mx-auto bg-white rounded-xl shadow-lg p-6 border-t-4 border-gray-800">
            <h2 class="text-2xl font-bold text-center mb-6">บันทึกรถออก (Check-Out)</h2>
            
            <div class="mb-4">
                <div id="qr-reader" class="w-full mb-4"></div>
                <button onclick="startScanner()" id="btnStartScan" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 rounded mb-4">
                    📷 เปิดกล้องสแกน QR Code
                </button>
                <button onclick="stopScanner()" id="btnStopScan" class="w-full bg-red-500 hover:bg-red-600 text-white py-2 rounded mb-4 hidden">
                    ปิดกล้อง
                </button>
            </div>

            <div class="flex items-center mb-6">
                <div class="flex-1 border-t border-gray-300"></div>
                <span class="px-3 text-gray-400 text-sm">หรือพิมพ์รหัส</span>
                <div class="flex-1 border-t border-gray-300"></div>
            </div>

            <div class="flex space-x-2">
                <input type="text" id="checkoutSearch" class="flex-1 p-3 border rounded focus:ring outline-none" placeholder="V-YYYYMMDD-XXXX">
                <button onclick="manualCheckout()" class="bg-gray-800 text-white px-4 rounded font-bold">ค้นหา</button>
            </div>
        </div>

        <!-- ============================================== -->
        <!-- VIEW 4: QR ลงทะเบียนลูกบ้าน -->
        <!-- ============================================== -->
        <div id="view-register" class="view-section max-w-sm mx-auto bg-white rounded-xl shadow-lg p-6 border-t-4 border-blue-500 text-center">
            <h2 class="text-xl font-bold mb-4">สร้าง QR ผูก LINE ลูกบ้าน</h2>
            <input type="text" id="regRoomNo" class="w-full p-3 border rounded mb-4 text-center text-lg" placeholder="ใส่บ้านเลขที่ / ห้องชุด">
            <button onclick="generateRegQR()" class="w-full bg-blue-600 text-white py-2 rounded font-bold mb-4">สร้าง QR Code</button>
            
            <div id="regQrContainer" class="hidden flex flex-col items-center">
                <div id="qrcodeReg" class="p-2 border rounded mb-2"></div>
                <p class="text-sm text-gray-600">ให้ลูกบ้านสแกนด้วยแอป LINE</p>
                <p class="text-xs text-gray-400 mt-1">(ระบบจะเปิดหน้าต่างแชทพร้อมส่งข้อความลงทะเบียน)</p>
            </div>
        </div>

    </main>

    <!-- Scripts Logic -->
    <script>
        let base64Photo = "";
        let html5QrcodeScanner = null;

        // ดึงข้อมูลลูกบ้านมาใส่ Datalist ตอนเปิดเว็บ
        window.onload = function() {
            if (typeof google !== 'undefined' && google.script) {
                google.script.run.withSuccessHandler(function(data) {
                    const list = document.getElementById('residentList');
                    data.forEach(room => {
                        let option = document.createElement('option');
                        option.value = room;
                        list.appendChild(option);
                    });
                }).getResidentsList();
            } else {
                console.log("รันนอก Google Apps Script: ใช้ข้อมูลจำลอง");
                const list = document.getElementById('residentList');
                ["A101", "A102", "B201"].forEach(room => {
                    let option = document.createElement('option');
                    option.value = room;
                    list.appendChild(option);
                });
            }
        };

        function switchView(viewName) {
            document.querySelectorAll('.view-section').forEach(el => el.classList.remove('active'));
            document.getElementById('view-' + viewName).classList.add('active');
            if(html5QrcodeScanner && viewName !== 'checkout') stopScanner();
        }

        function toggleRoomInput() {
            const type = document.querySelector('input[name="contactType"]:checked').value;
            const container = document.getElementById('roomInputContainer');
            const roomInput = document.getElementById('targetRoom');
            if (type === 'Room') {
                container.classList.remove('hidden');
                roomInput.required = true;
            } else {
                container.classList.add('hidden');
                roomInput.required = false;
                roomInput.value = '';
            }
        }

        function toggleOtherPurpose() {
            const select = document.getElementById('purpose');
            const otherInput = document.getElementById('otherPurpose');
            if (select.value === 'Other') {
                otherInput.classList.remove('hidden');
                otherInput.required = true;
            } else {
                otherInput.classList.add('hidden');
                otherInput.required = false;
            }
        }

        function handlePhoto(input) {
            if (input.files && input.files[0]) {
                const file = input.files[0];
                const reader = new FileReader();
                reader.onload = function(e) {
                    base64Photo = e.target.result;
                    document.getElementById('photoStatus').innerHTML = "✅ รูปถ่ายพร้อมแล้ว<br>(" + file.name + ")";
                    document.getElementById('photoStatus').classList.add('text-pastel-600');
                };
                reader.readAsDataURL(file);
            }
        }

        function submitForm(e) {
            e.preventDefault();
            const btn = document.getElementById('btnSubmit');
            btn.disabled = true;
            btn.innerHTML = '<svg class="animate-spin h-5 w-5 mr-3 text-white" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg> กำลังบันทึก...';

            const contactType = document.querySelector('input[name="contactType"]:checked').value;
            const room = document.getElementById('targetRoom').value;
            let purpose = document.getElementById('purpose').value;
            if (purpose === 'Other') purpose = document.getElementById('otherPurpose').value;

            const formData = {
                contactType: contactType,
                targetRoom: contactType === 'Room' ? room : contactType,
                purpose: purpose,
                photoBase64: base64Photo
            };

            if (typeof google !== 'undefined' && google.script) {
                google.script.run
                    .withSuccessHandler(function(response) {
                        btn.disabled = false;
                        btn.innerHTML = 'บันทึกข้อมูลเข้า';
                        
                        if(response.success) {
                            showSlip(response);
                            Swal.fire('สำเร็จ', 'บันทึกข้อมูลผู้ติดต่อเรียบร้อยแล้ว', 'success');
                        } else {
                            Swal.fire('ข้อผิดพลาด', response.error, 'error');
                        }
                    })
                    .withFailureHandler(function(err) {
                        btn.disabled = false;
                        btn.innerHTML = 'บันทึกข้อมูลเข้า';
                        Swal.fire('ข้อผิดพลาด', 'ไม่สามารถเชื่อมต่อเซิร์ฟเวอร์ได้', 'error');
                    })
                    .saveVisitor(formData);
            } else {
                setTimeout(() => {
                    btn.disabled = false;
                    btn.innerHTML = 'บันทึกข้อมูลเข้า';
                    let mockResponse = { success: true, id: "V-20260308-9999", time: "16:16", date: "08/03/2026", photo: "" };
                    showSlip(mockResponse);
                    Swal.fire('สำเร็จ', 'บันทึกข้อมูลผู้ติดต่อเรียบร้อยแล้ว (จำลอง)', 'success');
                }, 1000);
            }
        }

        function showSlip(data) {
            document.getElementById('slip-id').innerText = data.id;
            document.getElementById('slip-date').innerText = data.date;
            document.getElementById('slip-time').innerText = data.time;
            
            const typeNode = document.querySelector('input[name="contactType"]:checked').value;
            const room = document.getElementById('targetRoom').value;
            document.getElementById('slip-room').innerText = typeNode === 'Room' ? "ห้อง " + room : typeNode;
            
            let purpose = document.getElementById('purpose').value;
            document.getElementById('slip-purpose').innerText = purpose === 'Other' ? document.getElementById('otherPurpose').value : purpose;

            // Generate QR
            document.getElementById('qrcode').innerHTML = '';
            new QRCode(document.getElementById("qrcode"), {
                text: data.id,
                width: 128,
                height: 128
            });

            switchView('slip');
        }

        function resetForm() {
            document.getElementById('checkinForm').reset();
            document.getElementById('photoStatus').innerHTML = "แตะเพื่อถ่ายรูป";
            document.getElementById('photoStatus').classList.remove('text-pastel-600');
            base64Photo = "";
            toggleRoomInput();
            toggleOtherPurpose();
        }

        // ==========================================
        // Scanner Logic (Checkout)
        // ==========================================
        function startScanner() {
            document.getElementById('btnStartScan').classList.add('hidden');
            document.getElementById('btnStopScan').classList.remove('hidden');
            
            html5QrcodeScanner = new Html5Qrcode("qr-reader");
            html5QrcodeScanner.start(
                { facingMode: "environment" }, 
                { fps: 10, qrbox: 250 },
                (decodedText) => {
                    stopScanner();
                    processCheckoutRequest(decodedText);
                },
                (err) => { /* ignore */ }
            );
        }

        function stopScanner() {
            if(html5QrcodeScanner) {
                html5QrcodeScanner.stop().then(() => {
                    document.getElementById('btnStartScan').classList.remove('hidden');
                    document.getElementById('btnStopScan').classList.add('hidden');
                }).catch(err => console.log(err));
            }
        }

        function manualCheckout() {
            const val = document.getElementById('checkoutSearch').value;
            if(!val) return;
            processCheckoutRequest(val);
        }

        function processCheckoutRequest(visitorId) {
            Swal.fire({
                title: 'กำลังตรวจสอบ...',
                allowOutsideClick: false,
                didOpen: () => { Swal.showLoading() }
            });

            if (typeof google !== 'undefined' && google.script) {
                google.script.run
                    .withSuccessHandler(function(res) {
                        if(res.success) {
                            let statusHtml = res.status === 'Stamped' 
                                ? '<span style="color:green; font-weight:bold;">✅ ประทับตราแล้ว</span>' 
                                : '<span style="color:red; font-weight:bold;">❌ ยังไม่ประทับตรา</span><br><span style="font-size:0.8em">กรุณาตรวจสอบก่อนปล่อยรถออก</span>';
                                
                            Swal.fire({
                                title: 'บันทึกรถออกสำเร็จ',
                                html: `
                                    <b>ID:</b> ${res.id}<br>
                                    <b>ห้อง:</b> ${res.room}<br>
                                    <b>สถานะ:</b> ${statusHtml}<br><br>
                                    <b>เวลาออก:</b> ${res.timeOut}
                                `,
                                icon: res.status === 'Stamped' ? 'success' : 'warning',
                                confirmButtonText: 'ตกลง'
                            });
                            document.getElementById('checkoutSearch').value = '';
                        } else {
                            Swal.fire('ข้อผิดพลาด', res.message, 'error');
                        }
                    })
                    .processCheckout(visitorId);
            } else {
                setTimeout(() => {
                    Swal.fire({
                        title: 'บันทึกรถออกสำเร็จ (จำลอง)',
                        html: `
                            <b>ID:</b> ${visitorId}<br>
                            <b>ห้อง:</b> A101<br>
                            <b>สถานะ:</b> <span style="color:green; font-weight:bold;">✅ ประทับตราแล้ว</span><br><br>
                            <b>เวลาออก:</b> 16:30
                        `,
                        icon: 'success',
                        confirmButtonText: 'ตกลง'
                    });
                    document.getElementById('checkoutSearch').value = '';
                }, 1000);
            }
        }

        // ==========================================
        // Resident Registration QR
        // ==========================================
        function generateRegQR() {
            const room = document.getElementById('regRoomNo').value.trim();
            if(!room) {
                Swal.fire('แจ้งเตือน', 'กรุณาระบุเลขห้อง', 'warning');
                return;
            }
            
            // สร้าง URL ให้เปิดแอพ LINE ส่งข้อความหาบอท (line://nv/addOA ตามด้วยการส่งข้อความ หรือใช้ scheme line://msg/text/)
            // วิธีที่ชัวร์สุดเมื่อมีบอทอยู่แล้ว คือการส่งข้อความ
            const textToEncode = "line://msg/text/Reg:" + room; 
            
            document.getElementById('regQrContainer').classList.remove('hidden');
            document.getElementById('qrcodeReg').innerHTML = '';
            new QRCode(document.getElementById("qrcodeReg"), {
                text: textToEncode,
                width: 180,
                height: 180,
                colorDark : "#1e40af"
            });
        }
    </script>
</body>
</html>
