<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MY GUEST - Visitor System</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <!-- QRCode.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- LINE LIFF (เพิ่มใหม่) -->
    <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script>
        // ป้องกัน Error กรณีที่ Tailwind โหลดไม่ขึ้น
        try {
            tailwind.config = {
                theme: {
                    extend: {
                        colors: {
                            pastel: {
                                light: '#e0f2fe',
                                green: '#d1fae5', // เขียวพาสเทลอ่อนสุด
                                primary: '#34d399', // เขียวหลัก
                                dark: '#059669' // เขียวเข้ม
                            }
                        }
                    }
                }
            };
        } catch(e) { console.warn("Tailwind config error"); }
    </script>
    <style>
        body { font-family: 'Sarabun', sans-serif; background-color: #f0fdf4; }
        
        /* Fallback CSS: สำคัญมาก ป้องกันกรณีโหลด Tailwind ไม่ขึ้น แท็บเมนูจะได้ยังทำงานได้ */
        .hidden { display: none !important; }
        
        /* สไตล์สำหรับซ่อน/แสดงเนื้อหาตอน Print สลิปความร้อน */
        @media print {
            body * { visibility: hidden; }
            #print-slip-area, #print-slip-area * { visibility: visible; }
            #print-slip-area {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
                max-width: 80mm; /* ขนาดกระดาษสลิปความร้อน */
                margin: 0 auto;
                padding: 10px;
                color: black !important;
            }
            .no-print { display: none !important; }
        }
        
        /* เพิ่มมิติให้กับ Input */
        input[type="text"], select {
            box-shadow: inset 0 2px 4px 0 rgba(0, 0, 0, 0.03);
            transition: all 0.2s;
        }
        input[type="text"]:focus, select:focus {
            box-shadow: 0 0 0 3px rgba(52, 211, 153, 0.3);
            outline: none;
        }
    </style>
</head>
<body class="text-gray-800 pb-20">

    <!-- Navbar -->
    <nav class="bg-pastel-primary shadow-lg rounded-b-2xl mb-6">
        <div class="max-w-md mx-auto px-4 py-4 flex justify-between items-center text-white">
            <h1 class="text-2xl font-bold tracking-wider drop-shadow-md"><i class="fa-solid fa-leaf mr-2"></i>MY GUEST</h1>
            <div id="clock" class="text-sm font-medium bg-white/20 px-3 py-1 rounded-full shadow-inner">--:--:--</div>
        </div>
    </nav>

    <!-- Main Container -->
    <div class="max-w-md mx-auto px-4" id="app-container">
        
        <!-- Tab Navigation -->
        <div class="flex bg-white rounded-xl shadow-md mb-6 p-1 no-print border border-gray-100">
            <button onclick="switchTab('checkin')" id="tab-checkin" class="flex-1 py-2.5 text-center rounded-lg bg-pastel-primary text-white font-semibold shadow transition-all duration-200">
                <i class="fa-solid fa-sign-in-alt mr-1"></i> รถเข้า
            </button>
            <button onclick="switchTab('checkout')" id="tab-checkout" class="flex-1 py-2.5 text-center rounded-lg text-gray-500 font-semibold transition-all duration-200 hover:bg-gray-50">
                <i class="fa-solid fa-sign-out-alt mr-1"></i> รถออก
            </button>
            <!-- เพิ่มปุ่มสำหรับหน้าสร้าง QR ลูกบ้าน -->
            <button onclick="switchTab('admin')" id="tab-admin" class="flex-1 py-2.5 text-center rounded-lg text-gray-500 font-semibold transition-all duration-200 hover:bg-gray-50">
                <i class="fa-solid fa-qrcode mr-1"></i> QR ลูกบ้าน
            </button>
        </div>

        <!-- ======================= CHECK-IN FORM ======================= -->
        <div id="view-checkin" class="bg-white p-6 rounded-2xl shadow-xl border-t-4 border-t-pastel-primary">
            <h2 class="text-xl font-bold text-pastel-dark mb-5 border-b pb-2 flex items-center">
                <i class="fa-solid fa-pen-to-square mr-2"></i> บันทึกผู้มาติดต่อ
            </h2>
            <form id="checkinForm" onsubmit="handleCheckin(event)">
                
                <!-- Date/Time Display (Auto - Not editable) -->
                <div class="flex items-center justify-between bg-pastel-green/30 p-3 rounded-xl mb-6 border border-pastel-green shadow-sm">
                    <div class="flex items-center text-pastel-dark font-medium text-sm">
                        <i class="fa-regular fa-calendar-alt mr-2 text-lg"></i>
                        <span id="dateDisplay">--/--/----</span>
                    </div>
                    <div class="w-px h-6 bg-pastel-primary/30"></div>
                    <div class="flex items-center text-pastel-dark font-medium text-sm">
                        <i class="fa-regular fa-clock mr-2 text-lg"></i>
                        <span id="timeDisplay">--:--</span>
                    </div>
                </div>

                <!-- Contact Type (Modern Pills) -->
                <div class="mb-5">
                    <label class="block text-sm font-bold text-gray-700 mb-2">สถานที่ติดต่อ <span class="text-red-500">*</span></label>
                    <div class="grid grid-cols-2 gap-3 text-sm">
                        <label class="cursor-pointer relative">
                            <input type="radio" name="contactType" value="room" class="peer sr-only" onchange="toggleContactFields()" required>
                            <div class="text-center p-3 rounded-xl border-2 border-gray-100 bg-gray-50 text-gray-600 peer-checked:bg-pastel-primary peer-checked:text-white peer-checked:border-pastel-primary peer-checked:shadow-md transition-all duration-200">
                                <i class="fa-solid fa-door-open mb-1 block text-lg"></i> บ้านเลขที่/ห้องชุด
                            </div>
                        </label>
                        <label class="cursor-pointer relative">
                            <input type="radio" name="contactType" value="office" class="peer sr-only" onchange="toggleContactFields()">
                            <div class="text-center p-3 rounded-xl border-2 border-gray-100 bg-gray-50 text-gray-600 peer-checked:bg-pastel-primary peer-checked:text-white peer-checked:border-pastel-primary peer-checked:shadow-md transition-all duration-200">
                                <i class="fa-solid fa-building mb-1 block text-lg"></i> นิติบุคคลฯ
                            </div>
                        </label>
                        <label class="cursor-pointer relative">
                            <input type="radio" name="contactType" value="visit" class="peer sr-only" onchange="toggleContactFields()">
                            <div class="text-center p-3 rounded-xl border-2 border-gray-100 bg-gray-50 text-gray-600 peer-checked:bg-pastel-primary peer-checked:text-white peer-checked:border-pastel-primary peer-checked:shadow-md transition-all duration-200">
                                <i class="fa-solid fa-eye mb-1 block text-lg"></i> เยี่ยมชมโครงการ
                            </div>
                        </label>
                        <label class="cursor-pointer relative">
                            <input type="radio" name="contactType" value="other" class="peer sr-only" onchange="toggleContactFields()">
                            <div class="text-center p-3 rounded-xl border-2 border-gray-100 bg-gray-50 text-gray-600 peer-checked:bg-pastel-primary peer-checked:text-white peer-checked:border-pastel-primary peer-checked:shadow-md transition-all duration-200">
                                <i class="fa-solid fa-ellipsis-h mb-1 block text-lg"></i> อื่นๆ
                            </div>
                        </label>
                    </div>
                </div>

                <!-- Dynamic Fields based on Contact Type -->
                <div id="roomInputContainer" class="mb-5 hidden">
                    <label class="block text-sm font-bold text-gray-700 mb-1">ระบุเลขห้อง / บ้านเลขที่ <span class="text-red-500">*</span></label>
                    <input type="text" id="targetRoom" list="roomList" placeholder="กำลังโหลดข้อมูล..." class="w-full border-gray-200 rounded-xl p-3 border focus:outline-none" autocomplete="off">
                    <datalist id="roomList">
                        <!-- ตัวเลือกห้องจะถูกดึงมาใส่ตรงนี้โดย JavaScript -->
                    </datalist>
                </div>
                <div id="otherInputContainer" class="mb-5 hidden">
                    <label class="block text-sm font-bold text-gray-700 mb-1">ระบุรายละเอียดสถานที่ <span class="text-red-500">*</span></label>
                    <input type="text" id="targetOther" placeholder="โปรดระบุ" class="w-full border-gray-200 rounded-xl p-3 border focus:outline-none">
                </div>

                <!-- Purpose -->
                <div class="mb-5">
                    <label class="block text-sm font-bold text-gray-700 mb-1">วัตถุประสงค์ <span class="text-red-500">*</span></label>
                    <select id="purpose" class="w-full border-gray-200 rounded-xl p-3 border bg-white focus:outline-none" required>
                        <option value="">-- เลือกวัตถุประสงค์ --</option>
                        <option value="ญาติ/เพื่อน">ญาติ / เพื่อน</option>
                        <option value="นัดหมายไว้">นัดหมายไว้ล่วงหน้า</option>
                        <option value="ส่งของ/อาหาร">ส่งของ / อาหาร (Rider)</option>
                        <option value="ช่าง/ตกแต่งต่อเติม">ช่าง / ตกแต่งต่อเติม</option>
                        <option value="อื่นๆ">อื่นๆ</option>
                    </select>
                </div>

                <!-- Divider -->
                <hr class="my-5 border-gray-100">

                <!-- Vehicle Type -->
                <div class="mb-5">
                    <label class="block text-sm font-bold text-gray-700 mb-2">ประเภทรถ <span class="text-red-500">*</span></label>
                    <div class="grid grid-cols-2 gap-3 text-sm">
                        <label class="cursor-pointer relative">
                            <!-- ใช้ peer sr-only เพื่อซ่อนปุ่มวงกลม แต่ยัง focus/submit ได้ปกติ -->
                            <input type="radio" name="vehicleType" value="รถยนต์" class="peer sr-only" required>
                            <div class="text-center p-3 rounded-xl border-2 border-gray-100 bg-gray-50 text-gray-600 peer-checked:bg-pastel-primary peer-checked:text-white peer-checked:border-pastel-primary peer-checked:shadow-md transition-all duration-200">
                                <i class="fa-solid fa-car mr-2"></i> รถยนต์
                            </div>
                        </label>
                        <label class="cursor-pointer relative">
                            <input type="radio" name="vehicleType" value="รถจักรยานยนต์" class="peer sr-only" required>
                            <div class="text-center p-3 rounded-xl border-2 border-gray-100 bg-gray-50 text-gray-600 peer-checked:bg-pastel-primary peer-checked:text-white peer-checked:border-pastel-primary peer-checked:shadow-md transition-all duration-200">
                                <i class="fa-solid fa-motorcycle mr-2"></i> รถจักรยานยนต์
                            </div>
                        </label>
                    </div>
                </div>

                <!-- License Plate Input -->
                <div class="mb-5">
                    <label class="block text-sm font-bold text-gray-700 mb-1">เลขทะเบียนรถ <span class="text-red-500">*</span></label>
                    <input type="text" id="licensePlate" placeholder="เช่น กข 1234 กทม." class="w-full border-gray-200 rounded-xl p-3 border focus:outline-none" required>
                </div>

                <!-- Camera / License Plate Image -->
                <div class="mb-8">
                    <label class="block text-sm font-bold text-gray-700 mb-2">ภาพถ่ายทะเบียนรถ / ผู้มาติดต่อ <span class="text-red-500">*</span></label>
                    <div class="border-2 border-dashed border-pastel-primary bg-pastel-green/10 hover:bg-pastel-green/30 rounded-2xl p-6 text-center cursor-pointer transition-colors duration-200" onclick="document.getElementById('cameraInput').click()">
                        <!-- เอา required ออกจาก input file ที่ซ่อนไว้ ป้องกันเบราว์เซอร์แจ้งเตือนค้างเวลาส่งฟอร์ม -->
                        <input type="file" id="cameraInput" accept="image/*" capture="environment" class="hidden" onchange="previewImage(event)">
                        <div class="bg-white w-14 h-14 rounded-full flex items-center justify-center mx-auto mb-3 shadow-sm text-pastel-primary border border-pastel-green">
                            <i class="fa-solid fa-camera text-2xl"></i>
                        </div>
                        <p class="text-sm font-semibold text-pastel-dark" id="cameraText">แตะเพื่อเปิดกล้องถ่ายรูป</p>
                    </div>
                    <img id="imagePreview" class="mt-4 rounded-xl w-full hidden object-cover max-h-56 border-2 border-gray-100 shadow-sm">
                </div>

                <!-- Submit Button -->
                <button type="submit" class="w-full bg-pastel-primary hover:bg-pastel-dark text-white font-bold py-3.5 rounded-xl shadow-lg hover:shadow-xl transition-all duration-200 flex justify-center items-center text-lg transform hover:-translate-y-0.5">
                    <i class="fa-solid fa-save mr-2"></i> บันทึกข้อมูลเข้า
                </button>
            </form>
        </div>

        <!-- ======================= SLIP / PRINT VIEW ======================= -->
        <div id="view-slip" class="hidden">
            <div id="print-slip-area" class="bg-white p-6 rounded-2xl shadow-xl border-t-8 border-pastel-primary mb-5 text-center mx-auto max-w-sm">
                <h1 class="text-2xl font-bold text-gray-800"><i class="fa-solid fa-leaf text-pastel-primary"></i> MY GUEST</h1>
                <p class="text-xs text-gray-500 mb-4 border-b pb-3">บัตรผ่านผู้มาติดต่อ (Visitor Pass)</p>
                
                <div class="text-left text-sm space-y-2 mb-5 bg-gray-50 p-3 rounded-lg border border-gray-100">
                    <p><strong>รหัส (ID):</strong> <span id="slip-id" class="text-pastel-dark font-bold">--</span></p>
                    <p><strong>วันที่:</strong> <span id="slip-date">--</span></p>
                    <p><strong>เวลาเข้า:</strong> <span id="slip-time">--</span></p>
                    <p><strong>สถานที่ติดต่อ:</strong> <span id="slip-target" class="font-bold text-black">--</span></p>
                    <p><strong>วัตถุประสงค์:</strong> <span id="slip-purpose">--</span></p>
                    <p><strong>ประเภทรถ:</strong> <span id="slip-vehicle">--</span></p>
                    <p><strong>ทะเบียนรถ:</strong> <span id="slip-license" class="font-bold text-black">--</span></p>
                </div>

                <div class="flex justify-center mb-3">
                    <div id="qrcode" class="p-3 bg-white border-2 border-gray-100 rounded-xl shadow-sm"></div>
                </div>
                <p class="text-[10px] text-gray-400">สแกนเพื่อตรวจสอบสถานะ / ประทับตราออก</p>
                
                <div class="mt-5 pt-5 border-t-2 border-dashed border-gray-200 text-left">
                    <p class="text-xs text-gray-500 mb-8 text-center">ลายมือชื่อผู้ได้รับการติดต่อ / ประทับตรา</p>
                    <p class="text-xs text-center text-gray-400">_________________________________</p>
                </div>
            </div>
            
            <div class="flex gap-3 no-print">
                <button onclick="window.print()" class="flex-1 bg-gray-800 hover:bg-black text-white py-3.5 rounded-xl font-bold shadow-md flex justify-center items-center transition-colors">
                    <i class="fa-solid fa-print mr-2"></i> พิมพ์สลิป
                </button>
                <button onclick="resetForm()" class="flex-1 bg-pastel-primary hover:bg-pastel-dark text-white py-3.5 rounded-xl font-bold shadow-md flex justify-center items-center transition-colors">
                    <i class="fa-solid fa-plus mr-2"></i> คันต่อไป
                </button>
            </div>
        </div>

        <!-- ======================= CHECK-OUT VIEW ======================= -->
        <div id="view-checkout" class="hidden bg-white p-6 rounded-2xl shadow-xl border-t-4 border-t-pastel-primary">
            <h2 class="text-xl font-bold text-gray-800 mb-5 border-b pb-2 flex items-center">
                <i class="fa-solid fa-car-side mr-2 text-pastel-primary"></i> บันทึกรถออก
            </h2>
            
            <div class="mb-5">
                <label class="block text-sm font-bold text-gray-700 mb-2">รหัสผู้มาติดต่อ / สแกน QR</label>
                <div class="flex shadow-sm rounded-xl overflow-hidden border border-gray-200 focus-within:ring-2 focus-within:ring-pastel-primary focus-within:border-pastel-primary transition-all">
                    <input type="text" id="checkoutSearch" placeholder="ระบุ ID (เช่น V12345)" class="flex-1 p-3 border-none focus:outline-none">
                    <button onclick="searchVisitor()" class="bg-pastel-primary text-white px-5 hover:bg-pastel-dark transition-colors font-bold">
                        <i class="fa-solid fa-search"></i>
                    </button>
                </div>
                <p class="text-xs text-gray-400 mt-2 text-center"><i class="fa-solid fa-barcode mr-1"></i> (สามารถใช้เครื่องสแกนบาร์โค้ดยิงใส่ช่องด้านบนได้)</p>
            </div>

            <div id="checkoutResult" class="hidden mt-6 p-5 rounded-xl border-2 shadow-sm transition-all duration-300">
                <!-- Result will be populated by JS -->
            </div>
        </div>

        <!-- ======================= RESIDENT APPROVAL VIEW (MOCK) ======================= -->
        <div id="view-resident" class="hidden bg-white p-6 rounded-2xl shadow-xl border-t-8 border-pastel-primary text-center">
            <div class="w-20 h-20 bg-pastel-green rounded-full flex items-center justify-center mx-auto mb-5 text-pastel-dark text-4xl shadow-inner border-4 border-white">
                <i class="fa-solid fa-house-user"></i>
            </div>
            <h2 class="text-2xl font-bold text-gray-800 mb-2">ระบบประทับตราออนไลน์</h2>
            <p class="text-sm text-gray-500 mb-6 bg-gray-100 py-1 px-3 rounded-full inline-block">มีผู้มาติดต่อห้อง/บ้านของท่าน</p>
            
            <div class="bg-gray-50 p-4 rounded-xl text-left text-sm mb-6 border border-gray-100 shadow-sm">
                <p class="mb-2"><strong>รหัส:</strong> <span id="res-id" class="text-pastel-dark font-bold">--</span></p>
                <p class="mb-2"><strong>เวลาเข้า:</strong> <span id="res-time">--</span></p>
                <p class="mb-1"><strong>วัตถุประสงค์:</strong> <span id="res-purpose">--</span></p>
            </div>

            <button onclick="residentAction('Approved')" class="w-full bg-pastel-primary hover:bg-pastel-dark text-white py-3.5 rounded-xl font-bold shadow-md hover:shadow-lg transition-all mb-3 flex justify-center items-center text-lg">
                <i class="fa-solid fa-check-circle mr-2 text-xl"></i> ประทับตราอนุมัติ
            </button>
            <button onclick="residentAction('Rejected')" class="w-full bg-red-500 hover:bg-red-600 text-white py-3.5 rounded-xl font-bold shadow-md hover:shadow-lg transition-all flex justify-center items-center text-lg">
                <i class="fa-solid fa-times-circle mr-2 text-xl"></i> ปฏิเสธการเข้าพบ
            </button>
        </div>

        <!-- ======================= ADMIN GENERATE QR VIEW (เพิ่มใหม่) ======================= -->
        <div id="view-admin" class="hidden bg-white p-6 rounded-2xl shadow-xl border-t-4 border-t-pastel-primary">
            <h2 class="text-xl font-bold text-gray-800 mb-5 border-b pb-2 flex items-center">
                <i class="fa-solid fa-qrcode mr-2 text-pastel-primary"></i> สร้าง QR Code ลงทะเบียนลูกบ้าน
            </h2>
            <div class="mb-5">
                <label class="block text-sm font-bold text-gray-700 mb-2">ระบุบ้านเลขที่ / ห้องชุด</label>
                <div class="flex shadow-sm rounded-xl overflow-hidden border border-gray-200 focus-within:ring-2 focus-within:ring-pastel-primary focus-within:border-pastel-primary transition-all">
                    <input type="text" id="adminRoomInput" list="roomList" placeholder="เช่น 123/45" class="flex-1 p-3 border-none focus:outline-none" autocomplete="off">
                    <button onclick="generateResidentQR()" class="bg-pastel-primary text-white px-5 hover:bg-pastel-dark transition-colors font-bold">
                        สร้าง QR
                    </button>
                </div>
                <p class="text-xs text-gray-400 mt-2 text-center">พิมพ์แล้วกดสร้าง ให้ลูกบ้านสแกนเพื่อผูก LINE</p>
            </div>
            <div id="adminQrResult" class="hidden mt-6 text-center">
                <p class="text-lg font-bold text-pastel-dark mb-2" id="qrRoomLabel"></p>
                <div class="flex justify-center bg-white p-4 border-2 border-gray-100 rounded-xl shadow-sm inline-block">
                    <div id="residentQrcode"></div>
                </div>
                <p class="text-sm mt-3 text-gray-600 font-bold text-red-500"><i class="fa-brands fa-line mr-1"></i> ให้ลูกบ้านใช้แอป LINE สแกนเท่านั้น</p>
            </div>
        </div>

        <!-- ======================= LIFF REGISTER VIEW (เพิ่มใหม่ - หน้าที่ลูกบ้านจะเห็น) ======================= -->
        <div id="view-register" class="hidden bg-white p-6 rounded-2xl shadow-xl border-t-8 border-pastel-primary text-center">
            <div id="reg-loading">
                <i class="fa-solid fa-spinner fa-spin text-4xl text-pastel-primary mb-4"></i>
                <h2 class="text-xl font-bold text-gray-800">กำลังเชื่อมต่อ LINE...</h2>
                <p class="text-sm text-gray-500 mt-2" id="reg-room-text">กรุณารอสักครู่</p>
            </div>
            <div id="reg-result" class="hidden">
                <div class="w-20 h-20 bg-pastel-green rounded-full flex items-center justify-center mx-auto mb-5 text-pastel-dark text-4xl shadow-inner border-4 border-white">
                    <i class="fa-solid fa-check"></i>
                </div>
                <h2 class="text-2xl font-bold text-gray-800 mb-2">ลงทะเบียนสำเร็จ!</h2>
                <p class="text-sm text-gray-600 mb-2">ผูกบัญชีกับห้อง: <span id="res-success-room" class="font-bold text-pastel-dark text-lg">--</span></p>
                <p class="text-xs text-gray-500 bg-gray-100 py-3 px-3 rounded-lg mt-4">ระบบได้บันทึก LINE ID ของท่านแล้ว ท่านจะได้รับการแจ้งเตือนเมื่อมีผู้มาติดต่อ (ท่านสามารถปิดหน้านี้ได้ทันที)</p>
            </div>
        </div>

    </div>

    <!-- Application Logic -->
    <script>
        // --- การตั้งค่า LIFF ID สำหรับลงทะเบียนลูกบ้าน (เพิ่มบรรทัดนี้ไว้บนสุดของ script) ---
        const MY_LIFF_ID = "ใส่_LIFF_ID_ของคุณที่นี่"; // <- เอา LIFF ID มาใส่ตรงนี้

        // ฟังก์ชันครอบจักรวาล เพื่อป้องกัน Error เวลาค้นหา Element
        const getEl = (id) => document.getElementById(id);

        // ==========================================
        // 1. ระบบจัดการเวลา
        // ==========================================
        function initClock() {
            try {
                const clockEl = getEl('clock');
                const dateDisplay = getEl('dateDisplay');
                const timeDisplay = getEl('timeDisplay');
                const viewCheckin = getEl('view-checkin');

                function updateTime() {
                    const now = new Date();
                    if (clockEl) clockEl.innerText = now.toLocaleTimeString('th-TH');
                    
                    if (viewCheckin && !viewCheckin.classList.contains('hidden')) {
                        if (dateDisplay) dateDisplay.innerText = now.toLocaleDateString('th-TH');
                        if (timeDisplay) timeDisplay.innerText = now.toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'});
                    }
                }
                
                updateTime(); // รันทันที 1 ครั้ง
                setInterval(updateTime, 1000); // ตั้งเวลารันทุกวินาที
            } catch (err) {
                console.error("Clock Init Error:", err);
            }
        }

        // ==========================================
        // 2. ระบบจัดการแท็บเมนู
        // ==========================================
        function switchTab(tab) {
            try {
                // ซ่อนเนื้อหาทุกหน้า
                const views = ['view-checkin', 'view-checkout', 'view-slip', 'view-resident', 'view-admin', 'view-register'];
                views.forEach(id => {
                    const el = getEl(id);
                    if (el) el.classList.add('hidden');
                });
                
                // รีเซ็ตสไตล์ของปุ่มเมนูทั้งหมด
                const tabs = ['tab-checkin', 'tab-checkout', 'tab-admin'];
                tabs.forEach(id => {
                    const el = getEl(id);
                    if (el) el.className = "flex-1 py-2.5 text-center rounded-lg text-gray-500 font-semibold transition-all duration-200 hover:bg-gray-50";
                });

                // แสดงหน้าที่ต้องการ และไฮไลต์ปุ่ม
                const activeView = getEl('view-' + tab);
                const activeTab = getEl('tab-' + tab);

                if (activeView) activeView.classList.remove('hidden');
                if (activeTab) activeTab.className = "flex-1 py-2.5 text-center rounded-lg bg-pastel-primary text-white font-semibold shadow transition-all duration-200";
            } catch (err) {
                console.error("Tab Switch Error:", err);
            }
        }

        function toggleContactFields() {
            try {
                const typeNode = document.querySelector('input[name="contactType"]:checked');
                if (!typeNode) return;
                const type = typeNode.value;
                
                getEl('roomInputContainer').classList.add('hidden');
                getEl('otherInputContainer').classList.add('hidden');
                getEl('targetRoom').required = false;
                getEl('targetOther').required = false;

                if (type === 'room') {
                    getEl('roomInputContainer').classList.remove('hidden');
                    getEl('targetRoom').required = true;
                } else if (type === 'other') {
                    getEl('otherInputContainer').classList.remove('hidden');
                    getEl('targetOther').required = true;
                }
            } catch (err) {
                console.error("Toggle Fields Error:", err);
            }
        }

        // ==========================================
        // 3. ระบบจัดการรูปภาพ (บีบอัด Base64)
        // ==========================================
        let base64Image = "";
        function previewImage(event) {
            try {
                const file = event.target.files[0];
                if (!file) return;

                const textLabel = getEl('cameraText');
                if (textLabel) textLabel.innerText = "กำลังประมวลผลรูปภาพ...";

                const reader = new FileReader();
                reader.onload = function(e) {
                    const img = new Image();
                    img.onload = function() {
                        const canvas = document.createElement('canvas');
                        const MAX_WIDTH = 800;
                        const scaleSize = MAX_WIDTH / img.width;
                        canvas.width = MAX_WIDTH;
                        canvas.height = img.height * scaleSize;
                        const ctx = canvas.getContext('2d');
                        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                        
                        base64Image = canvas.toDataURL('image/jpeg', 0.7); 
                        const preview = getEl('imagePreview');
                        if (preview) {
                            preview.src = base64Image;
                            preview.classList.remove('hidden');
                        }
                        if (textLabel) textLabel.innerText = "เปลี่ยนรูปถ่าย";
                    }
                    img.onerror = function() {
                        Swal.fire('Error', 'ไม่สามารถประมวลผลรูปภาพได้ โปรดถ่ายรูปใหม่', 'error');
                        if (textLabel) textLabel.innerText = "แตะเพื่อเปิดกล้องถ่ายรูป";
                    }
                    img.src = e.target.result;
                }
                reader.onerror = function() {
                    Swal.fire('Error', 'เกิดข้อผิดพลาดในการอ่านไฟล์', 'error');
                    if (textLabel) textLabel.innerText = "แตะเพื่อเปิดกล้องถ่ายรูป";
                }
                reader.readAsDataURL(file);
            } catch (err) {
                console.error("Image Preview Error:", err);
            }
        }

        // ==========================================
        // 4. ระบบบันทึกข้อมูลและ Check-in/out
        // ==========================================
        function handleCheckin(e) {
            e.preventDefault();
            
            if (!base64Image || !base64Image.includes(',')) {
                Swal.fire('แจ้งเตือน', 'กรุณาถ่ายรูป / อัปโหลดรูปภาพให้เรียบร้อยก่อนกดบันทึก', 'warning');
                return;
            }

            try {
                const typeRadio = document.querySelector('input[name="contactType"]:checked');
                let targetText = "";
                if(typeRadio.value === 'room') targetText = "ห้อง " + getEl('targetRoom').value;
                else if(typeRadio.value === 'office') targetText = "สำนักงานนิติบุคคล";
                else if(typeRadio.value === 'visit') targetText = "เยี่ยมชมโครงการ";
                else targetText = "อื่นๆ: " + getEl('targetOther').value;

                const vehicleRadio = document.querySelector('input[name="vehicleType"]:checked');

                const payload = {
                    date: getEl('dateDisplay').innerText,
                    time: getEl('timeDisplay').innerText,
                    contactType: typeRadio.value,
                    target: targetText,
                    vehicleType: vehicleRadio ? vehicleRadio.value : '-',
                    licensePlate: getEl('licensePlate').value,
                    purpose: getEl('purpose').value,
                    image: base64Image.split(',')[1] 
                };

                Swal.fire({
                    title: 'กำลังบันทึกข้อมูล...',
                    text: 'กรุณารอสักครู่ ระบบกำลังสื่อสารกับเซิร์ฟเวอร์',
                    allowOutsideClick: false,
                    didOpen: () => { Swal.showLoading(); }
                });

                if (typeof google !== 'undefined' && google.script && google.script.run) {
                    google.script.run
                        .withSuccessHandler(showSlip)
                        .withFailureHandler(err => {
                            Swal.fire('เกิดข้อผิดพลาด', err.message || 'ไม่สามารถบันทึกข้อมูลได้', 'error');
                        })
                        .saveVisitorData(payload);
                } else {
                    setTimeout(() => {
                        const mockId = "V" + Math.floor(Date.now() / 1000).toString().slice(-6);
                        showSlip({
                            id: mockId,
                            date: payload.date,
                            time: payload.time,
                            target: payload.target,
                            purpose: payload.purpose,
                            vehicleType: payload.vehicleType,
                            licensePlate: payload.licensePlate
                        });
                        window.mockDB = window.mockDB || {};
                        window.mockDB[mockId] = { ...payload, id: mockId, status: 'Pending' };
                    }, 1500);
                }
            } catch (err) {
                Swal.fire('Error', 'เกิดข้อผิดพลาดในการรวบรวมข้อมูล: ' + err.message, 'error');
            }
        }

        function showSlip(data) {
            try {
                Swal.close(); 
                
                getEl('view-checkin').classList.add('hidden');
                getEl('view-slip').classList.remove('hidden');

                getEl('slip-id').innerText = data.id;
                getEl('slip-date').innerText = data.date;
                getEl('slip-time').innerText = data.time;
                getEl('slip-target').innerText = data.target;
                getEl('slip-purpose').innerText = data.purpose;
                getEl('slip-vehicle').innerText = data.vehicleType || '-';
                getEl('slip-license').innerText = data.licensePlate || '-';

                const qrcodeEl = getEl('qrcode');
                qrcodeEl.innerHTML = "";
                new QRCode(qrcodeEl, {
                    text: "https://myguest.web.app/?view=resident&id=" + data.id, 
                    width: 120,
                    height: 120,
                    colorDark : "#059669",
                    colorLight : "#ffffff",
                    correctLevel : QRCode.CorrectLevel.H
                });
                
                Swal.fire({
                    icon: 'success',
                    title: 'บันทึกสำเร็จ',
                    text: 'สามารถพิมพ์สลิปให้ผู้มาติดต่อได้ทันที',
                    confirmButtonColor: '#34d399'
                });
            } catch (err) {
                console.error("Show Slip Error:", err);
            }
        }

        function resetForm() {
            getEl('checkinForm').reset();
            const preview = getEl('imagePreview');
            if(preview) {
                preview.classList.add('hidden');
                preview.src = "";
            }
            base64Image = "";
            const camText = getEl('cameraText');
            if(camText) camText.innerText = "แตะเพื่อเปิดกล้องถ่ายรูป";
            toggleContactFields();
            getEl('view-slip').classList.add('hidden');
            getEl('view-checkin').classList.remove('hidden');
        }

        function searchVisitor() {
            const searchInput = getEl('checkoutSearch');
            if(!searchInput) return;
            const searchId = searchInput.value.trim();
            if(!searchId) return;

            Swal.fire({ title: 'กำลังค้นหา...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});

            try {
                if (typeof google !== 'undefined' && google.script && google.script.run) {
                    google.script.run
                        .withSuccessHandler(renderCheckoutResult)
                        .withFailureHandler(err => Swal.fire('Error', err.message || 'ไม่พบข้อมูล', 'error'))
                        .getVisitorData(searchId);
                } else {
                    setTimeout(() => {
                        const data = window.mockDB ? window.mockDB[searchId] : null;
                        if(data) renderCheckoutResult(data);
                        else Swal.fire('Error', 'ไม่พบข้อมูลรหัสดังกล่าว', 'error');
                    }, 800);
                }
            } catch (err) {
                Swal.fire('Error', 'เกิดข้อผิดพลาดในการค้นหา: ' + err.message, 'error');
            }
        }

        function renderCheckoutResult(data) {
            Swal.close(); 
            const resDiv = getEl('checkoutResult');
            if(!resDiv) return;
            resDiv.classList.remove('hidden');
            
            let statusColor = data.status === 'Approved' ? 'bg-green-50 text-green-700 border-green-300' : 
                              (data.status === 'Rejected' ? 'bg-red-50 text-red-700 border-red-300' : 'bg-yellow-50 text-yellow-700 border-yellow-300');
            
            let statusText = data.status === 'Approved' ? '<i class="fa-solid fa-check-circle"></i> ประทับตราแล้ว' : 
                             (data.status === 'Rejected' ? '<i class="fa-solid fa-times-circle"></i> ปฏิเสธการเข้าพบ' : '<i class="fa-solid fa-clock"></i> รอดำเนินการ');

            resDiv.className = `mt-6 p-5 rounded-xl border-2 shadow-sm transition-all duration-300 ${statusColor}`;
            resDiv.innerHTML = `
                <div class="flex justify-between items-start mb-3 border-b border-gray-200/50 pb-2">
                    <h3 class="font-bold text-lg">รหัส: ${data.id}</h3>
                    <span class="px-2 py-1 bg-white/80 rounded-md text-xs font-bold shadow-sm">${statusText}</span>
                </div>
                <div class="space-y-1 mb-4">
                    <p class="text-sm"><strong>ติดต่อ:</strong> ${data.target}</p>
                    <p class="text-sm"><strong>เวลาเข้า:</strong> ${data.time}</p>
                    <p class="text-sm"><strong>วัตถุประสงค์:</strong> ${data.purpose}</p>
                    ${data.vehicleType ? `<p class="text-sm"><strong>รถ:</strong> ${data.vehicleType} (${data.licensePlate})</p>` : ''}
                </div>
                
                <button onclick="confirmCheckout('${data.id}')" class="w-full bg-gray-800 hover:bg-black text-white py-3 rounded-xl shadow-md font-bold transition-transform hover:-translate-y-0.5">
                    <i class="fa-solid fa-sign-out-alt mr-2"></i> บันทึกรถออก (Check-out)
                </button>
            `;
        }

        function confirmCheckout(id) {
            Swal.fire({
                title: 'ยืนยันนำรถออก?',
                text: "บันทึกเวลาออกให้รหัส " + id,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#34d399',
                cancelButtonColor: '#d33',
                confirmButtonText: 'ยืนยัน',
                cancelButtonText: 'ยกเลิก',
                shape: 'rounded-xl'
            }).then((result) => {
                if (result.isConfirmed) {
                    Swal.fire({ title: 'กำลังบันทึก...', allowOutsideClick: false, didOpen: () => { Swal.showLoading(); }});
                    
                    try {
                        if (typeof google !== 'undefined' && google.script && google.script.run) {
                            google.script.run
                                .withSuccessHandler(() => {
                                    Swal.fire('สำเร็จ', 'บันทึกรถออกเรียบร้อย', 'success');
                                    getEl('checkoutResult').classList.add('hidden');
                                    getEl('checkoutSearch').value = '';
                                })
                                .withFailureHandler(err => Swal.fire('Error', err.message || 'บันทึกไม่สำเร็จ', 'error'))
                                .setCheckoutTime(id);
                        } else {
                            window.mockDB[id].status = 'Checked-Out';
                            Swal.fire('สำเร็จ', 'บันทึกรถออกเรียบร้อย (Mock)', 'success');
                            getEl('checkoutResult').classList.add('hidden');
                            getEl('checkoutSearch').value = '';
                        }
                    } catch (err) {
                        Swal.fire('Error', 'เกิดข้อผิดพลาดในการเชื่อมต่อ', 'error');
                    }
                }
            })
        }

        // ==========================================
        // 5. ระบบตรวจสอบเส้นทาง (Router) เมื่อโหลดหน้าเว็บ
        // ==========================================
        window.addEventListener('load', () => {
            initClock(); // เริ่มนับเวลาเมื่อโหลดหน้าเว็บเสร็จ
            try {
                const urlParams = new URLSearchParams(window.location.search);
                const view = urlParams.get('view');
                const id = urlParams.get('id');
                const regRoom = urlParams.get('reg_room'); 

                const tabCheckin = getEl('tab-checkin');
                const viewCheckin = getEl('view-checkin');

                if (view === 'resident' || regRoom) {
                    if (tabCheckin && tabCheckin.parentElement) tabCheckin.parentElement.classList.add('hidden');
                    if (viewCheckin) viewCheckin.classList.add('hidden');
                }

                if (view === 'resident' && id) {
                    const viewRes = getEl('view-resident');
                    if (viewRes) viewRes.classList.remove('hidden');
                    if (getEl('res-id')) getEl('res-id').innerText = id;
                    if (getEl('res-time')) getEl('res-time').innerText = "กำลังดึงข้อมูล..."; 
                    if (getEl('res-purpose')) getEl('res-purpose').innerText = "กำลังดึงข้อมูล...";
                } 
                else if (regRoom) {
                    const viewReg = getEl('view-register');
                    if (viewReg) viewReg.classList.remove('hidden');
                    if (getEl('reg-room-text')) getEl('reg-room-text').innerText = "กำลังผูก LINE กับบ้านเลขที่: " + regRoom;
                    initLIFFForRegistration(regRoom);
                }

                // สั่งดึงข้อมูลรายชื่อห้องมาใส่ Autocomplete ตอนเปิดเว็บ
                const roomInput = getEl('targetRoom');
                if(roomInput) roomInput.placeholder = "กำลังโหลดข้อมูลห้อง...";

                if (typeof google !== 'undefined' && google.script && google.script.run) {
                    google.script.run
                        .withSuccessHandler((rooms) => {
                            populateRoomList(rooms);
                            if(roomInput) roomInput.placeholder = "เช่น 123/45 (พิมพ์เพื่อค้นหา)";
                        })
                        .withFailureHandler(err => {
                            console.error('ไม่สามารถโหลดรายชื่อห้องได้', err);
                            if(roomInput) roomInput.placeholder = "โหลดข้อมูลล้มเหลว";
                            
                            // แสดง Error ให้เห็นชัดเจนว่าพังที่ Backend
                            Swal.fire({
                                title: 'เชื่อมต่อฐานข้อมูลล้มเหลว',
                                text: 'ไม่สามารถดึงรายชื่อห้องได้ กรุณาตรวจสอบการตั้งค่า Sheet ID หรือการ Deploy (ต้อง Deploy เป็นเวอร์ชันใหม่เสมอ) | Error Detail: ' + (err.message || 'Unknown Error'),
                                icon: 'error',
                                toast: true,
                                position: 'top',
                                showConfirmButton: false,
                                timer: 6000
                            });
                        })
                        .getRoomList();
                } else {
                    // โหมดจำลองข้อมูล (รันบนบราวเซอร์ทั่วไป)
                    populateRoomList(["101", "102", "123/45"]);
                    if(roomInput) roomInput.placeholder = "เช่น 123/45 (โหมดจำลอง)";
                }
            } catch (err) {
                console.error("Window Load Error:", err);
            }
        });

        function populateRoomList(rooms) {
            const dataList = getEl('roomList');
            if(!dataList) return;
            dataList.innerHTML = ''; 
            if(rooms && rooms.length > 0) {
                rooms.forEach(room => {
                    const option = document.createElement('option');
                    option.value = room;
                    dataList.appendChild(option);
                });
            }
        }

        function residentAction(action) {
            const idEl = getEl('res-id');
            const id = idEl ? idEl.innerText : '--';
            Swal.fire({
                title: 'ยืนยัน?',
                text: "คุณต้องการ " + (action==='Approved' ? 'ประทับตราอนุมัติ' : 'ปฏิเสธ') + " ใช่หรือไม่",
                icon: 'question',
                showCancelButton: true,
                confirmButtonColor: action==='Approved' ? '#34d399' : '#ef4444',
                confirmButtonText: 'ยืนยัน',
            }).then((result) => {
                if (result.isConfirmed) {
                    Swal.fire('สำเร็จ!', 'บันทึกข้อมูลแล้ว', 'success');
                }
            });
        }

        // ==========================================
        // 6. ระบบสร้าง QR และลงทะเบียนผ่าน LIFF
        // ==========================================
        function generateResidentQR() {
            try {
                const roomInput = getEl('adminRoomInput');
                if(!roomInput) return;
                const room = roomInput.value.trim();
                
                if(!room) {
                    Swal.fire('แจ้งเตือน', 'กรุณาระบุเลขห้อง/บ้านเลขที่ ก่อน', 'warning');
                    return;
                }
                if(!MY_LIFF_ID || MY_LIFF_ID === "2009358681-P2mTrvGB") {
                    Swal.fire('ยังไม่ได้ตั้งค่า', 'ผู้ดูแลระบบต้องใส่ค่า LIFF ID ในโค้ด HTML ก่อนจึงจะใช้งานได้', 'error');
                    return;
                }

                const liffUrl = "https://liff.line.me/" + MY_LIFF_ID + "?reg_room=" + encodeURIComponent(room);
                
                if(getEl('qrRoomLabel')) getEl('qrRoomLabel').innerText = "บ้านเลขที่: " + room;
                const qrContainer = getEl('residentQrcode');
                if(qrContainer) {
                    qrContainer.innerHTML = "";
                    new QRCode(qrContainer, {
                        text: liffUrl,
                        width: 200,
                        height: 200,
                        colorDark : "#059669",
                        colorLight : "#ffffff",
                        correctLevel : QRCode.CorrectLevel.H
                    });
                }
                if(getEl('adminQrResult')) getEl('adminQrResult').classList.remove('hidden');
            } catch (err) {
                console.error("Generate QR Error:", err);
            }
        }

        async function initLIFFForRegistration(room) {
            try {
                if (typeof liff === 'undefined') {
                    throw new Error("ระบบโหลดส่วนเสริม LINE (LIFF SDK) ไม่สำเร็จ กรุณาตรวจสอบอินเทอร์เน็ต");
                }

                await liff.init({ liffId: MY_LIFF_ID });
                
                if (!liff.isLoggedIn()) {
                    liff.login();
                    return;
                }

                const profile = await liff.getProfile();
                const userId = profile.userId;

                if (typeof google !== 'undefined' && google.script && google.script.run) {
                    google.script.run
                        .withSuccessHandler(function(response) {
                            if(getEl('reg-loading')) getEl('reg-loading').classList.add('hidden');
                            if(response && response.success) {
                                if(getEl('reg-result')) getEl('reg-result').classList.remove('hidden');
                                if(getEl('res-success-room')) getEl('res-success-room').innerText = room;
                            } else {
                                Swal.fire('เกิดข้อผิดพลาด', (response && response.message) ? response.message : 'ไม่สามารถบันทึกได้', 'error').then(() => liff.closeWindow());
                            }
                        })
                        .withFailureHandler(err => {
                            Swal.fire('Error', 'การเชื่อมต่อขัดข้อง: ' + err.message, 'error');
                        })
                        .registerResidentLine(room, userId);
                } else {
                    setTimeout(() => {
                        if(getEl('reg-loading')) getEl('reg-loading').classList.add('hidden');
                        if(getEl('reg-result')) getEl('reg-result').classList.remove('hidden');
                        if(getEl('res-success-room')) getEl('res-success-room').innerText = room + " (Mock: " + userId + ")";
                    }, 1500);
                }
            } catch (err) {
                console.error("LIFF Init Error:", err);
                Swal.fire('Error', err.message || 'ไม่สามารถเชื่อมต่อ LINE ได้ กรุณาสแกน QR Code นี้ผ่านแอปพลิเคชัน LINE เท่านั้น', 'error');
            }
        }
    </script>
</body>
</html>
