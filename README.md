<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>My To-Do App</title>
    <style>
        /* ส่วนของการตกแต่งหน้าตาแอป (CSS) */
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            background-color: #f0f2f5; 
            margin: 0; 
            padding: 20px; 
            display: flex; 
            justify-content: center; 
        }
        .app-container { 
            background: white; 
            width: 100%; 
            max-width: 400px; /* ขนาดพอดีกับหน้าจอมือถือ */
            padding: 25px; 
            border-radius: 20px; 
            box-shadow: 0 10px 20px rgba(0,0,0,0.1); 
        }
        h2 { text-align: center; color: #1a1a1a; margin-top: 0; }
        input[type="text"] { 
            width: calc(100% - 24px); 
            padding: 12px; 
            margin-bottom: 15px; 
            border: 2px solid #e1e4e8; 
            border-radius: 10px; 
            font-size: 16px; 
            outline: none;
        }
        input[type="text"]:focus { border-color: #007bff; }
        button.add-btn { 
            width: 100%; 
            padding: 12px; 
            background-color: #007bff; 
            color: white; 
            border: none; 
            border-radius: 10px; 
            font-size: 16px; 
            font-weight: bold;
            cursor: pointer; 
        }
        button.add-btn:active { background-color: #0056b3; }
        ul { list-style-type: none; padding: 0; margin-top: 25px; }
        li { 
            background: #f8f9fa; 
            margin-bottom: 10px; 
            padding: 12px 15px; 
            border-radius: 10px; 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            border-left: 5px solid #007bff; 
            font-size: 16px;
        }
        .delete-btn { 
            background: #ff4d4d; 
            color: white;
            border: none;
            padding: 6px 12px; 
            border-radius: 6px;
            font-size: 14px; 
            cursor: pointer;
        }
        .delete-btn:active { background: #cc0000; }
    </style>
</head>
<body>

<div class="app-container">
    <h2>📝 รายการสิ่งที่ต้องทำ</h2>
    <input type="text" id="taskInput" placeholder="พิมพ์งานที่ต้องทำตรงนี้...">
    <button class="add-btn" onclick="addTask()">+ เพิ่มรายการ</button>
    
    <ul id="taskList">
        </ul>
</div>

<script>
    // ส่วนของการทำงาน (JavaScript)
    function addTask() {
        let input = document.getElementById("taskInput");
        let taskText = input.value.trim();
        
        // ถ้าไม่ได้พิมพ์อะไรให้หยุดการทำงาน
        if (taskText === "") {
            alert("กรุณาพิมพ์รายการก่อนครับ!");
            return;
        }

        // สร้างรายการใหม่ (li)
        let li = document.createElement("li");
        li.innerHTML = `${taskText} <button class="delete-btn" onclick="this.parentElement.remove()">ลบ</button>`;
        
        // นำรายการใหม่ไปแสดงผล
        document.getElementById("taskList").appendChild(li);
        
        // ล้างช่องพิมพ์ข้อความ
        input.value = "";
    }
</script>

</body>
</html>
