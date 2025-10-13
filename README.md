# Group 5 – args4j Handlers Testing  
ITDS362: Software Quality Assurance and Testing  

---

## 🧭 Overview  
รายงานนี้เป็นส่วนหนึ่งของรายวิชา **ITDS362 – Software Quality Assurance and Testing**
มีวัตถุประสงค์เพื่อออกแบบและพัฒนา **Unit Test** สำหรับไลบรารี [args4j](https://github.com/kohsuke/args4j)  
โดยใช้เทคนิค **Input Space Partitioning (ISP)** ตาม Module 5–6  
เพื่อแสดงการวิเคราะห์ input domain และออกแบบชุดทดสอบอย่างเป็นระบบ  
ครอบคลุมเทคนิคทั้ง 5 ได้แก่ **PWC, ECC, BCC, MBCC และ ACoC**

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

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`CmdLineParser.parseArgument(String... args)`  
→ ทำหน้าที่รับค่า String argument แล้วแปลงเป็นตัวแปรใน Bean  

**2. Identify parameters, return types, return values, and exceptional behavior**  

| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|--------------|---------------|----------------------|
| `String[] args` | `void` | Assigns string value to field | `CmdLineException` (หาก input ผิดรูปแบบ) |

**3. Model the input domain**  

| Characteristic | b1 | b2 | b3 | b4 |
|----------------|----|----|----|----|
| C1 = Input value | Normal string | Empty | Very long | Null |

---

### 🧩 Task II: Choose combinations of values  

**4. Combine partitions into tests (PWC)**  
ใช้ Pairwise Coverage เพื่อให้ทุกคู่ของค่า characteristics ปรากฏอย่างน้อยหนึ่งครั้ง  

| Test Requirement | Combination | Description |
|------------------|--------------|--------------|
| (C1b1,C2b1) | Normal, Valid | ป้อนค่าปกติ |
| (C1b2,C2b2) | Empty, Missing | ป้อนค่าว่าง |
| (C1b3,C2b1) | Long, Valid | ป้อนข้อความยาวมาก |
| (C1b4,C2b2) | Null, Invalid | ป้อนไม่มีค่า |

**5. Derive test values**  

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-s Hello` | Success | ✅ |
| T2 | `-s ""` | Empty string | ⚠️ |
| T3 | `-s LongText` | Success | ✅ |
| T4 | `-s null` | CmdLineException | ❌ |

---

### f. Verify with JUnit  

| Method | Test ID | Behavior |
|--------|----------|----------|
| `parseValidString_shouldAssignCorrectly()` | T1 | Assigns correctly |
| `parseEmptyString_shouldBeAccepted()` | T2 | Accepts empty string |
| `parseNullString_shouldThrowException()` | T4 | Throws exception |

### g. Combine Interface-based & Functionality-based Characteristics  

| View | Focus | Example |
|------|--------|----------|
| Interface-based | รูปแบบ input เช่น ความยาวหรือว่างเปล่า | `"LongText"`, `""` |
| Functionality-based | พฤติกรรมระบบเมื่อ input ผิด | `"null"` → Exception |

---

## Test Suite 2 – EnumOptionHandlerTest  

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`CmdLineParser.parseArgument(String... args)`  

**2. Identify parameters, return types, return values, and exceptional behavior**  

| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|--------------|---------------|----------------------|
| `String[] args` | `void` | Enum value assigned | `CmdLineException` (invalid enum) |

**3. Model the input domain**  

| Characteristic | b1 | b2 | b3 | b4 |
|----------------|----|----|----|----|
| C1 = Enum value | Valid uppercase | Valid lowercase | Invalid | Missing |

---

### 🧩 Task II: Choose combinations of values  

**4. Combine partitions into tests (PWC)**  

| Test Requirement | Combination | Description |
|------------------|--------------|--------------|
| (C1b1) | Valid uppercase | Enum valid |
| (C1b2) | Valid lowercase | Case sensitivity |
| (C1b3) | Invalid | Value not exist |
| (C1b4) | Missing | Missing arg |

**5. Derive test values**  

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-e RED` | Success | ✅ |
| T2 | `-e blue` | Exception | ❌ |
| T3 | `-e YELLOW` | Exception | ❌ |

---

# 🧱 ECC – Equivalence Class Coverage  

## Test Suite 3 – FileOptionHandlerTest  

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`CmdLineParser.parseArgument(String... args)`  

**2. Identify parameters, return types, return values, and exceptional behavior**  

| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|--------------|---------------|----------------------|
| `String[] args` | `void` | File object assigned to field | `CmdLineException` (invalid path) |

**3. Model the input domain (Functionality-based)**  

| Characteristic | b1 | b2 | b3 | b4 |
|----------------|----|----|----|----|
| C1 = File path condition | Path exists | Directory | Path not exist | Invalid format |

---

### 🧩 Task II: Choose combinations of values  

**4. Combine partitions into tests (ECC)**  

| Test Requirement | Combination | Description |
|------------------|--------------|--------------|
| (C1b1) | Path exists | File ปกติ |
| (C1b2) | Directory | Folder |
| (C1b3) | Not exist | Missing file |
| (C1b4) | Invalid | Invalid path |

**5. Derive test values**  

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-f ./target/test.txt` | File.exists()==true | ✅ |
| T2 | `-f ./target/` | Directory accepted | ✅ |
| T3 | `-f ./no_file.txt` | File.exists()==false | ⚠️ |
| T4 | `-f ?:badpath` | CmdLineException | ❌ |

---

### f. Verify with JUnit  

| Method | Test ID | Behavior |
|--------|----------|----------|
| `parseExistingFile_shouldAssignFile_andExistsTrue()` | T1 | File exists |
| `parseDirectoryPath_shouldWork()` | T2 | Directory |
| `parseNonExistingFile_shouldNotExist()` | T3 | Not exist |
| `parseInvalidPath_shouldThrowException()` | T4 | Invalid |

### g. Combine Interface-based & Functionality-based Characteristics  

| View | Focus | Example |
|------|--------|----------|
| Interface-based | รูปแบบ path เช่น relative, มีช่องว่าง | `"./My Folder/test.txt"` |
| Functionality-based | พฤติกรรมระบบเมื่อ path ไม่ถูก | `"./no_file.txt"` → File.exists()==false |

---

## Test Suite 4 – URLOptionHandlerTest  

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`CmdLineParser.parseArgument(String... args)`  

**2. Identify parameters, return types, return values, and exceptional behavior**  

| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|--------------|---------------|----------------------|
| `String[] args` | `void` | URL object assigned to field | `CmdLineException` (invalid URL) |

**3. Model the input domain**  

| Characteristic | b1 | b2 | b3 | b4 | b5 |
|----------------|----|----|----|----|----|
| C1 = URL validity | Valid (https) | Valid with query | Missing host | Invalid protocol | Missing protocol |

---

### 🧩 Task II: Choose combinations of values  

**4. Combine partitions into tests (ECC)**  

| Test Requirement | Combination | Description |
|------------------|--------------|--------------|
| (C1b1) | Valid https | URL ปกติ |
| (C1b2) | Valid query | มี query fragment |
| (C1b3) | Missing host | Host=null |
| (C1b4) | Invalid protocol | protocol ผิด |
| (C1b5) | Missing protocol | ไม่มี protocol |

**5. Derive test values**  

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-u https://example.com` | Valid | ✅ |
| T2 | `-u https://example.com/api?q=1#frag` | Valid query | ✅ |
| T3 | `-u http://` | host==null | ⚠️ |
| T4 | `-u htp://wrong.com` | CmdLineException | ❌ |
| T5 | `-u example.com` | CmdLineException | ❌ |

---

### f. Verify with JUnit  

| Method | Test ID | Behavior |
|--------|----------|----------|
| `parseValidHttps_shouldAssignURLObject()` | T1 | Valid URL |
| `parseValidUrlWithPathQueryFragment_shouldKeepAllParts()` | T2 | URL parts |
| `protocolButNoHost_shouldThrowCmdLineException()` | T3 | Missing host |
| `badProtocol_shouldThrowCmdLineException()` | T4 | Invalid |
| `missingProtocol_shouldThrowCmdLineException()` | T5 | Missing protocol |

### g. Combine Interface-based & Functionality-based Characteristics  

| View | Focus | Example |
|------|--------|----------|
| Interface-based | รูปแบบ URL เช่น มี query fragment | `"https://example.com/api?q=1#frag"` |
| Functionality-based | พฤติกรรมเมื่อ URL ผิด เช่น protocol ผิด | `"htp://wrong.com"` → Exception |

---

# 🧱 BCC – Base Choice Coverage  
🕓 ยังไม่จัดทำ (เพื่อนสามารถเติมในรูปแบบเดียวกันได้)

---

# 🧱 MBCC – Multiple Base Choice Coverage  

---

## Test Suite 7 – UnexpectedCaseStringArrayOptionHandlerTest (StringArrayOptionHandler)

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`StringArrayOptionHandler.parseArguments(Parameters params)`  
→ อ่านอาร์กิวเมนต์หลายค่าแล้วเพิ่มลงในฟิลด์ `String[]` ของ bean

---

**2. Identify parameters, return types, return values, and exceptional behavior**  

| Parameters | Return Type | Return Value / Side Effect | Exceptional Behavior |
|-------------|--------------|-----------------------------|----------------------|
| `Parameters params` | `int` | จำนวนอาร์กิวเมนต์ที่ consume และเพิ่มลงใน `String[]` | `CmdLineException` (ถ้า parser หรือรูปแบบผิดพลาด) |

---

**3. Model the input domain (เลือก Base Choice + พัฒนาลักษณะ)**  

> Base Choice: อินพุตปกติ หลายค่า ไม่มีค่าว่าง

| ID | Characteristic | b1 (Base Choice) | b2 | b3 |
|----|----------------|------------------|----|----|
| C1 | เนื้อหาของอาร์กิวเมนต์ | สตริงปกติ (non-empty) | สตริงว่าง `""` | สตริงมีแต่ช่องว่าง `" "` |
| C2 | จำนวนอาร์กิวเมนต์ | หลายค่า (≥2) | ค่าเดียว | – |

---

### 🧩 Task II: Choose combinations of values (MBCC)

**4. Combine partitions into tests (จาก Base → เปลี่ยนทีละ characteristic)**  

| Test Requirement (TR) | Combination | คำอธิบาย |
|------------------|--------------|------------|
| Base | (C1=b1, C2=b1) | หลายค่า ปกติ |
| TR1 | (C1=b2, C2=b1) | แทรกค่าว่าง `""` กลางชุด |
| TR2 | (C1=b3, C2=b1) | แทรก `" "` ช่องว่างล้วนกลางชุด |
| TR3 | (C1=b1, C2=b2) | ลดเหลือค่าเดียว ปกติ |

---

**5. Derive test values**  

| Test ID | Input (ตัวอย่างพารามิเตอร์) | Expected Result | Outcome |
|----------|-------------------------------|-----------------|----------|
| **T-Base** | `{"a","b","c"}` | `["a","b","c"]` | ✅ Success |
| **T1** | `{"a","","b"}` | ยอมรับ `""` เป็น element | ⚠️ Special |
| **T2** | `{"a"," ","b"}` | `" "` ถูกละทิ้ง | ⚠️ Special |
| **T3** | `{"solo"}` | `["solo"]` | ✅ Success |

---

### f. Verify with JUnit  

| JUnit Method | Test ID | Behavior |
|---------------|----------|-----------|
| `testParseWithEmptyStringArgument()` | T1 | ยอมรับ `""` ได้ |
| `testParseWithSpaceAsArgument()` | T2 | `" "` ถูกละทิ้ง |
| *(เพิ่มเติม)* `testParseBaseMultiple()` | T-Base | หลายค่า ปกติ |
| *(เพิ่มเติม)* `testParseSingle()` | T3 | ค่าเดียว ปกติ |

---

### g. Combine Interface-based & Functionality-based Characteristics  

| มุมมอง | โฟกัส | ตัวอย่าง |
|----------|--------|----------|
| **Interface-based** | โครงสร้าง argument เช่น จำนวนสมาชิก / เป็น `""` หรือ `" "` | `{"a","","b"}`, `{"a"," ","b"}` |
| **Functionality-based** | พฤติกรรมหลัง parse (เก็บ, ละทิ้ง, นับจำนวน) | `""` → เก็บไว้, `" "` → ละทิ้ง |

---

## Test Suite 8 – UnexpectedCaseInetAddressOptionHandlerTest (InetAddressOptionHandler)

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`InetAddressOptionHandler.parse(String argument)`  
→ แปลงสตริงเป็น `InetAddress`

---

**2. Identify parameters, return types, return values, and exceptional behavior**  

| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|--------------|---------------|----------------------|
| `String argument` | `InetAddress` | คืนค่า InetAddress จาก argument | `CmdLineException` (ถ้า parse ไม่ได้) |

---

**3. Model the input domain (เลือก Base Choice + พัฒนาลักษณะ)**  

> Base Choice: IPv4 ปกติ `"1.2.3.4"`

| ID | Characteristic | b1 (Base Choice) | b2 |
|----|----------------|------------------|----|
| C1 | รูปแบบสตริงอินพุต | IPv4 ปกติ | สตริงว่าง `""` |
| C2 | ประเภทที่อยู่ | ปกติทั่วไป | **Loopback** (`127.0.0.1`) |

หมายเหตุ: ใน JDK, `InetAddress.getByName("")` → loopback

---

### 🧩 Task II: Choose combinations of values (MBCC)

**4. Combine partitions into tests (จาก Base → เปลี่ยนทีละ characteristic)**  

| Test Requirement (TR) | Combination | คำอธิบาย |
|------------------|--------------|------------|
| Base | (C1=b1, C2=b1) | IPv4 ปกติ |
| TR1 | (C1=b1, C2=b2) | Loopback (`127.0.0.1`) |
| TR2 | (C1=b2, C2=b1) | ใช้ค่าว่าง `""` (JDK map เป็น loopback) |

---

**5. Derive test values**  

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| **T-Base** | `"8.8.8.8"` | IPv4 ปกติ | ✅ Success |
| **T1** | `"127.0.0.1"` | Loopback address | ✅ Success |
| **T2** | `""` | Map เป็น loopback | ⚠️ Special |

---

### f. Verify with JUnit  

| JUnit Method | Test ID | Behavior |
|---------------|----------|-----------|
| `testParseLoopbackAddress()` | T1 | คืนค่า loopback |
| `testParseEmptyString()` | T2 | `""` → loopback |
| *(เพิ่มเติม)* `testParseBaseIPv4()` | T-Base | IPv4 ปกติ |
| *(เพิ่มเติม)* `testParseInvalidIp_shouldThrow()` | – | Invalid IP → Exception |

---

### g. Combine Interface-based & Functionality-based Characteristics  

| มุมมอง | โฟกัส | ตัวอย่าง |
|----------|--------|----------|
| **Interface-based** | รูปแบบสตริง เช่น IPv4, ว่าง | `"8.8.8.8"`, `""` |
| **Functionality-based** | พฤติกรรมระบบ เช่น map loopback หรือ throw | `""` → loopback, `"999.999.999.999"` → Exception |

---

### 🧾 Summary of MBCC  

| Handler | Base Choice | Special Cases Covered | ผลลัพธ์หลัก |
|----------|--------------|-----------------------|---------------|
| **StringArrayOptionHandler** | หลายค่า `"a","b","c"` | `""` (เก็บไว้), `" "` (ละทิ้ง), ค่าเดียว | การ parse array ยืดหยุ่นและ trim ถูกต้อง |
| **InetAddressOptionHandler** | IPv4 `"8.8.8.8"` | loopback (`127.0.0.1`), ค่าว่าง `""` | ครอบคลุม edge case ของ JDK (loopback mapping) |
---

# 🧱 ACoC – All Combinations Coverage  
🕓 ยังไม่จัดทำ (เพื่อนสามารถเติมภายหลังได้)

---

# 📊 Summary  

| Technique | Coverage | Handlers | Status |
|------------|-----------|-----------|----------|
| PWC | Pairwise combinations | String, Enum | ✅ Done |
| ECC | Equivalence classes | File, URL | ✅ Done |
| BCC | Base Choice | Boolean, Map | 🕓 Pending |
| MBCC | Multiple Base Choice | StringArray, InetAddress | ✅ Done |
| ACoC | All Combinations | SubCommand, CmdLineParser | 🕓 Pending |

---

# 📚 References  

- **Module 5:** Input Space Partitioning (Week 3)  
- **Module 6:** Input Space Partitioning 2 (Week 4)  
- args4j Official Repository: [https://github.com/kohsuke/args4j](https://github.com/kohsuke/args4j)
