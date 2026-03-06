import React, { useState, useEffect, useRef } from 'react';
import { Camera, Upload, QrCode, Search, CheckCircle, XCircle, Printer, Car, User, Clock, FileText, ArrowLeft, Send } from 'lucide-react';

// --- จำลองข้อมูลพื้นฐาน (Mock Data) ---
const MOCK_RESIDENTS = ['101/1', '101/2', '101/3', '102/1', '102/2', '201/1', '201/2', '999/9 (นิติบุคคล)'];

// --- โครงสร้างแอปพลิเคชันหลัก ---
export default function App() {
  const [scriptsLoaded, setScriptsLoaded] = useState(false);
  const [view, setView] = useState('checkin'); // 'checkin', 'slip', 'resident', 'checkout'
  const [visitorLogs, setVisitorLogs] = useState([]);
  const [currentSlipId, setCurrentSlipId] = useState(null);

  // โหลด Libraries สำหรับ QR Code (เนื่องจากเป็น Single File Component)
  useEffect(() => {
    const loadScript = (src) => new Promise((resolve) => {
      if (document.querySelector(`script[src="${src}"]`)) return resolve();
      const script = document.createElement('script');
      script.src = src;
      script.onload = resolve;
      document.head.appendChild(script);
    });

    Promise.all([
      loadScript('https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js'),
      loadScript('https://unpkg.com/html5-qrcode')
    ]).then(() => setScriptsLoaded(true));
  }, []);

  if (!scriptsLoaded) return <div className="flex justify-center items-center h-screen">กำลังโหลดระบบ...</div>;

  // ฟังก์ชันช่วยเหลือสำหรับฟอร์แมตวันที่
  const getCurrentDateTime = () => {
    const now = new Date();
    return now.toLocaleString('th-TH', { dateStyle: 'short', timeStyle: 'medium' });
  };

  // --- การนำทาง (Navigation) ---
  const Nav = () => (
    <div className="bg-blue-900 text-white p-4 shadow-md flex justify-between items-center print:hidden">
      <div className="flex items-center gap-2 font-bold text-xl">
        <Car /> MyGuest VMS
      </div>
      <div className="flex gap-4">
        <button onClick={() => setView('checkin')} className={`px-3 py-1 rounded ${view === 'checkin' ? 'bg-blue-700' : 'hover:bg-blue-800'}`}>รถเข้า (Check-in)</button>
        <button onClick={() => setView('checkout')} className={`px-3 py-1 rounded ${view === 'checkout' ? 'bg-blue-700' : 'hover:bg-blue-800'}`}>รถออก (Check-out)</button>
      </div>
    </div>
  );

  return (
    <div className="min-h-screen bg-gray-100 font-sans">
      {view !== 'slip' && view !== 'resident' && <Nav />}
      
      <main className="p-4 max-w-md mx-auto print:p-0 print:max-w-none">
        {view === 'checkin' && (
          <CheckInView 
            onSave={(data) => {
              setVisitorLogs([...visitorLogs, data]);
              setCurrentSlipId(data.id);
              setView('slip');
              
              // จำลองการส่ง LINE
              if(data.contactType === 'room') {
                alert(`[จำลองระบบ] ส่ง LINE Flex Message ไปยังเจ้าของห้อง ${data.roomNumber} แล้ว\n"มีผู้มาติดต่อ ทะเบียน: ${data.licensePlate}"`);
              }
            }} 
            getCurrentDateTime={getCurrentDateTime} 
          />
        )}
        {view === 'slip' && (
          <SlipView 
            visitor={visitorLogs.find(v => v.id === currentSlipId)} 
            onClose={() => setView('checkin')}
            onSimulateResident={() => setView('resident')}
          />
        )}
        {view === 'resident' && (
          <ResidentApprovalView 
            visitor={visitorLogs.find(v => v.id === currentSlipId)} 
            onUpdateStatus={(id, status) => {
              setVisitorLogs(visitorLogs.map(v => v.id === id ? { ...v, status: status } : v));
              alert(`บันทึกสถานะ: ${status === 'approved' ? 'อนุมัติ' : 'ปฏิเสธ'} เรียบร้อยแล้ว`);
              setView('checkin');
            }}
            onCancel={() => setView('checkin')}
          />
        )}
        {view === 'checkout' && (
          <CheckOutView 
            visitorLogs={visitorLogs} 
            onCheckOut={(id) => {
              setVisitorLogs(visitorLogs.map(v => v.id === id ? { ...v, timeOut: getCurrentDateTime() } : v));
            }}
          />
        )}
      </main>

      {/* สไตล์สำหรับการพิมพ์สลิป (Thermal Printer) */}
      <style dangerouslySetInnerHTML={{__html: `
        @media print {
          body * { visibility: hidden; }
          .print-area, .print-area * { visibility: visible; }
          .print-area { position: absolute; left: 0; top: 0; width: 100%; max-width: 80mm; margin: 0; padding: 10px; font-size: 12px; }
          .print-hidden { display: none !important; }
        }
      `}} />
    </div>
  );
}

// ==========================================
// 1. หน้าต่างบันทึกผู้มาติดต่อ (Check-in)
// ==========================================
const CheckInView = ({ onSave, getCurrentDateTime }) => {
  const [formData, setFormData] = useState({
    contactType: 'room',
    roomNumber: '',
    contactOther: '',
    purpose: 'relative',
    purposeOther: '',
    licensePlate: '',
    photoBase64: null
  });

  const [searchRoom, setSearchRoom] = useState('');
  const [showRoomDropdown, setShowRoomDropdown] = useState(false);
  const [cameraActive, setCameraActive] = useState(false);
  const videoRef = useRef(null);
  const canvasRef = useRef(null);

  // Auto-update time display
  const [timeDisplay, setTimeDisplay] = useState(getCurrentDateTime());
  useEffect(() => {
    const timer = setInterval(() => setTimeDisplay(getCurrentDateTime()), 1000);
    return () => clearInterval(timer);
  }, []);

  const handleCamera = async () => {
    if (cameraActive) {
      // Capture
      const video = videoRef.current;
      const canvas = canvasRef.current;
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      canvas.getContext('2d').drawImage(video, 0, 0);
      const photo = canvas.toDataURL('image/jpeg');
      setFormData({ ...formData, photoBase64: photo });
      
      // Stop tracks
      video.srcObject.getTracks().forEach(track => track.stop());
      setCameraActive(false);
    } else {
      // Start
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
        videoRef.current.srcObject = stream;
        setCameraActive(true);
      } catch (err) {
        alert('ไม่สามารถเปิดกล้องได้ กรุณาใช้วิธีอัปโหลดรูปภาพ');
      }
    }
  };

  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setFormData({ ...formData, photoBase64: reader.result });
      };
      reader.readAsDataURL(file);
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!formData.licensePlate) return alert('กรุณาระบุเลขทะเบียนรถ');
    if (formData.contactType === 'room' && !formData.roomNumber) return alert('กรุณาระบุบ้านเลขที่/ห้องชุด');

    const newLog = {
      id: 'V' + Date.now().toString().slice(-6),
      timeIn: timeDisplay,
      timeOut: null,
      status: 'pending', // pending, approved, rejected
      ...formData
    };
    onSave(newLog);
  };

  const filteredRooms = MOCK_RESIDENTS.filter(r => r.includes(searchRoom));

  return (
    <form onSubmit={handleSubmit} className="bg-white p-6 rounded-lg shadow-lg space-y-6">
      <h2 className="text-2xl font-bold text-gray-800 border-b pb-2">บันทึกรถเข้า (Check-in)</h2>

      {/* วันที่และเวลา */}
      <div className="bg-gray-100 p-3 rounded text-center font-mono text-lg text-gray-700">
        {timeDisplay}
      </div>

      {/* ผู้ที่มาติดต่อ */}
      <div className="space-y-3">
        <label className="font-semibold text-gray-700 block">ผู้ที่มาติดต่อ</label>
        <div className="grid grid-cols-2 gap-2">
          {['room:บ้านเลขที่/ห้องชุด', 'office:สำนักงานนิติฯ', 'tour:เยี่ยมชมโครงการ', 'other:อื่นๆ'].map(item => {
            const [val, label] = item.split(':');
            return (
              <label key={val} className={`p-2 border rounded flex items-center gap-2 cursor-pointer ${formData.contactType === val ? 'bg-blue-50 border-blue-500' : ''}`}>
                <input type="radio" name="contactType" value={val} checked={formData.contactType === val} onChange={(e) => setFormData({...formData, contactType: e.target.value})} className="hidden" />
                <div className={`w-4 h-4 rounded-full border flex-shrink-0 ${formData.contactType === val ? 'border-4 border-blue-600' : 'border-gray-400'}`}></div>
                <span className="text-sm">{label}</span>
              </label>
            )
          })}
        </div>

        {formData.contactType === 'room' && (
          <div className="relative mt-2">
            <input 
              type="text" placeholder="พิมพ์ค้นหาบ้านเลขที่/ห้องชุด..." className="w-full p-2 border rounded focus:ring-2 focus:ring-blue-500"
              value={searchRoom} onChange={(e) => { setSearchRoom(e.target.value); setShowRoomDropdown(true); }}
              onFocus={() => setShowRoomDropdown(true)}
            />
            {showRoomDropdown && searchRoom && (
              <ul className="absolute z-10 w-full bg-white border rounded shadow-lg max-h-40 overflow-y-auto">
                {filteredRooms.length > 0 ? filteredRooms.map(room => (
                  <li key={room} className="p-2 hover:bg-blue-100 cursor-pointer" onClick={() => { setFormData({...formData, roomNumber: room}); setSearchRoom(room); setShowRoomDropdown(false); }}>
                    {room}
                  </li>
                )) : <li className="p-2 text-gray-500 text-sm">ไม่พบข้อมูล (พิมพ์เพื่อเพิ่มใหม่ได้)</li>}
              </ul>
            )}
          </div>
        )}

        {formData.contactType === 'other' && (
          <input type="text" placeholder="ระบุรายละเอียด..." className="w-full p-2 border rounded mt-2" onChange={(e) => setFormData({...formData, contactOther: e.target.value})} required />
        )}
      </div>

      {/* ทะเบียนรถ & รูปภาพ */}
      <div className="space-y-3">
        <label className="font-semibold text-gray-700 block">ทะเบียนรถ</label>
        <input type="text" placeholder="เช่น กข 1234 กทม" className="w-full p-3 border rounded text-lg uppercase font-bold text-center" value={formData.licensePlate} onChange={(e) => setFormData({...formData, licensePlate: e.target.value})} required />
        
        <div className="border-2 border-dashed rounded-lg p-4 text-center">
          {formData.photoBase64 ? (
            <div className="relative">
              <img src={formData.photoBase64} alt="License Plate" className="w-full rounded" />
              <button type="button" onClick={() => setFormData({...formData, photoBase64: null})} className="absolute top-2 right-2 bg-red-500 text-white p-1 rounded-full"><XCircle size={20}/></button>
            </div>
          ) : cameraActive ? (
            <div className="relative">
              <video ref={videoRef} autoPlay playsInline className="w-full rounded bg-black"></video>
              <button type="button" onClick={handleCamera} className="mt-2 bg-blue-600 text-white px-4 py-2 rounded-full absolute bottom-4 left-1/2 transform -translate-x-1/2 flex items-center gap-2"><Camera size={20}/> ถ่ายรูป</button>
            </div>
          ) : (
            <div className="flex gap-2 justify-center">
              <button type="button" onClick={handleCamera} className="flex-1 bg-gray-200 p-3 rounded flex flex-col items-center justify-center hover:bg-gray-300">
                <Camera className="mb-1" /> เปิดกล้อง
              </button>
              <label className="flex-1 bg-gray-200 p-3 rounded flex flex-col items-center justify-center hover:bg-gray-300 cursor-pointer">
                <Upload className="mb-1" /> อัปโหลด
                <input type="file" accept="image/*" capture="environment" className="hidden" onChange={handleFileUpload} />
              </label>
            </div>
          )}
          <canvas ref={canvasRef} className="hidden"></canvas>
        </div>
      </div>

      {/* วัตถุประสงค์ */}
      <div className="space-y-3">
        <label className="font-semibold text-gray-700 block">วัตถุประสงค์</label>
        <select className="w-full p-3 border rounded bg-white" value={formData.purpose} onChange={(e) => setFormData({...formData, purpose: e.target.value})}>
          <option value="relative">พบญาติ / เพื่อน</option>
          <option value="appointment">นัดหมายไว้</option>
          <option value="delivery">ส่งของ / รับส่งอาหาร</option>
          <option value="contractor">ตกแต่ง / ต่อเติม / ช่าง</option>
          <option value="other">อื่นๆ</option>
        </select>
        {formData.purpose === 'other' && (
          <input type="text" placeholder="ระบุวัตถุประสงค์..." className="w-full p-2 border rounded" onChange={(e) => setFormData({...formData, purposeOther: e.target.value})} required />
        )}
      </div>

      {/* ปุ่มบันทึก */}
      <button type="submit" className="w-full bg-green-600 text-white text-lg font-bold py-4 rounded-lg shadow-lg hover:bg-green-700 flex justify-center items-center gap-2">
        <CheckCircle /> บันทึกข้อมูลเข้า
      </button>
    </form>
  );
};


// ==========================================
// 2. หน้าแสดงสลิป (Slip & Print)
// ==========================================
const SlipView = ({ visitor, onClose, onSimulateResident }) => {
  const qrRef = useRef(null);

  useEffect(() => {
    if (qrRef.current && visitor) {
      qrRef.current.innerHTML = ''; // Clear old QR
      new window.QRCode(qrRef.current, {
        text: visitor.id,
        width: 150,
        height: 150,
        colorDark : "#000000",
        colorLight : "#ffffff",
        correctLevel : window.QRCode.CorrectLevel.H
      });
    }
  }, [visitor]);

  if (!visitor) return null;

  return (
    <div className="bg-white min-h-screen flex flex-col">
      {/* ส่วนหัวสำหรับการทำงานของ รปภ. (ไม่พิมพ์) */}
      <div className="p-4 flex justify-between items-center bg-gray-100 border-b print-hidden">
        <button onClick={onClose} className="text-gray-600 flex items-center gap-1"><ArrowLeft size={16}/> กลับ</button>
        <button onClick={() => window.print()} className="bg-blue-600 text-white px-4 py-2 rounded flex items-center gap-2">
          <Printer size={18}/> พิมพ์สลิป
        </button>
      </div>

      {/* พื้นที่สำหรับพิมพ์สลิป */}
      <div className="print-area p-8 flex-1 flex flex-col items-center max-w-sm mx-auto w-full">
        <div className="text-center mb-6 w-full border-b-2 border-dashed border-gray-400 pb-4">
          <Car size={40} className="mx-auto text-gray-800 mb-2" />
          <h1 className="text-2xl font-bold uppercase tracking-wider">My Guest</h1>
          <p className="text-sm text-gray-500">Visitor Pass</p>
        </div>

        <div className="w-full space-y-3 text-sm mb-6">
          <div className="flex justify-between border-b border-gray-100 pb-1">
            <span className="text-gray-500">ID:</span>
            <span className="font-mono font-bold">{visitor.id}</span>
          </div>
          <div className="flex justify-between border-b border-gray-100 pb-1">
            <span className="text-gray-500">ทะเบียนรถ:</span>
            <span className="font-bold text-lg">{visitor.licensePlate}</span>
          </div>
          <div className="flex justify-between border-b border-gray-100 pb-1">
            <span className="text-gray-500">ติดต่อ:</span>
            <span className="font-semibold text-right max-w-[60%]">
              {visitor.contactType === 'room' ? `ห้อง ${visitor.roomNumber}` : 
               visitor.contactType === 'office' ? 'สำนักงานนิติฯ' : 
               visitor.contactType === 'tour' ? 'เยี่ยมชมโครงการ' : visitor.contactOther}
            </span>
          </div>
          <div className="flex justify-between border-b border-gray-100 pb-1">
            <span className="text-gray-500">เวลาเข้า:</span>
            <span className="">{visitor.timeIn}</span>
          </div>
        </div>

        {/* QR Code Area */}
        <div className="bg-white p-2 border rounded-lg shadow-sm mb-4">
          <div ref={qrRef}></div>
        </div>
        <p className="text-xs text-center text-gray-500 mb-8">สแกน QR Code นี้เพื่อประทับตราอนุมัติ<br/>หรือใช้สำหรับสแกนออก</p>

        {visitor.photoBase64 && (
          <div className="mt-4 border p-1 rounded print-hidden">
            <p className="text-xs text-gray-500 mb-1">รูปรถยนต์ (อ้างอิง)</p>
            <img src={visitor.photoBase64} alt="Car" className="w-full h-32 object-cover rounded" />
          </div>
        )}
      </div>

      {/* ปุ่มจำลองฝั่งลูกบ้าน (ไม่เกี่ยวกับการพิมพ์) */}
      <div className="p-4 bg-yellow-50 border-t print-hidden mt-auto">
        <p className="text-sm text-yellow-800 mb-2 font-semibold">เครื่องมือจำลองระบบ (Simulation):</p>
        <button onClick={onSimulateResident} className="w-full bg-green-500 text-white py-2 rounded shadow flex justify-center items-center gap-2">
          <User size={18}/> จำลองมุมมอง 'เจ้าของห้อง' กดลิงก์จาก LINE
        </button>
      </div>
    </div>
  );
};


// ==========================================
// 3 & 4. หน้าประทับตรา (Resident Approval - จำลอง LINE Flex/Web)
// ==========================================
const ResidentApprovalView = ({ visitor, onUpdateStatus, onCancel }) => {
  if (!visitor) return <div className="p-4 text-center">ไม่พบข้อมูลผู้ติดต่อ</div>;

  return (
    <div className="bg-gray-100 min-h-screen">
      <div className="bg-green-600 text-white p-4 shadow-md text-center font-bold text-lg">
        ระบบลูกบ้านประทับตรา
      </div>
      
      <div className="p-4 max-w-sm mx-auto mt-4">
        <div className="bg-white rounded-xl shadow-lg overflow-hidden">
          <div className="bg-blue-50 p-4 text-center border-b">
            <h2 className="text-blue-800 font-bold text-xl">มีผู้มาติดต่อห้องของท่าน</h2>
            <p className="text-sm text-blue-600 mt-1">กรุณายืนยันการเข้าพบ</p>
          </div>
          
          <div className="p-5 space-y-4">
            {visitor.photoBase64 && (
              <img src={visitor.photoBase64} alt="Car" className="w-full h-40 object-cover rounded-lg shadow-inner" />
            )}
            
            <div className="grid grid-cols-3 gap-2 text-sm border-b pb-3">
              <div className="text-gray-500">ทะเบียนรถ</div>
              <div className="col-span-2 font-bold text-lg text-right">{visitor.licensePlate}</div>
              
              <div className="text-gray-500">วัตถุประสงค์</div>
              <div className="col-span-2 text-right">
                {visitor.purpose === 'relative' ? 'พบญาติ/เพื่อน' : 
                 visitor.purpose === 'delivery' ? 'ส่งของ' : 
                 visitor.purpose === 'contractor' ? 'ช่าง/ตกแต่ง' : visitor.purposeOther || visitor.purpose}
              </div>
              
              <div className="text-gray-500">เวลาเข้า</div>
              <div className="col-span-2 text-right text-xs pt-1">{visitor.timeIn}</div>
            </div>

            {visitor.status === 'pending' ? (
              <div className="grid grid-cols-2 gap-3 pt-2">
                <button onClick={() => onUpdateStatus(visitor.id, 'rejected')} className="flex flex-col items-center justify-center p-3 border-2 border-red-500 text-red-600 rounded-lg hover:bg-red-50 font-bold">
                  <XCircle size={28} className="mb-1" />
                  ปฏิเสธ
                </button>
                <button onClick={() => onUpdateStatus(visitor.id, 'approved')} className="flex flex-col items-center justify-center p-3 bg-green-500 text-white rounded-lg hover:bg-green-600 shadow-md font-bold">
                  <CheckCircle size={28} className="mb-1" />
                  ประทับตรา
                </button>
              </div>
            ) : (
              <div className={`p-4 rounded-lg text-center font-bold flex flex-col items-center ${visitor.status === 'approved' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}`}>
                {visitor.status === 'approved' ? <CheckCircle size={32} className="mb-2"/> : <XCircle size={32} className="mb-2"/>}
                ท่านได้ {visitor.status === 'approved' ? 'อนุมัติประทับตรา' : 'ปฏิเสธ'} แล้ว
              </div>
            )}
          </div>
        </div>
        
        <button onClick={onCancel} className="mt-8 text-gray-500 w-full text-center text-sm underline">
          กลับสู่หน้าจอ รปภ.
        </button>
      </div>
    </div>
  );
};


// ==========================================
// 6. หน้าต่างตรวจสอบรถออก (Check-out)
// ==========================================
const CheckOutView = ({ visitorLogs, onCheckOut }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [scanning, setScanning] = useState(false);
  const scannerRef = useRef(null);

  // Active visitors (ยังไม่ออก)
  const activeVisitors = visitorLogs.filter(v => v.timeOut === null);
  
  // กรองตามคำค้นหา
  const filteredVisitors = activeVisitors.filter(v => 
    v.licensePlate.toLowerCase().includes(searchTerm.toLowerCase()) || 
    v.id.toLowerCase().includes(searchTerm.toLowerCase())
  );

  const startScanner = () => {
    setScanning(true);
    setTimeout(() => {
      const html5QrcodeScanner = new window.Html5QrcodeScanner("reader", { fps: 10, qrbox: {width: 250, height: 250} }, false);
      scannerRef.current = html5QrcodeScanner;
      html5QrcodeScanner.render((decodedText) => {
        setSearchTerm(decodedText);
        html5QrcodeScanner.clear();
        setScanning(false);
      }, (error) => {
        // Handle scan error silently
      });
    }, 100);
  };

  const stopScanner = () => {
    if (scannerRef.current) scannerRef.current.clear();
    setScanning(false);
  };

  const StatusBadge = ({ status }) => {
    switch(status) {
      case 'approved': return <span className="px-2 py-1 bg-green-100 text-green-700 text-xs rounded-full font-bold flex items-center gap-1"><CheckCircle size={12}/> ประทับตราแล้ว</span>;
      case 'rejected': return <span className="px-2 py-1 bg-red-100 text-red-700 text-xs rounded-full font-bold flex items-center gap-1"><XCircle size={12}/> ปฏิเสธการเข้าพบ</span>;
      default: return <span className="px-2 py-1 bg-yellow-100 text-yellow-700 text-xs rounded-full font-bold flex items-center gap-1"><Clock size={12}/> รอประทับตรา</span>;
    }
  };

  return (
    <div className="bg-white p-6 rounded-lg shadow-lg space-y-6">
      <h2 className="text-2xl font-bold text-gray-800 border-b pb-2 flex items-center gap-2">
        <Car /> ตรวจสอบรถออก (Check-out)
      </h2>

      {/* ค้นหา / สแกน */}
      <div className="flex gap-2">
        <div className="relative flex-1">
          <Search className="absolute left-3 top-3 text-gray-400" size={20} />
          <input 
            type="text" placeholder="ค้นหา ทะเบียน หรือ สแกน QR" 
            className="w-full pl-10 p-3 border rounded focus:ring-2 focus:ring-blue-500 uppercase"
            value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)}
          />
        </div>
        <button onClick={scanning ? stopScanner : startScanner} className={`px-4 rounded text-white flex items-center justify-center ${scanning ? 'bg-red-500' : 'bg-blue-600'}`}>
          <QrCode size={24} />
        </button>
      </div>

      {/* พื้นที่แสดงกล้องสแกน QR */}
      {scanning && (
        <div className="border-2 border-dashed border-gray-300 rounded overflow-hidden">
          <div id="reader" width="100%"></div>
        </div>
      )}

      {/* รายการรถที่ยังไม่ออก */}
      <div className="space-y-3 mt-4">
        <h3 className="text-sm font-semibold text-gray-500">รายการรถที่ยังอยู่ในโครงการ ({filteredVisitors.length})</h3>
        
        {filteredVisitors.length === 0 ? (
          <div className="text-center text-gray-400 py-8 border rounded border-dashed bg-gray-50">
            ไม่พบข้อมูลรถที่ตรงกับการค้นหา
          </div>
        ) : (
          filteredVisitors.map(visitor => (
            <div key={visitor.id} className="border rounded-lg p-4 shadow-sm relative overflow-hidden">
              {/* แถบสีบอกสถานะด้านซ้าย */}
              <div className={`absolute left-0 top-0 bottom-0 w-1 ${visitor.status === 'approved' ? 'bg-green-500' : visitor.status === 'rejected' ? 'bg-red-500' : 'bg-yellow-500'}`}></div>
              
              <div className="flex justify-between items-start pl-2">
                <div>
                  <div className="font-bold text-lg">{visitor.licensePlate}</div>
                  <div className="text-sm text-gray-600">ติดต่อ: {visitor.contactType === 'room' ? `ห้อง ${visitor.roomNumber}` : 'อื่นๆ'}</div>
                  <div className="text-xs text-gray-400 mt-1">เวลาเข้า: {visitor.timeIn}</div>
                </div>
                <div className="flex flex-col items-end gap-2">
                  <StatusBadge status={visitor.status} />
                  <span className="text-xs text-gray-400 font-mono">{visitor.id}</span>
                </div>
              </div>

              {/* ปุ่ม Action รถออก */}
              <div className="mt-4 pt-3 border-t flex gap-2">
                {visitor.status !== 'approved' && visitor.status !== 'rejected' && (
                  <div className="flex-1 text-xs text-yellow-600 bg-yellow-50 p-2 rounded text-center flex items-center justify-center">
                    ⚠️ ยังไม่ได้รับการประทับตราจากเจ้าของห้อง
                  </div>
                )}
                <button 
                  onClick={() => {
                    if(visitor.status !== 'approved') {
                      if(!window.confirm('รถคันนี้ยังไม่ได้รับการประทับตราอนุมัติ ยืนยันที่จะบันทึกรถออกใช่หรือไม่?')) return;
                    }
                    onCheckOut(visitor.id);
                  }} 
                  className={`flex-1 py-2 rounded font-bold text-white shadow flex justify-center items-center gap-1 ${visitor.status === 'approved' ? 'bg-blue-600 hover:bg-blue-700' : 'bg-gray-400 hover:bg-gray-500'}`}
                >
                  บันทึกรถออก
                </button>
              </div>
            </div>
          ))
        )}
      </div>
    </div>
  );
};
