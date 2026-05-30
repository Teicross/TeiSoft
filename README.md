# AI Micro-Tasking Transaction Pipeline Project

สถาปัตยกรรม Multi-Agent สำหรับการบริหารจัดการคิวงานและระบบ CI/CD บน Codex Platform ที่ออกแบบมาเพื่อควบคุมความปลอดภัยในการแก้ไขซอร์สโค้ด โดยการซอยแผนงานใหญ่ (Implementation Plan) ให้เป็นธุรกรรมย่อย (Atomic Micro-Transactions) และป้องกันไม่ให้ AI แก้ไขโค้ดเกินขอบเขตที่กำหนด

---

## 1. 📁 Project Structure

```text
📁โปรเจกต์ของคุณ/
├── 📁 .codex/
│   └── 📁 prompts/
│       ├── orchestrator.md   # คู่มือกฎเหล็กและกลไกสลับทรานแซกชัน [Orchestrator]
│       ├── coder.md          # กรอบการเขียนโค้ดเฉพาะจุดแบบปิด [Coder]
│       ├── reviewer.md       # วิธีการสแกนความปลอดภัยและ Logic Gate [Reviewer]
│       └── devops.md         # คำสั่งควบคุม Git CLI และการซิงค์บอร์ด Pipeline [DevOps]
└── codex-workflow.yaml       # ไฟล์ควบคุมท่อวิ่งงานและการสลับ Model ราย Agent
```

---

## 2. ⚙️ Heterogeneous Model Selection (Model-to-Task Matching)

เพื่อประสิทธิภาพสูงสุดและลดต้นทุน ทรานแซกชันนี้ทำการจัดสรรทรัพยากรประมวลผลแยกตามความซับซ้อนของแต่ละเอเจนท์:

| Agent Alias | Assigned Role | Target LLM Model | Core Reasoning Focus |
| :--- | :--- | :--- | :--- |
| **`orchestrator`** | Core Transaction Manager | **Claude 4.6 Opus** | การคิดเชิงตรรกะและการหั่นสถาปัตยกรรมแผนงานภาพใหญ่ |
| **`coder`** | Isolated Atomic Developer | **GPT-5.5 Medium** | ความสมดุลและความแม่นยำในการเขียน Functional Patch |
| **`reviewer`** | Quality & Security Gatekeeper | **Claude 4.6 Sonnet** | การจับผิดช่องโหว่ Security และการทำ Static Code Audit |
| **`devops`** | Pipeline & CLI Executor | **Gemini 3.5 Flash Medium** | ความเร็วระดับมิลลิวินาทีและการประหยัดต้นทุนในงาน CLI |

---

## 3. 🔄 Execution Workflow Lifecycle

กระบวนการรันงานถูกควบคุมในลักษณะ **State Machine แบบปิด (Synchronous Loop)** ซึ่งทำงานร่วมกันเป็นทอด ๆ ดังนี้:

1. **Phase 1: Planning & Lock State** – `orchestrator` อ่านแผนใหญ่ ล็อกสถานะระบบไม่ให้งานอื่นแทรก และตัดงานชิ้นแรกออกมาในรูปแบบ JSON Constraints
2. **Phase 2: Environment Setup** – `devops` รับคำสั่งมาสร้างกิ่งแยกเฉพาะภารกิจผ่าน Git และปรับบอร์ดติดตามให้มนุษย์รับทราบสถานะแบบเรียลไทม์
3. **Phase 3: Development & Isolation** – `coder` นำโจทย์ไปเขียนโค้ด หากพบว่าต้องแก้ส่วนอื่นนอกเหนือข้อตกลงจะหยุดงานทันทีเพื่อส่งสัญญาณแจ้งข้อจำกัด
4. **Phase 4: Verification Gate** – `devops` ทำการผลักโค้ดและเปิด PR เพื่อเชิญ `reviewer` เข้ามารันสคริปต์ตรวจสอบความปลอดภัยอย่างละเอียด
5. **Phase 5: Finalization & Squash** – เมื่อได้รับการอนุมัติ `devops` จะรันกระบวนการ **Squash and Merge** ยุบรวม Commit ย่อย ๆ ทั้งหมดให้เหลือเพียง 1 Commit เพื่อรักษาประวัติสายโค้ดให้สะอาด และย้ายการ์ดเป็น Done ก่อนที่ `orchestrator` จะคลายล็อกเพื่อรัน Task ถัดไป

---

## 🔀 4. Data Payload Specifications (Inter-Agent Protocols)

การสื่อสารระหว่าง Agent บนระบบไร้รอยต่อผ่านโครงสร้างข้อมูล JSON ที่เข้มงวดเพื่อล็อกขอบเขตของทรานแซกชัน:

### Payload A: Orchestrator ➔ DevOps & Coder (Task Ingestion)
```json
{
  "transaction_id": "TX-20260530-089",
  "task_id": "TSK-402",
  "meta": {
    "estimated_duration_minutes": 15,
    "priority": "HIGH"
  },
  "constraints": {
    "target_files": ["src/middleware/auth.ts"],
    "max_lines_added": 50,
    "forbidden_keywords": ["DROP", "ALTER", "DELETE"]
  },
  "instruction": "Implement validateSubScopeJwt middleware to verify fine-grained scopes from the incoming JWT payload array."
}
```

### Payload B: Coder ➔ DevOps (Artifact Delivery)
```json
{
  "transaction_id": "TX-20260530-089",
  "task_id": "TSK-402",
  "status": "COMPLETED",
  "artifacts": {
    "modified_files": ["src/middleware/auth.ts"],
    "lines_added": 24,
    "lines_deleted": 2,
    "patch_diff": "@@ -10,4 +10,24 @@ ... +export const validateSubScopeJwt = ..."
  }
}
```

### Payload C: Reviewer ➔ DevOps & Orchestrator (Verdict Audit)
```json
{
  "transaction_id": "TX-20260530-089",
  "audit_results": {
    "static_analysis": "PASSED",
    "security_vulnerability_scan": "CLEAN",
    "scope_leak_check": "VALIDATED_NO_LEAK"
  },
  "verdict": "APPROVED",
  "review_comments": []
}
```

---

## 🛡️ 5. Deep Exception & Rollback Handling

ระบบได้รับการออกแบบด้วยสถาปัตยกรรม Fault-Tolerant เพื่อป้องกันไม่ให้โครงสร้างซอร์สโค้ดและระบบฐานข้อมูลเสียหายโดยอัตโนมัติ:

* **Scope Leakage Invalidation:** หาก `coder` แอบแก้ไขโค้ดหรือไฟล์นอกเหนือโครงสร้างอาเรย์ `target_files` ตัว `reviewer` จะทำการดักตรวจจับเจอทันทีตั้งแต่ขั้นตอนแกะ Git Diff และบังคับตีกลับสถานะเป็น `REJECTED` พร้อมบล็อกไม่ให้ประมวลผลการทำงานขั้นต่อไป
* **Circuit Breaker Loop Interceptor:** หากระบบคิวงานย่อยเกิดการแก้ไขและรีวิวโค้ดวนลูปแล้วเกิดข้อผิดพลาดซ้ำเดิม (Reject) ติดต่อกันครบ **3 รอบ** ตัว `orchestrator` จะสั่งปรับสถานะโครงสร้างงานเป็น `SUSPEND` หยุดชะงักการประมวลผลทันที และยิง Alert แจ้งเตือนไปยังมนุษย์เพื่อแก้ปัญหาความล่าช้า
* **Force Workspace Purge:** กรณีที่ธุรกรรมล้มเหลวร้ายแรงจนต้องใช้คำสั่ง `ABORT_TRANSACTION` ตัวเอเจนท์ `devops` จะทำการเคลียร์พื้นที่สภาพแวดล้อม รันคำสั่งทำลายกิ่งย่อยขยะออกไปทั้งระบบ Local และ Remote ทันที เพื่อป้องกันโค้ดค้างสายการผลิต
```
