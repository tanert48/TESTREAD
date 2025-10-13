# Group 5 – args4j Handlers Testing  
ITDS362: Software Quality Assurance and Testing  

---

## 🧭 Overview  
รายงานนี้เป็นส่วนหนึ่งของรายวิชา **ITDS362 – Software Quality Assurance and Testing (SQAT)**  
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

## Test Suite 7 – StringArrayOptionHandlerTest  

| Characteristic | b1 | b2 | b3 |
|----------------|----|----|----|
| C1 = Array length | Empty | Single | Multiple |
| C2 = Contains null | Yes | No | - |

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-a ""` | Empty array | ⚠️ |
| T2 | `-a one` | Single | ✅ |
| T3 | `-a one two` | Multiple | ✅ |
| T4 | `-a null` | CmdLineException | ❌ |

---

## Test Suite 8 – InetAddressOptionHandlerTest  

| Characteristic | b1 | b2 | b3 |
|----------------|----|----|----|
| C1 = IP format | IPv4 | IPv6 | Invalid |
| C2 = Host reachable | Yes | No | - |

| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-ip 127.0.0.1` | Valid IPv4 | ✅ |
| T2 | `-ip ::1` | Valid IPv6 | ✅ |
| T3 | `-ip 999.999.999.999` | CmdLineException | ❌ |

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
