<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>My Guest - Visitor Management</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        pastel: { light: '#f0fdf4', DEFAULT: '#4ade80', dark: '#16a34a', text: '#14532d' }
                    }
                }
            }
        }
    </script>
    
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <!-- QRCode.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- HTML5 QRCode Scanner -->
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600&display=swap');
        body { font-family: 'Prompt', sans-serif; background-color: #f8fafc; }
        .view-section { display: none; }
        .view-section.active { display: block; }
        
        /* สไตล์สำหรับการพิมพ์สลิป (เครื่องพิมพ์ความร้อน) */
        @media print {
            body * { visibility: hidden; }
            #view-slip, #view-slip * { visibility: visible; }
            #view-slip { position: absolute; left: 0; top: 0; width: 100%; margin: 0; padding: 10px; box-shadow: none; border: none; }
            .no-print { display: none !important; }
            .print-area { width: 58mm; /* หรือ 80mm */ margin: 0 auto; text-align: center; font-size: 12px; }
        }
        
        /* Loader */
        .loader { border-top-color: #16a34a; -webkit-animation: spinner 1.5s linear infinite; animation: spinner 1.5s linear infinite; }
        @keyframes spinner { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="text-gray-800">

    <!-- Navbar -->
    <nav class="bg-pastel-dark text-white p-4 shadow-md sticky top-0 z-50 no-print">
        <div class="container mx-auto max-w-md flex justify-between items-center">
            <div class="text-xl font-bold flex items-center gap-2">
                <i class="fa-solid fa-leaf"></i> My Guest
            </div>
            <div class="flex gap-4 text-xl">
                <button onclick="switchView('view-form')" class="hover:text-pastel-light"><i class="fa-solid fa-house-chimney"></i></button>
                <button onclick="switchView('view-scan')" class="hover:text-pastel-light"><i class="fa-solid fa-qrcode"></i></button>
            </div>
        </div>
    </nav>

    <!-- Main Container -->
    <main class="container mx-auto max-w-md p-4 pb-24">
        
        <!-- Loading Overlay -->
        <div id="loading" class="fixed inset-0 bg-white bg-opacity-80 z-[100] flex flex-col justify-center items-center hidden">
            <div class="loader ease-linear rounded-full border-4 border-t-4 border-gray-200 h-12 w-12 mb-4"></div>
            <h2 class="text-center text-pastel-text text-xl font-semibold">กำลังประมวลผล...</h2>
        </div>

        <!-- ========================================== -->
        <!-- VIEW 1: ฟอร์มบันทึกผู้มาติดต่อ (สำหรับ รปภ.) -->
        <!-- ========================================== -->
        <section id="view-form" class="view-section active">
            <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
                <h2 class="text-xl font-bold text-pastel-dark mb-4 border-b pb-2">ลงทะเบียนผู้มาติดต่อ</h2>
                
                <form id="visitorForm" onsubmit="handleFormSubmit(event)">
                    <!-- วันที่/เวลา -->
                    <div class="grid grid-cols-2 gap-4 mb-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">วันที่</label>
                            <input type="text" id="inpDate" class="w-full bg-gray-100 border-none rounded-lg p-2 text-sm" readonly>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">เวลา</label>
                            <input type="text" id="inpTime" class="w-full bg-gray-100 border-none rounded-lg p-2 text-sm" readonly>
                        </div>
                    </div>

                    <!-- สถานที่ติดต่อ -->
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-2">สถานที่ติดต่อ</label>
                        <div class="grid grid-cols-2 gap-2">
                            <label class="border rounded-lg p-2 flex items-center gap-2 cursor-pointer hover:bg-pastel-light bg-pastel-light border-pastel-dark">
                                <input type="radio" name="contactType" value="room" class="accent-pastel-dark" checked onchange="toggleRoomInput()">
                                <span class="text-sm">บ้าน/ห้องชุด</span>
                            </label>
                            <label class="border rounded-lg p-2 flex items-center gap-2 cursor-pointer hover:bg-pastel-light">
                                <input type="radio" name="contactType" value="office" class="accent-pastel-dark" onchange="toggleRoomInput()">
                                <span class="text-sm">นิติบุคคล</span>
                            </label>
                            <label class="border rounded-lg p-2 flex items-center gap-2 cursor-pointer hover:bg-pastel-light">
                                <input type="radio" name="contactType" value="visit" class="accent-pastel-dark" onchange="toggleRoomInput()">
                                <span class="text-sm">เยี่ยมชมโครงการ</span>
                            </label>
                            <label class="border rounded-lg p-2 flex items-center gap-2 cursor-pointer hover:bg-pastel-light">
                                <input type="radio" name="contactType" value="other" class="accent-pastel-dark" onchange="toggleRoomInput()">
                                <span class="text-sm">อื่นๆ</span>
                            </label>
                        </div>
                    </div>

                    <!-- Input เลขห้อง (แสดงเมื่อเลือกบ้าน/ห้องชุด) -->
                    <div id="roomInputContainer" class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-1">เลขบ้าน / ห้องชุด</label>
                        <input type="text" id="inpRoom" placeholder="พิมพ์เพื่อค้นหา..." list="roomList" class="w-full border rounded-lg p-2.5 focus:ring-2 focus:ring-pastel-dark outline-none transition" required>
                        <datalist id="roomList">
                            <!-- จะถูกเติมด้วย JS -->
                        </datalist>
                    </div>

                    <!-- ทะเบียนรถ (กล้อง) -->
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-1">ถ่ายรูปทะเบียนรถ / บัตร</label>
                        <div class="relative border-2 border-dashed border-gray-300 rounded-xl p-4 text-center hover:bg-gray-50 transition cursor-pointer" onclick="document.getElementById('inpFile').click()">
                            <input type="file" id="inpFile" accept="image/*" capture="environment" class="hidden" onchange="previewImage(event)">
                            <div id="imagePlaceholder" class="text-gray-400">
                                <i class="fa-solid fa-camera text-3xl mb-2"></i>
                                <p class="text-sm">แตะเพื่อถ่ายรูป</p>
                            </div>
                            <img id="imagePreview" class="hidden mx-auto rounded-lg max-h-48 object-cover">
                        </div>
                    </div>

                    <!-- วัตถุประสงค์ -->
                    <div class="mb-6">
                        <label class="block text-sm font-medium text-gray-700 mb-1">วัตถุประสงค์</label>
                        <select id="inpPurpose" class="w-full border rounded-lg p-2.5 focus:ring-2 focus:ring-pastel-dark outline-none bg-white">
                            <option value="ญาติ/เพื่อน">ญาติ/เพื่อน</option>
                            <option value="นัดหมายไว้">นัดหมายไว้</option>
                            <option value="ส่งของ/อาหาร">ส่งของ/อาหาร</option>
                            <option value="ช่างตกแต่ง/ต่อเติม">ช่างตกแต่ง/ต่อเติม</option>
                            <option value="อื่นๆ">อื่นๆ</option>
                        </select>
                    </div>

                    <!-- ปุ่ม Submit -->
                    <button type="submit" class="w-full bg-pastel-dark hover:bg-green-700 text-white font-bold py-3 rounded-xl shadow-lg transition flex justify-center items-center gap-2">
                        <i class="fa-solid fa-save"></i> บันทึกข้อมูล
                    </button>
                </form>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- VIEW 2: หน้าสลิป และ QR Code -->
        <!-- ========================================== -->
        <section id="view-slip" class="view-section">
            <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 print-area relative">
                
                <div class="text-center mb-4">
                    <i class="fa-solid fa-leaf text-3xl text-pastel-dark mb-2"></i>
                    <h2 class="text-xl font-bold">My Guest</h2>
                    <p class="text-sm text-gray-500">บัตรผู้มาติดต่อ / Visitor Pass</p>
                </div>

                <div class="border-t border-b border-dashed border-gray-300 py-4 my-4 text-left space-y-2 text-sm">
                    <div class="flex justify-between"><span class="text-gray-500">รหัส:</span> <span id="slipId" class="font-semibold"></span></div>
                    <div class="flex justify-between"><span class="text-gray-500">วันที่:</span> <span id="slipDate" class="font-semibold"></span></div>
                    <div class="flex justify-between"><span class="text-gray-500">เวลาเข้า:</span> <span id="slipTimeIn" class="font-semibold"></span></div>
                    <div class="flex justify-between"><span class="text-gray-500">สถานที่:</span> <span id="slipTarget" class="font-semibold"></span></div>
                    <div class="flex justify-between"><span class="text-gray-500">จุดประสงค์:</span> <span id="slipPurpose" class="font-semibold"></span></div>
                </div>

                <div class="flex justify-center mb-4">
                    <div id="qrcode" class="p-2 bg-white border rounded-lg"></div>
                </div>
                
                <p class="text-xs text-center text-gray-500 mb-6">
                    โปรดให้เจ้าของบ้านประทับตราหรือสแกน QR Code<br>เพื่อยืนยันก่อนนำรถออกจากโครงการ
                </p>

                <!-- ปุ่ม Print -->
                <button onclick="window.print()" class="no-print w-full bg-gray-800 hover:bg-black text-white font-bold py-3 rounded-xl shadow transition flex justify-center items-center gap-2 mb-3">
                    <i class="fa-solid fa-print"></i> พิมพ์บัตร (Print)
                </button>
                <button onclick="resetForm()" class="no-print w-full bg-pastel-light text-pastel-text font-bold py-3 rounded-xl transition flex justify-center items-center gap-2">
                    <i class="fa-solid fa-arrow-left"></i> กลับหน้าแรก
                </button>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- VIEW 3: หน้า Scan Check-out (สำหรับ รปภ.) -->
        <!-- ========================================== -->
        <section id="view-scan" class="view-section">
            <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 text-center">
                <h2 class="text-xl font-bold text-pastel-dark mb-4">สแกนบัตรขาออก</h2>
                <div id="qr-reader" class="w-full bg-gray-100 rounded-xl overflow-hidden mb-4 border border-gray-300"></div>
                
                <div class="relative my-4">
                    <div class="absolute inset-0 flex items-center"><div class="w-full border-t border-gray-300"></div></div>
                    <div class="relative flex justify-center text-sm"><span class="px-2 bg-white text-gray-500">หรือ</span></div>
                </div>

                <form onsubmit="manualSearch(event)" class="flex gap-2">
                    <input type="text" id="inpSearch" placeholder="กรอกรหัส (เช่น V230...)" class="flex-1 border rounded-lg p-2 outline-none focus:border-pastel-dark">
                    <button type="submit" class="bg-pastel-dark text-white px-4 rounded-lg"><i class="fa-solid fa-search"></i></button>
                </form>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- VIEW 4: หน้าสรุปผลขาออก (Check-out Detail) -->
        <!-- ========================================== -->
        <section id="view-checkout" class="view-section">
            <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
                <h2 class="text-xl font-bold text-pastel-dark mb-4 border-b pb-2">รายละเอียดขาออก</h2>
                
                <div id="checkoutStatusBox" class="p-4 rounded-xl text-center mb-4 font-bold text-lg">
                    <!-- Status here -->
                </div>

                <div class="space-y-3 text-sm mb-6">
                    <div class="flex justify-between border-b pb-1"><span class="text-gray-500">รหัส:</span> <span id="coId" class="font-semibold"></span></div>
                    <div class="flex justify-between border-b pb-1"><span class="text-gray-500">สถานที่:</span> <span id="coTarget" class="font-semibold"></span></div>
                    <div class="flex justify-between border-b pb-1"><span class="text-gray-500">เวลาเข้า:</span> <span id="coTimeIn" class="font-semibold"></span></div>
                    <div>
                        <span class="text-gray-500 block mb-1">รูปรถ/บัตร:</span>
                        <img id="coImage" src="" class="w-full h-32 object-cover rounded-lg bg-gray-100">
                    </div>
                </div>

                <button id="btnConfirmCheckout" onclick="processCheckout()" class="w-full bg-pastel-dark hover:bg-green-700 text-white font-bold py-3 rounded-xl shadow-lg transition flex justify-center items-center gap-2 mb-3">
                    <i class="fa-solid fa-sign-out-alt"></i> บันทึกรถออก (Check-Out)
                </button>
                <button onclick="switchView('view-scan')" class="w-full bg-gray-200 text-gray-700 font-bold py-3 rounded-xl transition flex justify-center items-center gap-2">
                    ยกเลิก
                </button>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- VIEW 5: หน้าประทับตรา (สำหรับลูกบ้านสแกน QR) -->
        <!-- ========================================== -->
        <section id="view-resident-stamp" class="view-section">
            <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 text-center">
                <div class="w-16 h-16 bg-pastel-light text-pastel-dark rounded-full flex items-center justify-center mx-auto mb-4 text-3xl">
                    <i class="fa-solid fa-home"></i>
                </div>
                <h2 class="text-xl font-bold text-gray-800 mb-2">ยืนยันผู้มาติดต่อ</h2>
                <p class="text-sm text-gray-500 mb-6">ห้อง: <span id="rsRoom" class="font-bold text-pastel-dark"></span></p>

                <div class="bg-gray-50 rounded-xl p-4 text-left text-sm space-y-2 mb-6 border">
                    <div class="flex justify-between"><span class="text-gray-500">เวลาเข้า:</span> <span id="rsTimeIn"></span></div>
                    <div class="flex justify-between"><span class="text-gray-500">จุดประสงค์:</span> <span id="rsPurpose"></span></div>
                </div>

                <div id="rsActionButtons">
                    <button onclick="residentStampAction('stamp')" class="w-full bg-pastel-dark hover:bg-green-700 text-white font-bold py-3 rounded-xl shadow-lg transition flex justify-center items-center gap-2 mb-3">
                        <i class="fa-solid fa-check-circle"></i> ประทับตราอนุมัติ
                    </button>
                    <button onclick="residentStampAction('reject')" class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-3 rounded-xl shadow transition flex justify-center items-center gap-2">
                        <i class="fa-solid fa-times-circle"></i> ปฏิเสธการเข้าพบ
                    </button>
                </div>
                <div id="rsSuccessMsg" class="hidden text-green-600 font-bold p-4 bg-green-50 rounded-xl">
                    ดำเนินการเรียบร้อยแล้ว ปิดหน้านี้ได้ทันที
                </div>
            </div>
        </section>

    </main>

    <script>
        // Global State
        let currentBase64Image = null;
        let activeVisitorId = null;
        let html5QrcodeScanner = null;

        // Initialization
        window.onload = () => {
            updateDateTime();
            setInterval(updateDateTime, 60000); // อัปเดตเวลาทุก 1 นาที
            
            // จำลองการโหลดข้อมูลห้องพัก (ในการใช้จริงจะเรียก google.script.run)
            mockLoadResidents();

            // เช็ค URL Params ว่าเป็นการสแกน QR ของลูกบ้านหรือไม่
            checkUrlParams();
        };

        // Router & View Switcher
        function switchView(viewId) {
            document.querySelectorAll('.view-section').forEach(el => el.classList.remove('active'));
            document.getElementById(viewId).classList.add('active');

            // จัดการ Scanner 
            if (viewId === 'view-scan') {
                startScanner();
            } else if (html5QrcodeScanner) {
                try { html5QrcodeScanner.clear(); } catch(e){}
            }
        }

        // อัปเดตวันที่และเวลา
        function updateDateTime() {
            const now = new Date();
            const dateStr = now.toLocaleDateString('th-TH', { day: '2-digit', month: '2-digit', year: 'numeric' });
            const timeStr = now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
            document.getElementById('inpDate').value = dateStr;
            document.getElementById('inpTime').value = timeStr;
        }

        // จัดการฟอร์มสถานที่ติดต่อ (ซ่อน/แสดง input เลขห้อง)
        function toggleRoomInput() {
            const type = document.querySelector('input[name="contactType"]:checked').value;
            const roomContainer = document.getElementById('roomInputContainer');
            const roomInput = document.getElementById('inpRoom');
            const labels = document.querySelectorAll('input[name="contactType"]');
            
            // ปรับสีปุ่ม Radio
            labels.forEach(radio => {
                if(radio.checked) {
                    radio.parentElement.classList.add('bg-pastel-light', 'border-pastel-dark');
                } else {
                    radio.parentElement.classList.remove('bg-pastel-light', 'border-pastel-dark');
                }
            });

            if (type === 'room') {
                roomContainer.style.display = 'block';
                roomInput.required = true;
            } else {
                roomContainer.style.display = 'none';
                roomInput.required = false;
                roomInput.value = '';
            }
        }

        // แสดงพรีวิวรูปภาพ
        function previewImage(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    currentBase64Image = e.target.result;
                    document.getElementById('imagePlaceholder').classList.add('hidden');
                    const imgPreview = document.getElementById('imagePreview');
                    imgPreview.src = currentBase64Image;
                    imgPreview.classList.remove('hidden');
                }
                reader.readAsDataURL(file);
            }
        }

        // โหลดข้อมูลห้องพักจำลอง (GAS Simulation)
        function mockLoadResidents() {
            // โค้ดจริง: google.script.run.withSuccessHandler(populateDatalist).getResidentsList();
            const mockData = ["A101", "A102", "B201", "B202", "99/1", "99/2"];
            populateDatalist(mockData);
        }

        function populateDatalist(dataArray) {
            const datalist = document.getElementById('roomList');
            datalist.innerHTML = '';
            dataArray.forEach(room => {
                let option = document.createElement('option');
                option.value = room;
                datalist.appendChild(option);
            });
        }

        // จัดการเมื่อกด "บันทึกข้อมูล"
        function handleFormSubmit(e) {
            e.preventDefault();
            
            const contactType = document.querySelector('input[name="contactType"]:checked').value;
            const targetRoom = contactType === 'room' ? document.getElementById('inpRoom').value : 
                              (contactType === 'office' ? 'นิติบุคคล' : 
                              (contactType === 'visit' ? 'เยี่ยมชม' : 'อื่นๆ'));
            const purpose = document.getElementById('inpPurpose').value;

            if (!currentBase64Image) {
                Swal.fire({ icon: 'warning', title: 'กรุณาถ่ายรูป', text: 'ต้องมีภาพรถหรือบัตรประชาชน' });
                return;
            }

            const formData = {
                contactType: contactType,
                targetRoom: targetRoom,
                purpose: purpose,
                imageBase64: currentBase64Image
            };

            showLoading(true);

            // โค้ดจริง: google.script.run.withSuccessHandler(showSlip).saveVisitor(formData);
            
            // Mock System
            setTimeout(() => {
                showLoading(false);
                const mockResult = {
                    success: true,
                    id: "V" + new Date().getTime().toString().slice(-6),
                    date: document.getElementById('inpDate').value,
                    timeIn: document.getElementById('inpTime').value,
                    photoUrl: currentBase64Image,
                    target: targetRoom,
                    purpose: purpose
                };
                showSlip(mockResult);
            }, 1500);
        }

        // แสดงหน้า Slip และสร้าง QR Code
        function showSlip(data) {
            if (!data.success) {
                Swal.fire('Error', 'ไม่สามารถบันทึกข้อมูลได้', 'error');
                return;
            }

            document.getElementById('slipId').innerText = data.id;
            document.getElementById('slipDate').innerText = data.date;
            document.getElementById('slipTimeIn').innerText = data.timeIn;
            document.getElementById('slipTarget').innerText = data.target || document.getElementById('inpRoom').value || "อื่นๆ";
            document.getElementById('slipPurpose').innerText = data.purpose;

            // สร้าง QR Code (URL ชี้กลับมาที่ Web App)
            const webAppUrl = "https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec"; // เปลี่ยนเป็น URL ของคุณ
            const qrUrl = `${webAppUrl}?action=stamp&id=${data.id}`;
            
            document.getElementById('qrcode').innerHTML = '';
            new QRCode(document.getElementById("qrcode"), {
                text: qrUrl, width: 128, height: 128,
                colorDark : "#000000", colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });

            switchView('view-slip');
            
            Swal.fire({
                icon: 'success',
                title: 'บันทึกสำเร็จ',
                showConfirmButton: false,
                timer: 1500
            });
        }

        function resetForm() {
            document.getElementById('visitorForm').reset();
            document.getElementById('imagePlaceholder').classList.remove('hidden');
            document.getElementById('imagePreview').classList.add('hidden');
            document.getElementById('imagePreview').src = '';
            currentBase64Image = null;
            toggleRoomInput();
            updateDateTime();
            switchView('view-form');
        }

        // ==========================================
        // SCANNER & CHECKOUT LOGIC
        // ==========================================
        
        function startScanner() {
            if (html5QrcodeScanner == null) {
                html5QrcodeScanner = new Html5QrcodeScanner("qr-reader", { fps: 10, qrbox: {width: 250, height: 250} }, false);
            }
            html5QrcodeScanner.render(onScanSuccess, onScanFailure);
        }

        function onScanSuccess(decodedText, decodedResult) {
            // หยุดแสกน
            html5QrcodeScanner.clear();
            
            // ดึง ID ออกมาจาก URL (ถ้าสแกนจาก QR ของระบบ)
            let visitorId = decodedText;
            if(decodedText.includes('?action=stamp&id=')) {
                const urlParams = new URLSearchParams(decodedText.split('?')[1]);
                visitorId = urlParams.get('id');
            }

            fetchVisitorForCheckout(visitorId);
        }

        function onScanFailure(error) { /* เงียบไว้ รอจนกว่าจะเจอ */ }

        function manualSearch(e) {
            e.preventDefault();
            const val = document.getElementById('inpSearch').value;
            if(val) fetchVisitorForCheckout(val);
        }

        function fetchVisitorForCheckout(id) {
            showLoading(true);
            // โค้ดจริง: google.script.run.withSuccessHandler(showCheckoutDetail).getVisitorData(id);
            
            // Mock
            setTimeout(() => {
                showLoading(false);
                // จำลองว่าเจอบ้างไม่เจอบ้าง
                const mockData = {
                    id: id, TargetRoom: 'A101', TimeIn: '08:30', 
                    LicensePlatePhoto_URL: 'https://via.placeholder.com/300x200?text=Car', 
                    Status: Math.random() > 0.5 ? 'Stamped' : 'Pending'
                };
                showCheckoutDetail(mockData);
            }, 1000);
        }

        function showCheckoutDetail(data) {
            if (!data) {
                Swal.fire('ไม่พบข้อมูล', 'รหัสไม่ถูกต้อง หรือไม่มีในระบบ', 'error');
                switchView('view-scan');
                return;
            }

            activeVisitorId = data.id || data.ID;
            document.getElementById('coId').innerText = activeVisitorId;
            document.getElementById('coTarget').innerText = data.TargetRoom;
            document.getElementById('coTimeIn').innerText = data.TimeIn;
            if(data.LicensePlatePhoto_URL) document.getElementById('coImage').src = data.LicensePlatePhoto_URL;

            const statusBox = document.getElementById('checkoutStatusBox');
            const btnConfirm = document.getElementById('btnConfirmCheckout');
            
            statusBox.className = "p-4 rounded-xl text-center mb-4 font-bold text-lg"; // reset
            
            if (data.Status === 'Stamped') {
                statusBox.classList.add('bg-green-100', 'text-green-800');
                statusBox.innerHTML = '<i class="fa-solid fa-check-circle text-2xl mb-1"></i><br>ประทับตราแล้ว (อนุญาตให้ออก)';
                btnConfirm.disabled = false;
                btnConfirm.classList.remove('opacity-50', 'cursor-not-allowed');
            } else if (data.Status === 'Checked-Out') {
                 statusBox.classList.add('bg-gray-100', 'text-gray-800');
                statusBox.innerHTML = '<i class="fa-solid fa-sign-out-alt text-2xl mb-1"></i><br>รถคันนี้ออกไปแล้ว';
                btnConfirm.disabled = true;
                btnConfirm.classList.add('opacity-50', 'cursor-not-allowed');
            } else {
                statusBox.classList.add('bg-yellow-100', 'text-yellow-800');
                statusBox.innerHTML = '<i class="fa-solid fa-exclamation-triangle text-2xl mb-1"></i><br>ยังไม่ประทับตรา!';
                // รปภ สามารถกดออกได้ไหม? สมมติว่าให้กดได้แต่มี Warning
                btnConfirm.disabled = false;
                btnConfirm.classList.remove('opacity-50', 'cursor-not-allowed');
            }

            switchView('view-checkout');
        }

        function processCheckout() {
            if(!activeVisitorId) return;
            
            Swal.fire({
                title: 'ยืนยันบันทึกรถออก?',
                icon: 'question',
                showCancelButton: true,
                confirmButtonColor: '#16a34a',
                cancelButtonColor: '#d33',
                confirmButtonText: 'ยืนยัน'
            }).then((result) => {
                if (result.isConfirmed) {
                    showLoading(true);
                    // โค้ดจริง: google.script.run.withSuccessHandler((res)=>{...}).updateVisitorStatus(activeVisitorId, 'checkout');
                    
                    // Mock
                    setTimeout(() => {
                        showLoading(false);
                        Swal.fire('สำเร็จ', 'บันทึกรถออกเรียบร้อย', 'success');
                        switchView('view-scan');
                    }, 1000);
                }
            });
        }

        // ==========================================
        // URL PARAMETERS & RESIDENT ACTION LOGIC
        // ==========================================
        function checkUrlParams() {
            // โค้ดจริงจะรับค่ามาจาก template.urlParams ของ GAS
            // สมมติว่ารันแบบ Mock ดึงจาก window.location.search
            const urlParams = new URLSearchParams(window.location.search);
            const action = urlParams.get('action');
            const id = urlParams.get('id');

            if (action === 'stamp' && id) {
                activeVisitorId = id;
                document.querySelector('nav').style.display = 'none'; // ซ่อนเมนู รปภ.
                
                // จำลองดึงข้อมูล
                document.getElementById('rsRoom').innerText = "A101";
                document.getElementById('rsTimeIn').innerText = "10:30";
                document.getElementById('rsPurpose').innerText = "ส่งของ";
                
                switchView('view-resident-stamp');
            }
        }

        function residentStampAction(actionType) {
            showLoading(true);
            // โค้ดจริง: google.script.run.withSuccessHandler(onResidentStamped).updateVisitorStatus(activeVisitorId, actionType);
            
            // Mock
            setTimeout(() => {
                showLoading(false);
                document.getElementById('rsActionButtons').style.display = 'none';
                const msgBox = document.getElementById('rsSuccessMsg');
                msgBox.style.display = 'block';
                if(actionType === 'reject') {
                    msgBox.classList.replace('bg-green-50', 'bg-red-50');
                    msgBox.classList.replace('text-green-600', 'text-red-600');
                    msgBox.innerText = 'บันทึก "ปฏิเสธ" เรียบร้อยแล้ว';
                }
            }, 1000);
        }

        function showLoading(show) {
            document.getElementById('loading').style.display = show ? 'flex' : 'none';
        }

    </script>
</body>
</html>
