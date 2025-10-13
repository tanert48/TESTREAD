# Group 5 – args4j Handlers Testing  
ITDS362: Software Quality Assurance and Testing  
Project 1 – Input Space Partitioning (ISP)  

---

## 🧭 Overview  
รายงานนี้เป็นส่วนหนึ่งของรายวิชา **ITDS362 – Software Quality Assurance and Testing (SQAT)**  
มีวัตถุประสงค์เพื่อออกแบบและพัฒนา **Unit Test** สำหรับไลบรารี [args4j](https://github.com/kohsuke/args4j)  
โดยประยุกต์ใช้เทคนิค **Input Space Partitioning (ISP)** ตามสไลด์ Module 5–6  
และออกแบบ Test Suite ครอบคลุมทุกเทคนิค ISP ได้แก่  
**PWC, ECC, BCC, MBCC และ ACoC**

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

**2. Identify parameters, return types, return values, and exceptional behavior**  
| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|-------------|---------------|----------------------|
| `String[] args` | `void` | Assigns String value | `CmdLineException` (ถ้าพบ input ไม่ถูกต้อง) |

**3. Model the input domain**  
| Characteristic | b1 | b2 | b3 | b4 |
|----------------|----|----|----|----|
| C1 = Input type | Normal string | Empty | Very long | Null |

---

### 🧩 Task II: Choose combinations of values  

**4. Combine partitions into tests (PWC)**  
| Test Requirement | Combination | Description |
|------------------|--------------|-------------|
| (C1b1,C2b1) | Normal, Valid | Input ทั่วไป |
| (C1b2,C2b2) | Empty, Missing | Empty value |
| (C1b3,C2b1) | Long, Valid | Long input |
| (C1b4,C2b2) | Null, Invalid | Null input |

**5. Derive test values**  
| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-s Hello` | Success | ✅ |
| T2 | `-s ""` | Empty String | ⚠️ |
| T3 | `-s LongText` | Success | ✅ |
| T4 | `-s null` | CmdLineException | ❌ |

---

### f. Verify with JUnit  
| Method | Test ID | Behavior |
|--------|----------|----------|
| `parseValidString_shouldAssignCorrectly()` | T1 | Valid string |
| `parseEmptyString_shouldBeAccepted()` | T2 | Empty |
| `parseNullString_shouldThrowException()` | T4 | Invalid |

### g. Combine Interface-based & Functionality-based Characteristics  
| View | Focus | Example |
|------|--------|----------|
| Interface-based | รูปแบบข้อความ input เช่น ความยาว | `"LongText"` |
| Functionality-based | พฤติกรรมของระบบเมื่อ input ผิดรูปแบบ | `"null"` → Exception |

---

## Test Suite 2 – EnumOptionHandlerTest  

### 🧩 Task I: Model Input Domain  

**1. Identify testable functions**  
`CmdLineParser.parseArgument(String... args)`  

**2. Identify parameters, return types, return values, and exceptional behavior**  
| Parameters | Return Type | Return Value | Exceptional Behavior |
|-------------|-------------|---------------|----------------------|
| `String[] args` | `void` | Enum value assigned | `CmdLineException` (invalid enum) |

**3. Model the input domain**  
| Characteristic | b1 | b2 | b3 | b4 |
|----------------|----|----|----|----|
| C1 = Enum value | Valid uppercase | Valid lowercase | Invalid | Missing |

---

### 🧩 Task II: Choose combinations of values  

**4. Combine partitions into tests (PWC)**  
| Test Requirement | Combination | Description |
|------------------|--------------|-------------|
| (C1b1) | Valid uppercase | Correct enum |
| (C1b2) | Valid lowercase | Case sensitivity |
| (C1b3) | Invalid | Not exist |
| (C1b4) | Missing | Missing arg |

**5. Derive test values**  
| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-e RED` | Success | ✅ |
| T2 | `-e blue` | CmdLineException | ❌ |
| T3 | `-e YELLOW` | CmdLineException | ❌ |

---

# 🧱 ECC – Equivalence Class Coverage  

## Test Suite 3 – FileOptionHandlerTest  

(ตามที่พี่เขียนแบบสไลด์ triangle() — Task I / II ครบทุกข้อ a–g พร้อมตาราง b1–b4 และ T1–T4)

[...เนื้อหา ECC – FileOptionHandlerTest และ URLOptionHandlerTest แบบที่เราทำไว้ก่อนหน้า รวมอยู่ตรงนี้...]

---

# 🧱 BCC – Base Choice Coverage  

*(โครงเว้นไว้สำหรับเพื่อนเติม)*  
## Test Suite 5 – BooleanOptionHandlerTest  
🕓 ยังไม่จัดทำ (ตามไฟล์: BooleanOptionHandlerTest.java)

## Test Suite 6 – MapOptionHandlerTest  
🕓 ยังไม่จัดทำ (ตามไฟล์: MapOptionHandlerTest.java)

---

# 🧱 MBCC – Multiple Base Choice Coverage  

## Test Suite 7 – StringArrayOptionHandlerTest  

### 🧩 Task I: Model Input Domain  
`CmdLineParser.parseArgument(String... args)`  
→ รับค่าหลายค่าผ่าน argument array  

| Characteristic | b1 | b2 | b3 |
|----------------|----|----|----|
| C1 = Array length | Empty | Single | Multiple |
| C2 = Contains null | Yes | No | - |

### 🧩 Task II: Choose combinations of values  
| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-a ""` | Empty array | ⚠️ |
| T2 | `-a one` | Single | ✅ |
| T3 | `-a one two` | Multiple | ✅ |
| T4 | `-a null` | CmdLineException | ❌ |

---

## Test Suite 8 – InetAddressOptionHandlerTest  

### 🧩 Task I: Model Input Domain  
| Characteristic | b1 | b2 | b3 |
|----------------|----|----|----|
| C1 = IP format | IPv4 | IPv6 | Invalid |
| C2 = Host reachable | Yes | No | - |

### 🧩 Task II: Choose combinations of values  
| Test ID | Input | Expected Result | Outcome |
|----------|--------|-----------------|----------|
| T1 | `-ip 127.0.0.1` | Valid IPv4 | ✅ |
| T2 | `-ip ::1` | Valid IPv6 | ✅ |
| T3 | `-ip 999.999.999.999` | CmdLineException | ❌ |

---

# 🧱 ACoC – All Combinations Coverage  

*(โครงเว้นไว้สำหรับเพื่อนเติม)*  
## Test Suite 9 – SubCommandHandlerTest  
🕓 ยังไม่จัดทำ (ตามไฟล์: SubCommandHandlerTest.java)

## Test Suite 10 – CmdLineParserTest  
🕓 ยังไม่จัดทำ (ตามไฟล์: CmdLineParserTest.java)

---

# 📊 Summary  

| Technique | Description | Coverage | Status |
|------------|-------------|-----------|----------|
| PWC | Pairwise combinations | String, Enum | ✅ Done |
| ECC | Equivalence classes | File, URL | ✅ Done |
| BCC | Base Choice | Boolean, Map | 🕓 Pending |
| MBCC | Multiple Base Choice | StringArray, InetAddress | ✅ Done |
| ACoC | All Combinations | SubCommand, CmdLineParser | 🕓 Pending |

---

# 📚 References  

- Week 3: Module 5 – Input Space Partitioning  
- Week 4: Module 6 – Input Space Partitioning (Part 2)  
- args4j Official Repository: [https://github.com/kohsuke/args4j](https://github.com/kohsuke/args4j)
