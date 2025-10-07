# 🧩 Team 2 — File and Network-related Handlers  
**Project:** ITDS362 – Unit Test for Open-Source Software (args4j)  
**Members:** Group 5  
**ISP Technique Used:** ECC (Equivalence Class Coverage)

---

## 🔹 Test Suite 1 – FileOptionHandlerTest

### 🎯 Objective
ทดสอบ `FileOptionHandler` ซึ่งทำหน้าที่แปลงค่า argument จาก command line ให้เป็น object `java.io.File`  
เพื่อยืนยันว่าระบบสามารถจัดการ path ที่มีอยู่จริง, ไม่มีอยู่จริง, และ path ที่เป็น directory หรือมีช่องว่างได้ถูกต้อง

---

### 🔸 Characteristics
- **Interface-based characteristic:** ชนิดข้อมูลของ argument เป็น string path ของไฟล์หรือโฟลเดอร์  
- **Functionality-based characteristic:** การ map ค่าจาก argument สู่ field `File` และตรวจสอบการสร้าง object ที่ถูกต้องโดยไม่ throw exception  

---

### 🔹 Input Domain Modeling (IDM)

| ขั้นตอน | รายละเอียด |
|----------|-------------|
| **a. Identify testable function** | `CmdLineParser.parseArgument(String... args)` |
| **b. Identify parameters / returns / exceptions** | Parameter: `args[]`; Return: void; Exception: `CmdLineException` |
| **c. Model the input domain** | แบ่งเป็น 2 equivalence classes: <br>• Class 1 – path ที่มีอยู่จริง (valid) <br>• Class 2 – path ที่ไม่มีอยู่ (invalid) |
| **d. Combine partitions → Test Requirements** | ใช้เทคนิค **ECC (Equivalence Class Coverage)** เพื่อทดสอบอย่างน้อยหนึ่งค่าในแต่ละ class |
| **e. Test Values** | `["-f", "existing_file.txt"]`, `["-f", "fake_path.txt"]` |
| **f. Expected Values** | เคส valid → `File.exists() = true` <br> เคส invalid → `File.exists() = false` แต่ยังได้ object `File` |

---

### ✅ Test Summary
| Case | Input | Expected Output | Result |
|------|--------|-----------------|---------|
| Valid file path | `-f existing.txt` | สร้าง object File และ `exists()==true` | ✅ |
| Valid directory | `-f <directory>` | object File ที่เป็น directory | ✅ |
| Invalid path | `-f fake.txt` | object File แต่ `exists()==false` | ✅ |

---

## 🔹 Test Suite 2 – URLOptionHandlerTest

### 🎯 Objective
ทดสอบ `URLOptionHandler` ซึ่งแปลง string argument ให้เป็น `java.net.URL`  
เพื่อยืนยันว่าการ parse URL ถูกต้องตามรูปแบบ และสามารถจับ `MalformedURLException` ได้ในกรณี URL ไม่ถูกต้อง

---

### 🔸 Characteristics
- **Interface-based characteristic:** argument เป็น string ที่แสดง URL (มีหรือไม่มี protocol)  
- **Functionality-based characteristic:** ตรวจสอบการสร้าง URL object และการโยน `CmdLineException` เมื่อรูปแบบไม่ถูกต้อง  

---

### 🔹 Input Domain Modeling (IDM)

| ขั้นตอน | รายละเอียด |
|----------|-------------|
| **a. Identify testable function** | `CmdLineParser.parseArgument(String... args)` |
| **b. Identify parameters / returns / exceptions** | Parameter: `args[]`; Return: void; Exception: `CmdLineException` (เกิดจาก `MalformedURLException`) |
| **c. Model the input domain** | แบ่งเป็น 2 equivalence classes: <br>• Class 1 – URL ที่ถูกต้อง (valid) <br>• Class 2 – URL ที่ผิดรูปแบบ (invalid) |
| **d. Combine partitions → Test Requirements** | ใช้เทคนิค **ECC (Equivalence Class Coverage)** เพื่อทดสอบอย่างน้อยหนึ่งค่าในแต่ละ class |
| **e. Test Values** | `["-u","https://example.com"]` (valid), `["-u","example.com"]` (invalid), `["-u","htp://wrong.com"]` (invalid) |
| **f. Expected Values** | เคส valid → สร้าง URL object สำเร็จ <br> เคส invalid → `CmdLineException` ถูกโยนออกมา |

---

### ✅ Test Summary
| Case | Input | Expected Output | Result |
|------|--------|-----------------|---------|
| Valid URL | `-u https://example.com` | สร้าง URL object ได้ถูกต้อง | ✅ |
| URL with path/query | `-u https://example.com/api?q=1#frag` | host/path/query/ref ครบ | ✅ |
| Invalid URL (no protocol) | `-u example.com` | `CmdLineException` (MalformedURLException ภายใน) | ✅ |
| Invalid protocol | `-u htp://wrong.com` | `CmdLineException` | ✅ |
| No host | `-u http://` | `CmdLineException` | ✅ |

---

## 📘 Summary of Team 2
| Test Suite | Class Under Test | ISP Technique | Valid Class | Invalid Class | Expected Exception |
|-------------|-----------------|----------------|--------------|----------------|--------------------|
| FileOptionHandlerTest | `FileOptionHandler` | ECC | path มีอยู่จริง | path ไม่อยู่ | ❌ |
| URLOptionHandlerTest | `URLOptionHandler` | ECC | URL ถูกต้อง | URL ผิดรูปแบบ | ✅ CmdLineException |

---

### 🪶 License Header (ต้องอยู่บนสุดของทุกไฟล์)
```java
/* Copyright (C) 2025 Team 2
 * You may use, distribute and modify this code under the terms of the MIT license.
 */
