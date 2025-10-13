**Project:** ITDS362 – Unit Test for Open-Source Software (args4j)  
**ISP Technique Used:** ECC (Equivalence Class Coverage)

---

## Test Suite 1 – FileOptionHandlerTest

### Objective
ทดสอบ `FileOptionHandler` เพื่อยืนยันว่าการ parse argument จาก command line  
สามารถสร้าง `File` object ได้ถูกต้องทั้งกรณี path ที่มีอยู่จริง ไม่มีอยู่จริง และมีช่องว่าง

### Characteristics
| ประเภท | รายละเอียด |
|----------|-------------|
| Interface-based | argument เป็น string ที่แทน path ของไฟล์หรือโฟลเดอร์ |
| Functionality-based | ตรวจสอบการ map ค่า argument → field `File` และการจัดการ path ทั้ง valid และ invalid |

### Input Domain Modeling (IDM)
| ขั้นตอน | รายละเอียด |
|----------|-------------|
| a | Identify testable function → `CmdLineParser.parseArgument(String... args)` |
| b | Identify parameters / returns / exceptions → args[], void, CmdLineException |
| c | Model the input domain → path valid / invalid |
| d | Combine partitions → ECC (เลือก 1 ค่าในแต่ละ class) |
| e | Test Values → `"existing_file.txt"`, `"fake_path.txt"`, `"./relative/file.txt"` |
| f | Expected Values → valid → exists()==true ; invalid → exists()==false แต่ไม่ throw exception |

### Test Summary
| Case | Input | Expected Output |
|------|--------|-----------------|
| Valid file path | `-f existing.txt` | File.exists()==true |
| Valid directory | `-f <directory>` | File.isDirectory()==true |
| Invalid path | `-f fake.txt` | File.exists()==false |
| Relative path | `-f ./test/file.txt` | Canonical path normalize ถูกต้อง |

---

## Test Suite 2 – URLOptionHandlerTest

### Objective
ทดสอบ `URLOptionHandler` เพื่อยืนยันว่าการ parse URL จาก command line  
สามารถสร้าง `URL` object ได้ถูกต้องทั้งกรณี URL ถูกและผิดรูปแบบ

### Characteristics
| ประเภท | รายละเอียด |
|----------|-------------|
| Interface-based | argument เป็น string ที่แทน URL |
| Functionality-based | ตรวจสอบการสร้าง URL object และพฤติกรรมเมื่อ protocol หรือ host ไม่ถูกต้อง |

### Input Domain Modeling (IDM)
| ขั้นตอน | รายละเอียด |
|----------|-------------|
| a | Identify testable function → `CmdLineParser.parseArgument(String... args)` |
| b | Identify parameters / returns / exceptions → args[], void, CmdLineException |
| c | Model the input domain → URL valid / invalid |
| d | Combine partitions → ECC |
| e | Test Values → `"https://example.com"`, `"https://example.com/api?q=1#frag"`, `"example.com"`, `"htp://wrong.com"`, `"http://"` |
| f | Expected Values → valid → URL object ถูกต้อง ; invalid → CmdLineException หรือ host=null |

### Test Summary
| Case | Input | Expected Output |
|------|--------|-----------------|
| Valid URL | `-u https://example.com` | URL object ถูกต้อง |
| Valid with path/query | `-u https://example.com/api?q=1#frag` | URL ครบทุกส่วน |
| Invalid (no protocol) | `-u example.com` | CmdLineException |
| Invalid (bad protocol) | `-u htp://wrong.com` | CmdLineException |
| Protocol without host | `-u http://` | host == null |

---

## Problem and Solutions
| ปัญหา | วิธีแก้ |
|--------|----------|
| Build Environment | ใช้ `mvn clean install` เพื่อ build จากซอร์สล่าสุด |
| Handler ไม่ถูกเรียกใช้ | หลัง build ใหม่ handler ถูกโหลดอัตโนมัติ |
| Test Case ผิดพฤติกรรม | แก้เทสให้ตรง behavior ของ `java.net.URL` (host=null) |

---
