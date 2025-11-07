# Process Hollowing Educational Project

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/yourusername/process-hollowing-project.svg?style=social)](https://github.com/yourusername/process-hollowing-project/stargazers)

## Description (Mô tả dự án)

Dự án này là một bài tập giáo dục về kỹ thuật Process Hollowing trong lĩnh vực an ninh mạng, tập trung vào việc hiểu cơ chế hoạt động của mã độc để xây dựng khả năng nhận diện và phòng thủ. Dự án bao gồm việc triển khai Proof-of-Concept (PoC) mô phỏng cách malware sử dụng kỹ thuật này, cải thiện khả năng bypass AV (như Windows Defender), và phát triển công cụ phát hiện. 

**Mục tiêu chính:** 
- Hiểu sâu về Process Hollowing để phát triển quy tắc giám sát và ứng phó.
- Không dành cho mục đích tấn công; chỉ sử dụng trong môi trường lab cô lập (VM) với dummy payload không gây hại.
- Dự án dựa trên các nguồn mở như repo GitHub (m0n0ph1, NATsCodes) và hướng dẫn từ red-team-sncf.github.io, kết hợp papers học thuật (ví dụ: "Ten Process Injection Techniques Survey" từ Elastic, 2017).

Dự án đạt yêu cầu học tập: Phần A (PoC: 4đ), Phần B (Bypass AV: 2đ), Phần C (Detection Tool: 4đ). Cam kết: Dự án chỉ phục vụ giáo dục, không gây hại, và được kiểm tra bởi giảng viên.

## Concept (Khái niệm về Process Hollowing)

Process Hollowing là một kỹ thuật mã độc (malware) sử dụng để ẩn náu và tránh phát hiện. Kẻ tấn công khởi tạo một tiến trình hợp pháp (như svchost.exe) ở trạng thái suspended (tạm dừng), sau đó thay thế nội dung bộ nhớ thực thi gốc bằng code độc hại (payload), và cuối cùng resume tiến trình để code độc chạy dưới vỏ bọc tiến trình hợp pháp.

**Lý do kẻ tấn công sử dụng:**
- **Ẩn náu:** Malware trông như tiến trình hệ thống bình thường, tránh signature-based detection từ AV.
- **Evasion:** Không tạo file mới trên đĩa, giảm dấu vết; có thể bypass AV bằng cách mimic hành vi hợp pháp (không unmap image gốc để tránh crash).
- **API liên quan chính:** CreateProcess (với CREATE_SUSPENDED), VirtualAllocEx, WriteProcessMemory, SetThreadContext, ResumeThread. Trong phiên bản "complete", thêm fix relocations (reloc table), import address table (IAT patching), và xử lý API sets (e.g., api-ms-win-* forwarded to combase.dll).

**IOC/IOA (Indicators of Compromise/Attack):**
- Tiến trình hợp pháp với memory write bất thường từ process khác.
- Thay đổi thread context hoặc PEB (Process Environment Block) ImageBase.
- Module load lạ, mismatch signer/parent process, CPU spikes.

Dự án sử dụng dummy payload (in thông điệp đơn giản) để mô phỏng an toàn, tập trung vào học cơ chế để phòng thủ (e.g., Sysmon rules, EDR hardening).

## Execution (Thực thi và Hướng dẫn sử dụng)

### Yêu cầu hệ thống
- Windows 10/11 (VM cô lập, không kết nối mạng sản xuất).
- Visual Studio 2022 hoặc MinGW để compile C/C++.
- Công cụ giám sát: Procmon, Process Explorer, Sysmon (tải từ Microsoft Sysinternals).
- Quyền admin để chạy PoC (trong lab only).
- Snapshot VM trước khi chạy để rollback.

### Cài đặt (Installation)
1. Clone repo: `git clone https://github.com/Koiiin/Process-Hollowing/`
2. Mở Visual Studio hoặc MinGW, import các file code (hollow.cpp, dummy.c, detection.cpp).
3. Compile dummy payload: `cl dummy.c /o dummy.exe` (hoặc gcc).
4. Compile PoC: `cl hollow.cpp /o hollow.exe /EHsc` (thêm /D_WIN64 cho x64).
5. Compile detection tool: `cl detection.cpp /o detector.exe`.
6. Cài Sysmon: Tải và config với rules.xml (bao gồm Rule 1-3 từ dự án).

### Sử dụng (Usage)
#### Phần A: Chạy PoC
- Chạy: `hollow.exe` (target svchost.exe, inject dummy.exe).
- Quan sát: Sử dụng Process Explorer để xem memory changes, thread resume. Output: "Hollowed Process Simulated!".
- Log với Procmon: Filter cho API như WriteProcessMemory.

#### Phần B: Test Bypass AV
- Tắt Defender real-time tạm thời (lab only): Settings > Update & Security > Windows Security > Virus & threat protection > Manage settings.
- Chạy PoC với obfuscation (XOR payload, dynamic API load).
- Kiểm tra: Không flag từ Defender; so log trước/sau cải tiến.

#### Phần C: Chạy Detection Tool
- Chạy: `detector.exe` (monitor background).
- Test: Chạy PoC, tool sẽ log "Hollowing detected in PID X".
- Tích hợp Sysmon: Áp dụng rules, kiểm tra event viewer cho alerts.

**Cảnh báo an toàn:** Chỉ chạy trong VM snapshot. Không sử dụng ngoài lab. Nếu gặp lỗi, kiểm tra quyền admin hoặc bit compatibility (x64).

### Tài liệu bổ sung
- **Báo cáo:** Xem file report.md cho tóm tắt phát hiện, sơ đồ chuỗi sự kiện (Draw.io), và khuyến nghị (AppLocker, least privilege).
- **References:** 
  - https://github.com/m0n0ph1/Process-Hollowing
  - https://red-team-sncf.github.io/complete-process-hollowing.html
  - Papers: "A New Approach for Detecting Process Injection Attacks" (IITIS, 2023).

## License
MIT License. Xem LICENSE file để chi tiết.

**Liên hệ:** Nếu có câu hỏi, mở issue trên GitHub. Dự án này dành cho học tập, không khuyến khích sử dụng sai mục đích.
