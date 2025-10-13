# 🧪 ITDS362 – Project 1: Input Space Partitioning on args4j

## 🧭 Overview  
รายงานนี้เป็นส่วนหนึ่งของรายวิชา **ITDS362 – Software Quality Assurance and Testing (SQAT)**  
มีวัตถุประสงค์เพื่อออกแบบและพัฒนา **Unit Test** สำหรับโปรเจกต์โอเพนซอร์ส [args4j](https://github.com/kohsuke/args4j)  
โดยประยุกต์ใช้เทคนิค **Input Space Partitioning (ISP)** ตามแนวทางในสไลด์ Module 5 – 6 (Week 3–4)

---

## ⚙️ Techniques Used
| Technique | Description | Handler 1 | Handler 2 |
|------------|--------------|------------|------------|
| **PWC** | Pairwise Coverage | `StringOptionHandler` | `EnumOptionHandler` |
| **ECC** | Equivalence Class Coverage | `FileOptionHandler` | `URLOptionHandler` |
| **BCC** | Base Choice Coverage | `BooleanOptionHandler` | `MapOptionHandler` |
| **MBCC** | Multiple Base Choice Coverage | `StringArrayOptionHandler` | `InetAddressOptionHandler` |
| **ACoC** | All Combinations Coverage | `SubCommandHandler` | `CmdLineParser` |

---

# 🧱 PWC – Pairwise Coverage

## Test Suite 1 – StringOptionHandlerTest  
*(ข้อมูลจากเพื่อน: ใช้ Pairwise Coverage เพื่อทดสอบค่าที่เป็น String input)*  

### a. Identify Testable Function  
`CmdLineParser.parseArgument(String... args)`

### b. Identify Parameters, Return, Exceptions  
| Parameters | Return | Exception |
|-------------|---------|------------|
| String[] args | void | CmdLineException |

### c. Model Input Domain  
**Interface-based:** ความยาวของสตริง, ความว่างเปล่า, ตัวอักษรพิเศษ  
**Functionality-based:** ตรวจสอบการ map ค่า string ไปยัง field ภายใน bean

### d. Combine Partitions (PWC)  
ใช้ Pairwise Coverage เพื่อให้ทุกคู่ของค่า characteristics ถูกทดสอบอย่างน้อย 1 ครั้ง  

### e. Derived Test Values  
| Input | Expected Result |
|--------|-----------------|
| `-s Hello` | Assign สำเร็จ |
| `-s ""` | ค่าว่าง |
| `-s LongText` | ยอมรับค่า |
| `-s null` | CmdLineException |

---

## Test Suite 2 – EnumOptionHandlerTest  
*(ข้อมูลจากเพื่อน: ใช้ Pairwise Coverage เพื่อทดสอบค่าของ Enum)*  

### a–f. Input Domain Modeling  
| Characteristic | Classes |
|----------------|----------|
| Enum value validity | { Valid, Invalid } |
| Case sensitivity | { Upper, Lower } |

### Derived Test Values  
| Input | Expected Result |
|--------|-----------------|
| `-e RED` | Assign สำเร็จ |
| `-e BLUE` | Assign สำเร็จ |
| `-e green` | Exception |
| `-e YELLOW` | Exception |

---

# 🧱 ECC – Equivalence Class Coverage

## Test Suite 3 – FileOptionHandlerTest  

### a. Identify Testable Function  
`CmdLineParser.parseArgument(String... args)`

### b. Identify Parameters, Return, Exceptions  
| Item | Detail |
|------|---------|
| Parameters | String[] args |
| Return | void |
| Exception | CmdLineException |

---

### c. Model the Input Domain

#### Interface-based Characteristics
| ID | Characteristic | Classes | Complete | Disjoint |
|----|----------------|----------|-----------|-----------|
| C1 | Path exists | { True, False } | ✅ | ✅ |
| C2 | Path type | { File, Directory } | ✅ | ✅ |
| C3 | Path contains space | { True, False } | ✅ | ✅ |

#### Functionality-based Characteristics
| ID | Characteristic | Classes | Complete | Disjoint |
|----|----------------|----------|-----------|-----------|
| F1 | Parse result | { Success, Error } | ✅ | ✅ |
| F2 | Resource validity | { Valid path, Invalid path } | ✅ | ✅ |

---

### d. Combine Partitions → Test Requirements
| TR | C1 | C2 | C3 | Expected Behavior |
|----|----|----|----|-------------------|
| TR1 | True | File | False | File exists |
| TR2 | True | Directory | False | Directory accepted |
| TR3 | False | File | False | File not found |
| TR4 | True | File | True | Handle space correctly |

---

### e. Derive Test Values  
| Test | Input | Expected Result |
|------|--------|-----------------|
| T1 | `-f ./target/test.txt` | exists()==true |
| T2 | `-f ./target/` | Directory recognized |
| T3 | `-f ./no_file.txt` | exists()==false |
| T4 | `-f "./My Folder/file.txt"` | Path handled correctly |

---

### f. Verify with JUnit  
| Method | TR | Behavior |
|--------|----|----------|
| parseExistingFile_shouldAssignFile_andExistsTrue | TR1 | File exists |
| parseDirectoryPath_shouldWork | TR2 | Directory |
| parseNonExistingFile_shouldNotExist | TR3 | Not found |
| parsePathWithSpaces_shouldWork | TR4 | Space in path |

---

## Test Suite 4 – URLOptionHandlerTest  

### a. Identify Testable Function  
`CmdLineParser.parseArgument(String... args)`

### b. Identify Parameters, Return, Exceptions  
| Parameters | Return | Exception |
|-------------|---------|------------|
| String[] args | void | CmdLineException |

---

### c. Model the Input Domain  

#### Interface-based Characteristics
| ID | Characteristic | Classes |
|----|----------------|----------|
| C1 | Protocol validity | { Valid, Invalid } |
| C2 | Host present | { Yes, No } |
| C3 | Structure | { Simple, With Query+Fragment } |

#### Functionality-based Characteristics
| ID | Characteristic | Classes |
|----|----------------|----------|
| F1 | Parsing result | { Success, CmdLineException } |
| F2 | Host interpretation | { Non-null, Null } |

---

### d. Combine Partitions → Test Requirements
| TR | C1 | C2 | Expected Behavior |
|----|----|-----------------------|
| TR1 | Valid | Host present | URL valid |
| TR2 | Valid | Host missing | host==null |
| TR3 | Invalid | Host present | CmdLineException |
| TR4 | Invalid | Host missing | CmdLineException |

---

### e. Derive Test Values
| Test | Input | Expected Result |
|------|--------|-----------------|
| T1 | `-u https://example.com` | Valid |
| T2 | `-u https://example.com/api?q=1#frag` | URL parts kept |
| T3 | `-u http://` | host == null |
| T4 | `-u htp://wrong.com` | CmdLineException |
| T5 | `-u example.com` | CmdLineException |

---

# 🧱 BCC – Base Choice Coverage

## Test Suite 5 – BooleanOptionHandlerTest  
🕓 *ยังไม่จัดทำ (ตามไฟล์: BooleanOptionHandlerTest.java)*

## Test Suite 6 – MapOptionHandlerTest  
🕓 *ยังไม่จัดทำ (ตามไฟล์: MapOptionHandlerTest.java)*

---

# 🧱 MBCC – Multiple Base Choice Coverage

## Test Suite 7 – StringArrayOptionHandlerTest  
*(ข้อมูลจากไฟล์ `UnexpectedCaseStringArrayOptionHandlerTest.java`)*  

### Summary  
ทดสอบความถูกต้องในการรับค่า arguments หลายค่าในรูปแบบ array โดยใช้ **MBCC**  
เลือก base choice แล้วเปลี่ยนค่าทีละ characteristic เช่น ขนาดของ array, การมีค่า null, และรูปแบบการคั่นข้อมูล

---

## Test Suite 8 – InetAddressOptionHandlerTest  
*(ข้อมูลจากไฟล์ `UnexpectedCaseInetAddressOptionHandlerTest.java`)*  

### Summary  
ทดสอบความถูกต้องของการแปลงค่า IP address จาก argument โดยใช้ **MBCC**  
เช่น IP ถูกต้อง / ผิดรูปแบบ / IPv4 / IPv6 เพื่อยืนยันการโยน exception ถูกต้องตามสเปก

---

# 🧱 ACoC – All Combinations Coverage

## Test Suite 9 – SubCommandHandlerTest  
🕓 *ยังไม่จัดทำ (ตามไฟล์: SubCommandHandlerTest.java)*

## Test Suite 10 – CmdLineParserTest  
🕓 *ยังไม่จัดทำ (ตามไฟล์: CmdLineParserTest.java)*

---

## 📚 References  
- Module 5: Input Space Partitioning  
- Module 6: Input Space Partitioning (Part 2)  
- args4j Official Repository: [https://github.com/kohsuke/args4j](https://github.com/kohsuke/args4j)
