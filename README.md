<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>MY Guest</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
  <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Kanit', sans-serif; background-color: #f0fdf4; }
    .pastel-green { background-color: #dcfce7; }
    .btn-green { background-color: #10b981; color: white; transition: all 0.3s; }
    .btn-green:hover { background-color: #059669; transform: translateY(-2px); }
    .card { background: white; border-radius: 1.5rem; box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
    ::-webkit-scrollbar { width: 6px; }
    ::-webkit-scrollbar-track { background: #f1f1f1; border-radius: 4px; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
    ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    @media print {
      body * { visibility: hidden; }
      #slip-print, #slip-print * { visibility: visible; }
      #slip-print { position: absolute; left: 0; top: 0; width: 80mm; padding: 10px; border: none; }
      .no-print { display: none !important; }
    }
  </style>
</head>
<body class="p-2 md:p-8">

  <div id="app" class="max-w-md mx-auto">
    <!-- Header -->
    <div class="text-center mb-6 no-print">
      <h1 class="text-3xl font-bold text-emerald-700">MY Guest</h1>
      <p class="text-emerald-500 text-sm">ระบบจัดการผู้มาติดต่ออัจฉริยะ</p>
    </div>

    <!-- Navigation -->
    <div id="nav" class="flex gap-1 mb-6 no-print text-sm">
      <button onclick="showPage('check-in')" id="nav-check-in" class="flex-1 py-3 rounded-xl border-2 border-emerald-500 bg-emerald-50 text-emerald-700 font-bold shadow-sm transition-all">บันทึกเข้า</button>
      <button onclick="showPage('check-out')" id="nav-check-out" class="flex-1 py-3 rounded-xl border-2 border-transparent bg-white text-emerald-700 font-medium shadow-sm transition-all">ตรวจสอบออก</button>
      <button onclick="showPage('register')" id="nav-register" class="flex-1 py-3 rounded-xl border-2 border-transparent bg-white text-emerald-700 font-medium shadow-sm transition-all">QR ลูกบ้าน</button>
    </div>

    <!-- Page: Check-In Form -->
    <div id="page-check-in" class="page card p-6">
      <h2 class="text-xl font-semibold mb-4 text-gray-700 flex items-center">
        <span class="w-2 h-6 bg-emerald-500 rounded-full mr-2"></span>บันทึกผู้มาติดต่อ
      </h2>
      
      <!-- 🌟 เอา onsubmit ออก เพื่อป้องกันหน้ารีเฟรชจนเป็นสีขาว -->
      <form id="checkInForm" class="space-y-4">
        
        <div class="flex space-x-2">
          <div class="flex-1">
            <label class="block text-xs font-bold text-gray-400 uppercase mb-1">วันที่</label>
            <input type="text" id="display-date" class="w-full p-3 rounded-xl border border-gray-200 bg-gray-100 text-gray-500 outline-none cursor-not-allowed text-sm text-center font-medium" readonly>
          </div>
          <div class="flex-1">
            <label class="block text-xs font-bold text-gray-400 uppercase mb-1">เวลา</label>
            <input type="text" id="display-time" class="w-full p-3 rounded-xl border border-gray-200 bg-gray-100 text-emerald-600 outline-none cursor-not-allowed text-sm text-center font-bold tracking-widest" readonly>
          </div>
        </div>

        <div>
          <label class="block text-xs font-bold text-gray-400 uppercase mb-1">ประเภทการติดต่อ</label>
          <div class="grid grid-cols-2 gap-2 mb-2">
            <label class="p-3 border rounded-xl flex items-center space-x-2 cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="contactType" value="บ้านเลขที่/ห้องชุด" checked class="accent-emerald-500" onchange="toggleRoomInput()">
              <span class="text-sm">บ้านเลขที่/ห้องชุด</span>
            </label>
            <label class="p-3 border rounded-xl flex items-center space-x-2 cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="contactType" value="สำนักงานนิติฯ" class="accent-emerald-500" onchange="toggleRoomInput()">
              <span class="text-sm">สำนักงานนิติฯ</span>
            </label>
            <label class="p-3 border rounded-xl flex items-center space-x-2 cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="contactType" value="เยี่ยมชมโครงการ" class="accent-emerald-500" onchange="toggleRoomInput()">
              <span class="text-sm">เยี่ยมชมโครงการ</span>
            </label>
            <label class="p-3 border rounded-xl flex items-center space-x-2 cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="contactType" value="อื่นๆ" class="accent-emerald-500" onchange="toggleRoomInput()">
              <span class="text-sm">อื่นๆ</span>
            </label>
          </div>
        </div>

        <div id="roomInputContainer">
          <label class="block text-xs font-bold text-gray-400 uppercase mb-1">เลขห้อง / บ้านเลขที่</label>
          <input type="text" id="targetRoom" name="targetRoom" list="residentList" class="w-full p-3 rounded-xl border border-emerald-100 bg-emerald-50 focus:outline-none focus:ring-2 focus:ring-emerald-500" placeholder="กำลังเชื่อมต่อฐานข้อมูล..." autocomplete="off">
          <datalist id="residentList"></datalist>
          <p id="resident-status" class="text-[10px] text-gray-400 mt-1 italic">กำลังโหลดข้อมูลจาก Sheet...</p>
        </div>

        <div>
          <label class="block text-xs font-bold text-gray-400 uppercase mb-1">ข้อมูลยานพาหนะ</label>
          <div class="grid grid-cols-2 gap-2 mb-2">
            <label class="p-3 border rounded-xl flex items-center space-x-2 cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="vehicleType" value="รถยนต์" checked class="accent-emerald-500">
              <span class="text-sm">🚗 รถยนต์</span>
            </label>
            <label class="p-3 border rounded-xl flex items-center space-x-2 cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="vehicleType" value="รถจักรยานยนต์" class="accent-emerald-500">
              <span class="text-sm">🏍️ มอเตอร์ไซค์</span>
            </label>
          </div>
          <input type="text" id="licensePlate" class="w-full p-3 rounded-xl border border-emerald-100 bg-emerald-50 focus:outline-none focus:ring-2 focus:ring-emerald-500" placeholder="ระบุป้ายทะเบียน (เช่น กค 1234 กทม.)">
        </div>

        <div>
          <label class="block text-xs font-bold text-gray-400 uppercase mb-1">รูปถ่ายทะเบียนรถ</label>
          <div class="relative">
            <input type="file" id="camera" accept="image/*" capture="environment" class="hidden" onchange="previewImage(event)">
            <button type="button" onclick="document.getElementById('camera').click()" class="w-full py-4 rounded-xl border-2 border-dashed border-emerald-200 text-emerald-600 hover:bg-emerald-50 font-medium">
              📸 กดเพื่อเปิดกล้องถ่ายรูป
            </button>
            <img id="imgPreview" class="mt-2 rounded-xl hidden w-full h-40 object-cover border shadow-sm">
          </div>
        </div>

        <div>
          <label class="block text-xs font-bold text-gray-400 uppercase mb-1">วัตถุประสงค์</label>
          <div class="grid grid-cols-2 gap-2 text-sm">
            <label class="flex items-center space-x-2 p-2 border rounded-lg cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="purpose" value="ญาติ" checked class="accent-emerald-500"> <span>ญาติ/เพื่อน</span>
            </label>
            <label class="flex items-center space-x-2 p-2 border rounded-lg cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="purpose" value="ส่งของ" class="accent-emerald-500"> <span>ส่งของ</span>
            </label>
            <label class="flex items-center space-x-2 p-2 border rounded-lg cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="purpose" value="นัดหมายไว้" class="accent-emerald-500"> <span>นัดหมายไว้</span>
            </label>
            <label class="flex items-center space-x-2 p-2 border rounded-lg cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="purpose" value="ผู้รับเหมา/ช่าง" class="accent-emerald-500"> <span>ผู้รับเหมา/ช่าง</span>
            </label>
            <label class="flex items-center space-x-2 p-2 border rounded-lg cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="purpose" value="เยี่ยมชมโครงการ" class="accent-emerald-500"> <span>เยี่ยมชมโครงการ</span>
            </label>
            <label class="flex items-center space-x-2 p-2 border rounded-lg cursor-pointer has-[:checked]:bg-emerald-50 has-[:checked]:border-emerald-300">
              <input type="radio" name="purpose" value="ประชุม" class="accent-emerald-500"> <span>ประชุม</span>
            </label>
          </div>
        </div>

        <!-- 🌟 เปลี่ยนเป็นปุ่มธรรมดา (type="button") และจับ event ด้วย onclick -->
        <button type="button" id="btnSubmit" onclick="handleCheckIn()" class="w-full py-4 rounded-2xl btn-green font-bold text-lg shadow-lg shadow-emerald-200 mt-4">
          บันทึกข้อมูลและออกสลิป
        </button>
      </form>
    </div>

    <!-- Page: Register Resident -->
    <div id="page-register" class="page hidden card p-6">
      <h2 class="text-xl font-semibold mb-4 text-gray-700 flex items-center">
        <span class="w-2 h-6 bg-emerald-500 rounded-full mr-2"></span>สร้าง QR ลงทะเบียนลูกบ้าน
      </h2>
      <div class="space-y-4">
        <div class="p-3 bg-emerald-50 border border-emerald-100 rounded-xl mb-4">
          <p class="text-xs text-emerald-700 leading-relaxed">
            📌 <b>วิธีใช้งาน:</b> ระบุเลขห้อง แล้วสร้าง QR Code เมื่อลูกบ้านสแกน ให้กด "ส่งข้อความ" ผ่านแอป LINE เพื่อลงทะเบียน
          </p>
        </div>

        <div>
          <label class="block text-xs font-bold text-gray-400 uppercase mb-1">ระบุบ้านเลขที่ / ห้องชุด</label>
          <input type="text" id="regTargetRoom" list="residentList" class="w-full p-4 rounded-xl border border-emerald-100 bg-emerald-50/50 focus:outline-none focus:ring-2 focus:ring-emerald-500 text-lg font-bold text-emerald-800" placeholder="เช่น 34/1">
        </div>

        <button onclick="generateRegQR()" class="w-full py-4 rounded-xl btn-green font-bold shadow-md shadow-emerald-200 mt-2 text-lg">
          สร้าง QR Code
        </button>

        <div id="regQrResult" class="hidden mt-6 text-center border-t border-dashed border-gray-200 pt-6">
           <h3 class="text-xl font-black text-emerald-700" id="regRoomDisplay"></h3>
           <p class="text-sm text-gray-500 mb-4 mt-1">สแกนเพื่อรับการแจ้งเตือนผู้มาติดต่อ</p>
           
           <div class="flex justify-center p-4 bg-white border-4 border-emerald-100 rounded-3xl mx-auto w-fit shadow-sm">
             <img id="regQrImage" class="w-48 h-48">
           </div>
           
           <div class="mt-4 p-3 bg-red-50 rounded-xl border border-red-100 inline-block animate-pulse">
             <p class="text-xs text-red-600 font-bold">⚠️ สำคัญ: เมื่อสแกนแล้ว</p>
             <p class="text-xs text-red-600">ให้กดส่งข้อความใน LINE เพื่อยืนยันด้วยครับ</p>
           </div>
        </div>
      </div>
    </div>

    <!-- Page: Slip Display -->
    <div id="page-slip" class="page hidden text-center">
      <div id="slip-print" class="card p-6 bg-white border-t-8 border-emerald-500">
        <h3 class="text-2xl font-black text-emerald-700">MY Guest</h3>
        <p class="text-[10px] text-gray-400 uppercase tracking-widest border-b pb-2 mb-4">Visitor Entry Pass</p>
        
        <div class="text-left space-y-2 text-sm mb-4 border-b border-dashed pb-4">
          <div class="flex justify-between"><span class="text-gray-400">ID:</span> <span class="font-mono font-bold" id="slip-id"></span></div>
          <div class="flex justify-between"><span class="text-gray-400">ติดต่อห้อง:</span> <span class="font-bold" id="slip-room"></span></div>
          <div class="flex justify-between"><span class="text-gray-400">ยานพาหนะ:</span> <span class="font-bold" id="slip-vehicle"></span></div>
          <div class="flex justify-between"><span class="text-gray-400">วัน/เวลาเข้า:</span> <span id="slip-time"></span></div>
          <div class="flex justify-between"><span class="text-gray-400">วัตถุประสงค์:</span> <span id="slip-purpose"></span></div>
        </div>

        <div class="flex justify-center mb-4 p-2 bg-emerald-50 rounded-xl">
          <div id="qrcode"></div>
        </div>
        
        <p class="text-[10px] text-gray-500 italic leading-tight">** กรุณาให้เจ้าของห้องประทับตราผ่าน LINE **<br>และส่งคืน รปภ. เมื่อนำรถออก</p>
      </div>
      
      <div class="flex flex-col gap-2 mt-6 no-print">
        <button onclick="window.print()" class="w-full py-3 rounded-xl bg-gray-800 text-white font-medium">🖨️ พิมพ์สลิป (เครื่องพิมพ์ความร้อน)</button>
        <button onclick="showPage('check-in')" class="w-full py-3 rounded-xl btn-green font-medium">กลับหน้าหลัก</button>
      </div>
    </div>

    <!-- Page: Check-Out -->
    <div id="page-check-out" class="page hidden card p-6">
      <h2 class="text-xl font-semibold mb-4 text-gray-700">ตรวจสอบรถออก</h2>
      <div class="space-y-4">
        
        <div class="relative">
          <input type="text" id="searchId" class="w-full p-4 pl-12 rounded-xl border border-emerald-100 bg-emerald-50/50 outline-none focus:ring-2 focus:ring-emerald-500" placeholder="ระบุ Visitor ID หรือ เลือกจากรายการด้านล่าง...">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 absolute left-4 top-4 text-emerald-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
          </svg>
        </div>
        <button onclick="searchVisitor()" class="w-full py-4 rounded-xl btn-green font-bold shadow-md shadow-emerald-200">ตรวจสอบข้อมูล</button>
        
        <div id="checkoutInfo" class="hidden mt-4 p-4 rounded-2xl border-2 border-dashed border-gray-200 bg-white space-y-3">
           <div class="flex items-center justify-between">
              <span class="text-sm text-gray-400">สถานะ:</span>
              <span id="co-status" class="px-3 py-1 rounded-full text-xs font-bold"></span>
           </div>
           <div class="flex items-center justify-between">
              <span class="text-sm text-gray-400">วันที่ / เวลาเข้า:</span>
              <span class="font-bold text-gray-700 text-sm"><span id="co-date"></span> <span id="co-time" class="text-emerald-600"></span></span>
           </div>
           <div class="flex items-center justify-between">
              <span class="text-sm text-gray-400">ประเภทติดต่อ:</span>
              <span id="co-contactType" class="font-bold text-gray-700 text-sm"></span>
           </div>
           <div class="flex items-center justify-between">
              <span class="text-sm text-gray-400">ห้องที่ติดต่อ:</span>
              <span id="co-room" class="font-bold text-gray-700 text-sm"></span>
           </div>
           
           <div class="flex items-center justify-between border-t border-dashed border-gray-200 pt-3 mt-1">
              <span class="text-sm text-gray-400">ประเภทรถ:</span>
              <span id="co-vType" class="font-bold text-emerald-700 text-sm"></span>
           </div>
           <div class="flex items-center justify-between">
              <span class="text-sm text-gray-400">ทะเบียนรถ:</span>
              <span id="co-lPlate" class="font-bold text-emerald-700 text-sm bg-emerald-50 px-2 py-1 rounded-md"></span>
           </div>
           
           <div id="co-photo-container" class="hidden mt-2">
              <span class="text-xs text-gray-400 mb-1 block">รูปถ่ายที่บันทึกไว้:</span>
              <img id="co-photo" class="w-full h-32 object-cover rounded-xl border border-gray-200 shadow-sm">
           </div>

           <button id="btnCheckout" onclick="handleCheckout()" class="w-full mt-4 py-4 rounded-xl bg-orange-500 text-white font-bold shadow-lg shadow-orange-100 hidden">
             บันทึกรถออกจากโครงการ
           </button>
        </div>

        <div id="activeVisitorsSection" class="mt-6 border-t-2 border-dashed border-gray-100 pt-5">
          <div class="flex justify-between items-center mb-3">
            <h3 class="text-sm font-bold text-gray-500 flex items-center"><span class="mr-2">🚗</span> รถที่ยังไม่ออกจากโครงการ</h3>
            <button onclick="loadActiveVisitors()" class="text-xs text-emerald-600 font-medium hover:underline flex items-center">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
              </svg> รีเฟรช
            </button>
          </div>
          <div id="activeListContainer" class="space-y-2 max-h-60 overflow-y-auto pr-1">
             <!-- แสดงรายการจากระบบ -->
          </div>
        </div>

      </div>
    </div>

    <!-- Page: Resident Stamp -->
    <div id="page-stamp" class="page hidden card p-6 text-center">
      <h2 class="text-xl font-semibold mb-4">ประทับตราอนุมัติ</h2>
      <div id="stamp-content">
        <img id="stamp-img" class="rounded-xl w-full h-48 object-cover mb-4 shadow-md">
        <p class="mb-4">ผู้มาติดต่อห้อง: <span id="stamp-room" class="font-bold text-emerald-600"></span></p>
        <button onclick="submitStamp('Stamped')" class="w-full py-4 rounded-xl btn-green font-bold text-lg mb-2 shadow-lg">✅ อนุมัติให้เข้าพบ</button>
        <button onclick="submitStamp('Rejected')" class="w-full py-4 rounded-xl bg-red-500 text-white font-bold text-lg shadow-lg">❌ ปฏิเสธการเข้าพบ</button>
      </div>
    </div>

  </div>

  <script>
    // ==========================================
    // ⚙️ ตั้งค่าระบบ
    // ==========================================
    const LINE_BOT_ID = "@397vnkca"; // ไอดีบอทของคุณ
    // ==========================================

    let currentPhoto = "";
    let currentVisitorId = "";

    function updateClock() {
      const now = new Date();
      const dateEl = document.getElementById('display-date');
      const timeEl = document.getElementById('display-time');
      if (dateEl && timeEl) {
        dateEl.value = now.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
        timeEl.value = now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
      }
    }
    updateClock();
    setInterval(updateClock, 1000);

    document.addEventListener('DOMContentLoaded', function() {
      const statusText = document.getElementById('resident-status');
      if(statusText) statusText.innerText = "กำลังดึงข้อมูลลูกบ้าน...";
      
      // ดัก Error กรณีเปิดนอก Google Apps Script
      if (typeof google === 'undefined') {
        if(statusText) statusText.innerText = "กำลังรันในโหมดพรีวิว";
        return;
      }
      
      google.script.run
        .withSuccessHandler(populateResidents)
        .withFailureHandler(function(error) {
          if(statusText) {
            statusText.innerText = "❌ โหลดข้อมูลล้มเหลว";
            statusText.classList.replace('text-gray-400', 'text-red-500');
          }
        })
        .getResidents();
        
      const params = new URLSearchParams(window.location.search);
      if (params.get('page') === 'stamp') {
        currentVisitorId = params.get('id');
        showStampPage(currentVisitorId);
      }
    });

    function populateResidents(data) {
      const list = document.getElementById('residentList');
      const input = document.getElementById('targetRoom');
      const status = document.getElementById('resident-status');
      
      if(list && input) {
        list.innerHTML = data.map(r => `<option value="${r.room}">`).join('');
        if(status) {
          status.innerText = `✅ เชื่อมต่อฐานข้อมูลแล้ว`;
          status.classList.replace('text-gray-400', 'text-emerald-500');
          status.classList.remove('italic');
        }
      }
    }

    function toggleRoomInput() {
      const type = document.querySelector('input[name="contactType"]:checked').value;
      document.getElementById('roomInputContainer').style.display = (type === 'บ้านเลขที่/ห้องชุด') ? 'block' : 'none';
    }

    function previewImage(event) {
      if (!event.target.files || !event.target.files[0]) return;
      const reader = new FileReader();
      reader.onload = function(e) {
        document.getElementById('imgPreview').src = e.target.result;
        document.getElementById('imgPreview').classList.remove('hidden');
        currentPhoto = e.target.result;
      }
      reader.readAsDataURL(event.target.files[0]);
    }

    // 🌟 แก้ไขฟังก์ชันการคลิกบันทึก
    function handleCheckIn() {
      // ตรวจสอบความถูกต้องเบื้องต้น
      const contactType = document.querySelector('input[name="contactType"]:checked').value;
      const targetRoom = document.getElementById('targetRoom').value;

      if (contactType === 'บ้านเลขที่/ห้องชุด' && !targetRoom) {
        return Swal.fire('แจ้งเตือน', 'กรุณาระบุเลขห้องที่ต้องการติดต่อ', 'warning');
      }

      const btn = document.getElementById('btnSubmit');
      btn.disabled = true;

      // 🌟 เพิ่ม Popup โหลดดักไว้ ป้องกันคลิกซ้ำ และให้รู้ว่าระบบทำงานอยู่
      Swal.fire({
        title: 'กำลังบันทึกข้อมูล...',
        text: 'กรุณารอสักครู่ ระบบกำลังส่งข้อมูลเข้า Sheet และแจ้งเตือนผ่าน LINE',
        allowOutsideClick: false,
        showConfirmButton: false,
        didOpen: () => {
          Swal.showLoading();
        }
      });

      const formData = {
        contactType: contactType,
        targetRoom: targetRoom || '-',
        vehicleType: document.querySelector('input[name="vehicleType"]:checked').value,
        licensePlate: document.getElementById('licensePlate').value || 'ไม่ระบุ',
        purpose: document.querySelector('input[name="purpose"]:checked').value,
        image: currentPhoto
      };

      if (typeof google === 'undefined') return; // กันพังตอนพรีวิว

      google.script.run
        .withSuccessHandler(res => {
          btn.disabled = false;
          Swal.close(); // ปิด Popup โหลด
          
          if(res.success) {
            showSlip(res);
          } else {
            Swal.fire('ผิดพลาดจากระบบหลังบ้าน', res.error, 'error');
          }
        })
        .withFailureHandler(err => {
          btn.disabled = false;
          Swal.close(); // ปิด Popup โหลด
          Swal.fire('ข้อผิดพลาดการเชื่อมต่อ', 'ไม่สามารถบันทึกได้ กรุณาลองใหม่อีกครั้ง', 'error');
        })
        .saveVisitor(formData);
    }

    function showSlip(res) {
      document.getElementById('slip-id').innerText = res.id;
      document.getElementById('slip-room').innerText = res.data[5];
      document.getElementById('slip-vehicle').innerText = `${res.data[9]} (${res.data[10]})`;
      document.getElementById('slip-time').innerText = res.data[1] + ' ' + res.data[2];
      document.getElementById('slip-purpose').innerText = res.data[7];
      
      const qrUrl = `https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=${res.id}`;
      document.getElementById('qrcode').innerHTML = `<img src="${qrUrl}" alt="QR" class="w-32 h-32">`;
      showPage('slip');
    }

    function showPage(pageId) {
      document.querySelectorAll('.page').forEach(p => p.classList.add('hidden'));
      document.getElementById('page-' + pageId).classList.remove('hidden');
      
      const pages = ['check-in', 'check-out', 'register'];
      pages.forEach(p => {
        const nav = document.getElementById('nav-' + p);
        if (nav) {
          if (p === pageId) {
            nav.classList.add('border-emerald-500', 'bg-emerald-50', 'font-bold');
            nav.classList.remove('border-transparent', 'bg-white', 'font-medium');
          } else {
            nav.classList.remove('border-emerald-500', 'bg-emerald-50', 'font-bold');
            nav.classList.add('border-transparent', 'bg-white', 'font-medium');
          }
        }
      });

      if(pageId === 'check-in') {
        document.getElementById('checkInForm').reset();
        document.getElementById('imgPreview').classList.add('hidden');
        currentPhoto = "";
        toggleRoomInput();
      } else if(pageId === 'check-out') {
        document.getElementById('checkoutInfo').classList.add('hidden');
        document.getElementById('searchId').value = '';
        loadActiveVisitors();
      } else if(pageId === 'register') {
        document.getElementById('regQrResult').classList.add('hidden');
        document.getElementById('regTargetRoom').value = '';
      }
    }

    let activeVisitorsTimeout;
    function loadActiveVisitors() {
      const container = document.getElementById('activeListContainer');
      container.innerHTML = `
        <div class="flex justify-center items-center py-6">
          <svg class="animate-spin h-6 w-6 text-emerald-500 mr-2" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <span class="text-sm text-gray-500">กำลังดึงข้อมูลล่าสุด...</span>
        </div>
      `;
      
      clearTimeout(activeVisitorsTimeout);
      activeVisitorsTimeout = setTimeout(() => {
        if (container.innerHTML.includes('กำลังดึงข้อมูลล่าสุด')) {
          container.innerHTML = '<div class="p-4 border border-dashed border-red-200 rounded-xl bg-red-50 text-center"><p class="text-sm text-red-500">❌ โหลดนานผิดปกติ กรุณารีเฟรช</p></div>';
        }
      }, 10000); 

      if (typeof google === 'undefined') return;

      google.script.run
        .withSuccessHandler((data) => {
           clearTimeout(activeVisitorsTimeout);
           renderActiveVisitors(data);
        })
        .withFailureHandler(function(error) {
           clearTimeout(activeVisitorsTimeout);
           container.innerHTML = `<div class="p-4 border border-dashed border-red-200 rounded-xl bg-red-50 text-center"><p class="text-sm text-red-500">❌ ดึงข้อมูลล้มเหลว</p></div>`;
        })
        .getActiveVisitors();
    }

    function renderActiveVisitors(data) {
      const container = document.getElementById('activeListContainer');
      if(!data || data.length === 0) {
        container.innerHTML = '<div class="p-4 border border-dashed rounded-xl bg-gray-50 text-center"><p class="text-sm text-gray-400">ไม่มีรถตกค้างในโครงการ</p></div>';
        return;
      }
      
      let html = '';
      data.forEach(v => {
        let statusBadge = v.status === 'Stamped' 
          ? '<span class="text-[10px] bg-green-100 text-green-700 px-2 py-0.5 rounded-full font-bold">ประทับตราแล้ว</span>' 
          : '<span class="text-[10px] bg-orange-100 text-orange-700 px-2 py-0.5 rounded-full font-bold">รอประทับตรา</span>';
          
        html += `
          <div class="p-3 border rounded-xl flex justify-between items-center bg-white shadow-sm hover:border-emerald-300 transition-all cursor-pointer" onclick="selectActiveVisitor('${v.id}')">
            <div class="flex-1">
              <div class="flex items-center space-x-2">
                <p class="font-bold text-emerald-700 text-sm">${v.licensePlate}</p>
                ${statusBadge}
              </div>
              <p class="text-xs text-gray-400 mt-1">เวลาเข้า: ${v.time} | ห้อง: <span class="font-medium text-gray-600">${v.room}</span></p>
            </div>
            <button type="button" class="text-xs bg-emerald-50 text-emerald-600 px-3 py-2 rounded-lg font-bold hover:bg-emerald-100 ml-2">เลือก</button>
          </div>
        `;
      });
      container.innerHTML = html;
    }

    function selectActiveVisitor(id) {
      document.getElementById('searchId').value = id;
      searchVisitor();
    }

    function searchVisitor() {
      const id = document.getElementById('searchId').value;
      if(!id) return Swal.fire('แจ้งเตือน', 'กรุณาระบุ ID ก่อนตรวจสอบ', 'warning');
      
      Swal.fire({
        title: 'กำลังตรวจสอบข้อมูล...',
        allowOutsideClick: false,
        didOpen: () => { Swal.showLoading(); }
      });
      
      if (typeof google === 'undefined') return;

      google.script.run
        .withSuccessHandler(v => {
          Swal.close();
          if(v) {
            const info = document.getElementById('checkoutInfo');
            const badge = document.getElementById('co-status'); 
            info.classList.remove('hidden');
            
            document.getElementById('co-room').innerText = v.room;
            document.getElementById('co-date').innerText = v.date;
            document.getElementById('co-time').innerText = v.time;
            document.getElementById('co-contactType').innerText = v.contactType;
            document.getElementById('co-vType').innerText = v.vehicleType;
            document.getElementById('co-lPlate').innerText = v.licensePlate;
            
            const photoContainer = document.getElementById('co-photo-container');
            if(v.photoUrl) {
              document.getElementById('co-photo').src = v.photoUrl;
              photoContainer.classList.remove('hidden');
            } else {
              photoContainer.classList.add('hidden');
            }
            
            currentVisitorId = v.id;
            
            if(v.status === 'Stamped') {
              badge.innerText = "ประทับตราแล้ว";
              badge.className = "inline-block px-3 py-1 rounded-full text-xs font-bold mb-2 bg-green-100 text-green-700";
              document.getElementById('btnCheckout').classList.remove('hidden');
            } else if(v.status === 'Checked-Out') {
              badge.innerText = "ออกจากโครงการไปแล้ว";
              badge.className = "inline-block px-3 py-1 rounded-full text-xs font-bold mb-2 bg-gray-100 text-gray-700";
              document.getElementById('btnCheckout').classList.add('hidden');
            } else {
              badge.innerText = "ยังไม่ประทับตรา";
              badge.className = "inline-block px-3 py-1 rounded-full text-xs font-bold mb-2 bg-red-100 text-red-700";
              document.getElementById('btnCheckout').classList.remove('hidden');
            }
          } else {
            Swal.fire('ไม่พบข้อมูล', 'ไม่พบรหัสผู้ติดต่อนี้', 'error');
          }
        })
        .withFailureHandler(err => {
          Swal.close();
          Swal.fire('ข้อผิดพลาด', 'ไม่สามารถเชื่อมต่อฐานข้อมูลได้', 'error');
        })
        .getVisitorById(id);
    }

    function handleCheckout() {
      Swal.fire({
        title: 'ยืนยันนำรถออก?',
        icon: 'question',
        showCancelButton: true,
        confirmButtonColor: '#10b981',
        cancelButtonColor: '#d33',
        confirmButtonText: 'ยืนยัน',
        cancelButtonText: 'ยกเลิก'
      }).then((result) => {
        if (result.isConfirmed) {
          Swal.fire({
            title: 'กำลังบันทึกข้อมูล...',
            allowOutsideClick: false,
            didOpen: () => { Swal.showLoading(); }
          });
          
          if (typeof google === 'undefined') return;

          google.script.run
            .withSuccessHandler(() => {
              Swal.close();
              Swal.fire('สำเร็จ', 'บันทึกเวลาออกเรียบร้อย', 'success');
              showPage('check-in');
              document.getElementById('checkoutInfo').classList.add('hidden');
              document.getElementById('searchId').value = '';
            })
            .withFailureHandler(err => {
              Swal.close();
              Swal.fire('ข้อผิดพลาด', 'บันทึกข้อมูลล้มเหลว', 'error');
            })
            .updateVisitorStatus(currentVisitorId, 'Checked-Out');
        }
      });
    }

    function showStampPage(id) {
      if (typeof google === 'undefined') return;
      google.script.run.withSuccessHandler(v => {
        if(v) {
          document.getElementById('nav').style.display = 'none';
          document.getElementById('stamp-room').innerText = v.room;
          document.getElementById('stamp-img').src = v.photoUrl || "https://dummyimage.com/600x400/cccccc/ffffff&text=No+Image";
          showPage('stamp');
        }
      }).getVisitorById(id);
    }

    function submitStamp(status) {
      if (typeof google === 'undefined') return;
      google.script.run.withSuccessHandler(() => {
        document.getElementById('stamp-content').innerHTML = `
          <div class="py-10">
            <div class="text-6xl mb-4 text-emerald-500">✅</div>
            <h3 class="text-2xl font-bold text-emerald-700">บันทึกสำเร็จ</h3>
            <p class="text-gray-500 mt-2">ขอบคุณที่ใช้บริการ MY Guest</p>
          </div>
        `;
      }).updateVisitorStatus(currentVisitorId, status);
    }

    function generateRegQR() {
      const room = document.getElementById('regTargetRoom').value;
      const botId = LINE_BOT_ID; 
      
      if (!room) return Swal.fire('แจ้งเตือน', 'กรุณาระบุบ้านเลขที่ ที่ต้องการสร้าง QR Code', 'warning');
      
      const textMessage = "#REG-" + room;
      const lineUrl = `https://line.me/R/oaMessage/${botId}/?${encodeURIComponent(textMessage)}`;
      const qrApiUrl = `https://api.qrserver.com/v1/create-qr-code/?size=250x250&data=${encodeURIComponent(lineUrl)}`;
      
      document.getElementById('regQrImage').src = qrApiUrl;
      document.getElementById('regRoomDisplay').innerText = `ผูกบัญชีบ้านเลขที่: ${room}`;
      document.getElementById('regQrResult').classList.remove('hidden');
    }
  </script>
</body>
</html>
