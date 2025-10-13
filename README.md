# 🧩 Group 5 — args4j Unit Testing Project  
**Course:** ITDS362 – Software Quality Assurance and Testing  
**Project:** Unit Test for Open-Source Software  
**Target Project:** [kohsuke/args4j](https://github.com/kohsuke/args4j)

---

## Overview
โปรเจกต์นี้เป็นส่วนหนึ่งของวิชา **ITDS362 – Software Quality Assurance and Testing**  
โดยกลุ่ม 5 ได้เลือกโปรเจกต์โอเพนซอร์ส **args4j** เพื่อสร้างและออกแบบชุดทดสอบใหม่ (Test Suites)  
โดยใช้เทคนิค **Input Space Partitioning (ISP)** ที่แตกต่างกันในแต่ละส่วน เพื่อให้ครอบคลุมการทำงานของ Handlers หลายประเภท

---

# 🧩 Test Suite 1 – StringOptionHandlerTest  
**ISP Technique:** PWC (Pairwise Coverage)

### Objective
ทดสอบคลาส `StringOptionHandler` เพื่อยืนยันว่าการรับค่า string argument  
สามารถ parse ได้ถูกต้องภายใต้การจับคู่ของพารามิเตอร์หลายแบบ

### Characteristics
| ประเภท | รายละเอียด |
|----------|-------------|
| Interface-based | argument เป็น string ที่รับมาจาก command line |
| Functionality-based | ตรวจสอบการ map ค่าที่รับมาไปยัง field ที่เกี่ยวข้องในคลาสได้ถูกต้อง |

### Input Domain Modeling (IDM)
| ขั้นตอน | รายละเอียด |
|----------|-------------|
| a | Identify testable function → `CmdLineParser.parseArgument(String... args)` |
| b | Identify parameters / returns / exceptions → args[], void, CmdLineException |
| c | Model input domain → (1) empty string, (2) normal string, (3) long string, (4) null |
| d | Combine partitions → PWC (จับคู่ค่าระหว่างพารามิเตอร์ต่าง ๆ) |
| e | Test Values → `["-s", "Hello"]`, `["-s", ""]`, `["-s", "LongText"]`, `["-s", null]` |
| f | Expected Values → valid → assign สำเร็จ ; invalid → throw exception |

### Test Summary
| Case | Input | Expected Output |
|------|--------|-----------------|
| Normal string | `-s Hello` | Assign สำเร็จ |
| Empty string | `-s ""` | Assign ได้ (ค่าค่าว่าง) |
| Long string | `-s "ThisIsLongText"` | Assign สำเร็จ |
| Null value | `-s null` | CmdLineException |

---

# 🧩 Test Suite 2 – EnumOptionHandlerTest  
**ISP Technique:** PWC (Pairwise Coverage)

### Objective
ทดสอบคลาส `EnumOptionHandler` เพื่อยืนยันว่าการ parse ค่าที่เป็น enum  
สามารถแปลงได้ถูกต้องตามค่าที่กำหนดไว้ และจัดการกับค่าที่ไม่อยู่ใน enum ได้อย่างถูกต้อง

### Characteristics
| ประเภท | รายละเอียด |
|----------|-------------|
| Interface-based | argument เป็น string ที่แทนชื่อของ enum |
| Functionality-based | ตรวจสอบการแปลงค่าจาก string → enum และจัดการค่าที่ไม่ตรงกับ enum |

### Input Domain Modeling (IDM)
| ขั้นตอน | รายละเอียด |
|----------|-------------|
| a | Identify testable function → `CmdLineParser.parseArgument(String... args)` |
| b | Identify parameters / returns / exceptions → args[], void, CmdLineException |
| c | Model input domain → (1) valid enum name, (2) invalid enum name, (3) case sensitivity |
| d | Combine partitions → PWC (จับคู่ค่าระหว่าง enum case และ string case) |
| e | Test Values → `"RED"`, `"BLUE"`, `"green"`, `"YELLOW"` |
| f | Expected Values → valid → assign สำเร็จ ; invalid → CmdLineException |

### Test Summary
| Case | Input | Expected Output |
|------|--------|-----------------|
| Valid enum (RED) | `-e RED` | Assign สำเร็จ |
| Valid enum (BLUE) | `-e BLUE` | Assign สำเร็จ |
| Invalid enum (lowercase) | `-e green` | CmdLineException |
| Non-existing enum | `-e YELLOW` | CmdLineException |

---

# 🧩 Test Suite 3 – FileOptionHandlerTest  
**ISP Technique:** ECC (Equivalence Class Coverage)

### Objective
ทดสอบคลาส `FileOptionHandler` เพื่อยืนยันว่าการแปลงค่า argument จาก command line  
สามารถสร้าง `File` object ได้ถูกต้องทั้งกรณี path ที่มีอยู่จริง ไม่มีอยู่จริง และมีช่องว่างในชื่อไฟล์

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
| d | Combine partitions → ECC (เลือก representative 1 ค่าในแต่ละ class) |
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

# 🧩 Test Suite 4 – URLOptionHandlerTest  
**ISP Technique:** ECC (Equivalence Class Coverage)

### Objective
ทดสอบคลาส `URLOptionHandler` เพื่อยืนยันว่าการ parse URL จาก command line  
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

## License Header
```java
/* Copyright (C) 2025
 * You may use, distribute and modify this code under the terms of the MIT license.
 */
