# ส่วนของนักศึกษา — Test Suite 3 & 4 (ตาม Requirement ข้อ 6–7)

---

## Test Suite 3 – FileOptionHandlerTest

### (a) Identify testable functions
- `CmdLineParser.parseArgument(String... args)` — แปลงอาร์กิวเมนต์ `-f <file>` เป็น `java.io.File` แล้วกำหนดลง field ของ bean

### (b) Identify parameters, return types, return values, and exceptional behavior
| Parameters       | Return Type | Return Value                      | Exceptional Behavior |
|------------------|-------------|-----------------------------------|----------------------|
| `String... args` | `void`      | ค่า `File` ถูก assign ให้ field   | ไม่มี (โค้ดทดสอบของชุดนี้ **ไม่คาดหวังการโยน** สำหรับ path ใด ๆ) |

> หมายเหตุ: ผิดพาธไฟล์ไม่โยน exception — โยนเฉพาะกรณีคำสั่ง CLI ผิด (เช่น -f แล้วไม่มีชื่อไฟล์). 

### (c) Model the input domain 

#### Interface-based characteristics 
| ID | Characteristic   | Partitions                 | ใช้ในเทสต์ |
|----|------------------|----------------------------|------------|
| C1 | Path existence   | { **exists**, **not exists** } | ใช้ทั้งสอง |
| C2 | File type        | { **file**, **directory** }   | ใช้ทั้งสอง |
| C3 | Path content     | { **normal**, **contains space** } | ใช้ทั้งสอง |

#### Functionality-based characteristic 
| ID | Characteristic                  | Partitions                                               | ใช้ในเทสต์ |
|----|---------------------------------|----------------------------------------------------------|------------|
| F1 | Mapping correctness to `File`   | { **assigned & matches canonical path**, **assigned & not exists** } | ใช้ทั้งคู่ |

> อธิบาย: โค้ดของชุดนี้ทดสอบ “ความถูกต้องของการ map ไปเป็น `File`” (เช็ค `not null`, `exists()`, `isFile()/isDirectory()`, และบางเคสเทียบ `getCanonicalPath()`)

### (d) การรวม partitions เพื่อสร้าง test requirements (Technique)
- ใช้ **ECC (Equivalence Class Coverage / Each-Choice)**: เลือกค่าอย่างน้อยหนึ่งค่าจากทุกบล็อกของแต่ละ characteristic ให้ถูกใช้ในอย่างน้อย  หนึ่ง test case

### (e) Test values & expected values (สอดคล้องกับ JUnit ในโค้ด)
| Test ID | Method name                                               | Partitions (ย่อ)                         | Expected |
|--------|---------------------------------------------------------------------|------------------------------------------|---------|
| F1     | `parseExistingFile_shouldAssignFile_andExistsTrue`                  | C1=exists, C2=file, C3=normal, F1=assigned&canonical | `exists()==true`, `isFile()==true`, canonical path ตรง |
| F2     | `parseDirectoryPath_shouldAssignFile_andIsDirectoryTrue`            | C1=exists, C2=directory, C3=normal       | `isDirectory()==true` |
| F3     | `parseNonExistingFile_shouldAssignFileObject_andExistsFalse`        | C1=not exists, C2=file, C3=normal, F1=assigned&not exists | `exists()==false` |
| F4     | `parsePathWithSpaces_shouldWork_andExistsTrue`                      | C1=exists, C2=file, C3=contains space    | `exists()==true` (รองรับช่องว่างใน path) |

### (f) ตรวจสอบความสอดคล้องกับ JUnit (Traceability)
| Test ID | Partitions Covered                                  |
|--------|------------------------------------------------------|
| F1     | C1=exists, C2=file, C3=normal, F1=assigned&canonical |
| F2     | C1=exists, C2=directory, C3=normal                   |
| F3     | C1=not exists, C2=file, C3=normal, F1=assigned&not exists |
| F4     | C1=exists, C2=file, C3=contains space                |

### (g) Explain test cases 
- **F1**: สร้างไฟล์จริง แล้ว parse ด้วย `-f <abs path>` → คาดหวัง `bean.file` ถูก assign, `exists()==true`, `isFile()==true`, และ canonical path ตรง  
- **F2**: ใช้พาธที่เป็น **directory** จริง → คาดหวัง `isDirectory()==true`  
- **F3**: ใช้พาธที่ “ไม่มีไฟล์อยู่จริง” → parse ได้, `exists()==false`  
- **F4**: สร้างไดเร็กทอรีชื่อมีช่องว่าง แล้วสร้างไฟล์ `"a b.txt"` ภายใน → parse ได้, `exists()==true` (ยืนยันรองรับ space)

---

## Test Suite 4 – URLOptionHandlerTest

### (a) Identify testable functions
- `CmdLineParser.parseArgument(String... args)` — แปลงอาร์กิวเมนต์ `-u <url>` เป็น `java.net.URL` แล้วกำหนดลง field ของ bean

### (b) Identify parameters, return types, return values, and exceptional behavior
| Parameters       | Return Type | Return Value                    | Exceptional Behavior |
|------------------|-------------|---------------------------------|----------------------|
| `String... args` | `void`      | ค่า `URL` ถูก assign ให้ field  | **โยน `CmdLineException`** เมื่อ URL format ผิด (missing protocol, bad protocol, protocol but no host) |

### (c) Model the input domain

#### Interface-based characteristics  
| ID | Characteristic | Partitions | ใช้ในเทสต์ |
|----|----------------|-------------|--------------|
| **C1** | Protocol | { **valid** (`http`, `https`), **invalid** (`htp`), **missing** } | ใช้ทั้งสาม |
| **C2** | Host | { **present**, **missing** } | ใช้ทั้งสอง |
| **C3** | Path / Query / Fragment | { **none**, **present** (`/a/b?q=1#frag`) } | ใช้ทั้งสอง |

#### Functionality-based characteristic
| ID | Characteristic                      | Partitions                                   | ใช้ในเทสต์ |
|----|-------------------------------------|----------------------------------------------|------------|
| F1 | URL object decomposition correctness | { **assigned & parts match**, **reject & throw** } | ใช้ทั้งคู่ |

> อธิบาย: มีเคสที่ต้อง parse แล้วตรวจ `getProtocol/Host/Path/Query/Ref` และเคสที่ต้อง **throw** เมื่อรูปแบบ URL ผิด

### (d) การรวม partitions เพื่อสร้าง test requirements (Technique)
- ใช้ **ECC**: ให้แต่ละบล็อกของทุก characteristic ถูกใช้อย่างน้อยหนึ่งครั้งในชุดเคส U1–U5

### (e) Test values & expected values (สอดคล้องกับ JUnit ในโค้ด)
| Test ID | Method name (ตามโค้ด)                                       | Partitions (ย่อ)                    | Expected |
|--------|---------------------------------------------------------------|-------------------------------------|---------|
| U1     | `parseValidHttps_shouldAssignURLObject`                       | C1=valid(`https`), C2=present, C3=none, F1=assigned&parts match | Parse OK (`getProtocol=https`, มี host) |
| U2     | `parseValidUrlWithPathQueryFragment_shouldKeepAllParts`       | C1=valid, C2=present, C3=present, F1=assigned&parts match | Parse OK (`getPath=/a/b`, `getQuery=q=1`, `getRef=frag`) |
| U3     | `missingProtocol_shouldThrowCmdLineException`                 | C1=missing                          | **throw `CmdLineException`** |
| U4     | `badProtocol_shouldThrowCmdLineException`                     | C1=invalid (`htp`)                  | **throw `CmdLineException`** |
| U5     | `protocolButNoHost_shouldThrowCmdLineException`               | C2=missing                          | **throw `CmdLineException`** |

### (f) ตรวจสอบความสอดคล้องกับ JUnit (Traceability)
| Test ID | Partitions Covered                                 |
|--------|-----------------------------------------------------|
| U1     | C1=valid, C2=present, C3=none, F1=assigned&parts match |
| U2     | C1=valid, C2=present, C3=present, F1=assigned&parts match |
| U3     | C1=missing (reject & throw)                         |
| U4     | C1=invalid (reject & throw)                         |
| U5     | C2=missing (reject & throw)                         |

### (g) Explain test cases (จากโค้ดของคุณ)
- **U1**: URL พื้นฐาน (`https://...`) → ต้อง assign เป็น `URL` สำเร็จ และ `getProtocol/Host` ถูกต้อง  
- **U2**: URL ที่มี path/query/fragment → ต้องคงค่า `getPath=/a/b`, `getQuery=q=1`, `getRef=frag`  
- **U3**: ไม่มี protocol (เช่น `"example.com"`) → parser ต้อง **โยน** `CmdLineException`  
- **U4**: protocol ผิดสะกด (`"htp://..."`) → **โยน** `CmdLineException`  
- **U5**: มี `://` แต่ไม่มี host (`"://invalid"` หรือ `"https://"` เปล่า) → **โยน** `CmdLineException`
