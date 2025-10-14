# 🧱 Input Space Partitioning (ISP) — Test Suites (Student Portion)

เอกสารนี้คือ README ของ **ส่วนงานนักศึกษา** (2 test suites) ที่ออกแบบและพัฒนาโดยใช้เกณฑ์ **ECC (Each-Choice Coverage)** ครอบคลุมทั้ง Interface-based Characteristics, ตาราง IDM, ตาราง Combination, Traceability, วิธีรัน และคำชี้แจงตามข้อกำหนด 7(a)–(g)

> อ้างอิงแนวคิดจาก Offutt & Ammann และสไลด์ ITDS362 Module 5–6

---

## 🎯 Goals & Scope

- เป้าหมาย: ออกแบบและเขียน **ชุดทดสอบ 2 ชุด** (ไม่ซ้ำของเดิม) สำหรับ args4j option handlers
- เทคนิค ISP ที่ใช้: **ECC (Each-Choice Coverage)** ทั้งสองชุด
- ระดับการทดสอบ: **Unit test** (JUnit) บน Maven
- ขอบเขต: เฉพาะส่วนของนักศึกษาคนนี้ (ไม่รวมชุดของเพื่อนในกลุ่ม)

---

## 🧪 Test Suite A — `URLOptionHandlerTest.java`

**Technique:** ECC (Each-Choice)  
**Approach:** Interface-based  
**Option:** `-u <url>`

### 1) Testable Function
- `CmdLineParser.parseArgument(String... args)` ทำหน้าที่ map สตริงพารามิเตอร์ไปเป็น `java.net.URL` ที่ field ของ bean

### 2) Parameters / Return / Exceptional behavior
| Parameters     | Return Type | Return Value                    | Exceptional Behavior                          |
|----------------|-------------|---------------------------------|-----------------------------------------------|
| `String[] args`| `void`      | `URL` assigned to option field  | `CmdLineException` เมื่อฟอร์แมต URL ผิด (protocol/host/missing) |

### 3) Input Domain Modelling (IDM)
**Characteristics & Partitions**
- **C1 Protocol:** { **valid** (`http|https`), **invalid**, **missing** }
- **C2 Host:** { **present**, **missing** }
- **C3 Path/Query/Fragment:** { **none**, **present** (เช่น `/a/b?q=1#frag`) }

> หมายเหตุ: เลือกให้ **ครบทุกบล็อก** อย่างน้อย 1 ครั้ง (ตาม ECC) โดยไม่ต้องครอบทุก combination ทั้งหมดแบบ ACoC

### 4) ECC Combinations (Test Requirements → Test Cases)
| Test ID | C1 (Protocol)      | C2 (Host) | C3 (Path/Query/Frag) | Expected Result |
|--------|---------------------|-----------|----------------------|-----------------|
| **U1** | valid (`https`)     | present   | none                 | Parsed OK (`getProtocol=https`, `getHost` ถูกต้อง) |
| **U2** | valid (`https`)     | present   | present              | Parsed OK (`getPath=/a/b`, `getQuery=q=1`, `getRef=frag`) |
| **U3** | invalid (`htp`)     | present   | none                 | **CmdLineException** |
| **U4** | missing (`example.com`) | present(implicit) | none          | **CmdLineException** |
| **U5** | valid               | missing (`https://`) | none          | **CmdLineException** |

### 5) Derive Concrete Values
- U1: `-u https://ex.com`
- U2: `-u https://ex.com/a/b?q=1#frag`
- U3: `-u htp://ex.com`
- U4: `-u example.com`
- U5: `-u https://`

---

## 🧪 Test Suite B — `FileOptionHandlerTest.java`

**Technique:** ECC (Each-Choice)  
**Approach:** Interface-based  
**Option:** `-f <file>`

### 1) Testable Function
- `CmdLineParser.parseArgument(String... args)` ทำหน้าที่ map สตริงพารามิเตอร์ไปเป็น `java.io.File` ที่ field ของ bean

### 2) Parameters / Return / Exceptional behavior
| Parameters     | Return Type | Return Value                    | Exceptional Behavior |
|----------------|-------------|---------------------------------|----------------------|
| `String[] args`| `void`      | `File` assigned to option field | **ไม่มี**กรณี “invalid OS path string” (handler จะไม่โยนเพราะไม่ตรวจความถูกต้องของ path ระดับ OS); จะโยน `CmdLineException` เฉพาะ syntax ผิด เช่น ไม่มีอาร์กิวเมนต์ตามหลัง `-f` |

### 3) Input Domain Modelling (IDM) — *สอดคล้องพฤติกรรมจริงของ args4j*
**Characteristics & Partitions**
- **C1 Path existence:** { **exists**, **not exists** }
- **C2 File type:** { **regular file**, **directory** }
- **C3 Path content:** { **normal**, **contains space** }
- **C4 OS validity:** { **normal string**, **invalid/unsupported string** } → *ยังถูกห่อเป็น `File` ได้โดยไม่โยน exception*

### 4) ECC Combinations (Test Requirements → Test Cases)
| Test ID | Combination                                | Description           | Expected Result |
|--------|---------------------------------------------|-----------------------|-----------------|
| **F1** | C1=exists + C2=file + C3=normal             | Existing file         | Parsed OK, `exists()==true`, `isFile()==true` |
| **F2** | C1=not exists + C2=file + C3=normal         | Non-existing file     | Parsed OK, `exists()==false` |
| **F3** | C1=exists + C2=directory + C3=normal        | Existing directory    | Parsed OK, `isDirectory()==true` |
| **F4** | C1=exists + C2=file + C3=contains space     | Path contains space   | Parsed OK |
| **F5** | C1=not exists + C2=file + C4=invalid string | OS-invalid string     | Parsed OK, **no exception**, `exists()==false` |

### 5) Derive Concrete Values (ตัวอย่าง)
- F1: temp ไฟล์จริง (เช่น `Files.createTempFile(...)`)
- F2: สตริงไฟล์ที่มั่นใจว่าไม่มีอยู่ (เช่น `${tmp}/no_such_file_12345.txt`)
- F3: temp โฟลเดอร์ (เช่น `Files.createTempDirectory(...)`)
- F4: path มีช่องว่าง (เช่น `"${tmp}/a b.txt"`)
- F5: สตริงที่ *อาจ* invalid บาง OS (ยัง parse ได้ ไม่โยน): เช่น `"::invalid::<name>"` (ผลลัพธ์คาดหวัง `exists()==false`)

> เคสโยน `CmdLineException` ที่เหมาะสมกับชุดนี้คือ **syntax ผิด** (เช่น `-f` ไม่มีอาร์กิวเมนต์ต่อท้าย) ซึ่งเป็น “CLI format error” ไม่ใช่ “OS path validation” — ถ้าจะเพิ่มให้ครอบคลุม behavior ของ parser ก็สามารถทำเป็นเคสแยกได้

---

## 🧾 Traceability (Mapping Test ↔ Partitions/Requirements)

| Test ID | Method (ตัวอย่างชื่อ)                           | Requirement / Intent          | Partitions Covered                    |
|--------|---------------------------------------------------|-------------------------------|---------------------------------------|
| U1     | `url_valid_minimal_should_parse`                  | URL valid (base)              | C1=valid, C2=present, C3=none         |
| U2     | `url_with_path_query_fragment_should_parse`       | URL valid (extended parts)    | C1=valid, C3=present                  |
| U3     | `url_invalid_protocol_should_throw`               | URL invalid protocol          | C1=invalid                            |
| U4     | `url_missing_protocol_should_throw`               | URL missing protocol          | C1=missing                            |
| U5     | `url_missing_host_should_throw`                   | URL missing host              | C2=missing                            |
| F1     | `file_existing_should_parse`                      | File exists                   | C1=exists, C2=file                    |
| F2     | `file_non_existing_should_parse_as_nonexist`      | File not exists               | C1=not exists                         |
| F3     | `directory_should_be_parsed_as_directory`         | Directory exists              | C2=directory                          |
| F4     | `file_path_with_space_should_parse`               | Path contains space           | C3=contains space                     |
| F5     | `file_invalid_string_parsed_no_exception`         | OS-invalid string no-throw    | C4=invalid string (parse as File)     |

> จุดประสงค์ของตารางนี้คือยืนยันว่า **ทุกบล็อกใน IDM ถูกแตะอย่างน้อยหนึ่งครั้ง** ตามนิยาม **ECC**

---

## (7d) Roadmap — ISP Techniques Coverage (ทีม)

> ข้อกำหนดระบุว่าทั้งทีมต้องใช้ให้ครบ **5 เทคนิค ISP** (ACoC, ECC, PWC, TWC, BCC/MBCC) ใน **10 test suites** รวมกัน

- ส่วนของนักศึกษาคนนี้: **ECC** จำนวน **2/10**  
  - `URLOptionHandlerTest.java` → **ECC**  
  - `FileOptionHandlerTest.java` → **ECC**  
- ส่วนอื่น ๆ (ACoC, PWC, TWC, BCC/MBCC): ให้ทีมเติมใน README รวมของทีม

---

## ⚙️ How to Run

```bash
# รันเฉพาะสองไฟล์เทสต์ของนักศึกษา
mvn -q -Dtest=URLOptionHandlerTest,FileOptionHandlerTest test

# หรือรันเทสต์ทั้งหมดในโมดูล/โปรเจกต์
mvn -q test
