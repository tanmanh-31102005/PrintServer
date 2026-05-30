# 🖨️ Print Server Management System

<div align="center">

![Print Server](https://img.shields.io/badge/Print%20Server-Windows%20Server-blue?style=for-the-badge&logo=windows)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Domain%20ABC.local-green?style=for-the-badge&logo=microsoft)
![VMware](https://img.shields.io/badge/VMware-Workstation-orange?style=for-the-badge&logo=vmware)
![PowerShell](https://img.shields.io/badge/PowerShell-Scripts-5391FE?style=for-the-badge&logo=powershell)

**Hệ thống Quản lý Máy In Tập Trung (Centralized Printer Server)**  
*Đồ án môn Quản Trị Hệ Thống Mạng — Nhóm 6*

</div>

---

## 📋 Mục Lục

- [Tổng Quan Dự Án](#-tổng-quan-dự-án)
- [Kiến Trúc Hệ Thống](#-kiến-trúc-hệ-thống)
- [Công Nghệ Sử Dụng](#-công-nghệ-sử-dụng)
- [Tính Năng Chính](#-tính-năng-chính)
- [Cấu Hình Hệ Thống](#-cấu-hình-hệ-thống)
- [Cấu Trúc Thư Mục](#-cấu-trúc-thư-mục)
- [Hướng Dẫn Triển Khai](#-hướng-dẫn-triển-khai)
- [Demo Script](#-demo-script)
- [Nhóm Thực Hiện](#-nhóm-thực-hiện)

---

## 🎯 Tổng Quan Dự Án

Dự án xây dựng một **hệ thống quản lý máy in tập trung** cho doanh nghiệp vừa và nhỏ, sử dụng **Windows Server** làm Print Server kết hợp với **Active Directory Domain Services (AD DS)** để quản lý người dùng và phân quyền truy cập máy in theo từng phòng ban.

### 🎯 Mục Tiêu

- Tập trung hóa quản lý toàn bộ máy in trong tổ chức
- Phân quyền in ấn theo phòng ban thông qua Active Directory Groups
- Triển khai Printer Pool để cân bằng tải công việc in
- Thiết lập độ ưu tiên (Priority) cho từng máy in/phòng ban
- Tự động phân phối máy in đến client thông qua Group Policy (GPO)
- Giám sát và ghi log hoạt động in ấn

---

## 🏗️ Kiến Trúc Hệ Thống

```
                        ┌─────────────────────────┐
                        │   Domain: ABC.local      │
                        │   Domain Controller      │
                        │   + DNS Server           │
                        │   + AD DS                │
                        │   + Print Server         │
                        │   IP: 192.168.244.x      │
                        └────────────┬────────────┘
                                     │ (Domain Join)
               ┌─────────────────────┼──────────────────────┐
               │                     │                      │
     ┌─────────▼──────┐   ┌──────────▼──────┐   ┌──────────▼──────┐
     │  user_design   │   │  user_office    │   │user_accounting  │
     │ (VM Client 1)  │   │ (VM Client 2)   │   │ (VM Client 3/4) │
     │ GRP_Design     │   │ GRP_Office      │   │ GRP_Accounting  │
     └─────────┬──────┘   └──────────┬──────┘   └──────────┬──────┘
               │                     │                      │
     ┌─────────▼──────┐   ┌──────────▼──────┐   ┌──────────▼──────┐
     │ PRN_Design_01  │   │ PRN_Office_01   │   │PRN_Accounting_01│
     │ PRN_Design_02  │   │ PRN_Office_02   │   │PRN_Accounting_02│
     │ (In màu)       │   │ PRN_Office_03   │   │ (In hóa đơn)    │
     │ Priority: 50   │   │ PRN_Pool_Office │   │ Priority: 75    │
     └────────────────┘   │ (In B/W)        │   └─────────────────┘
                          │ Priority: 25/30 │
                          └─────────────────┘
```

---

## 🛠️ Công Nghệ Sử Dụng

| Công Nghệ | Phiên Bản | Mục Đích |
|-----------|-----------|---------|
| **Windows Server** | 2019/2022 | Domain Controller & Print Server |
| **Active Directory Domain Services** | — | Quản lý người dùng & phân quyền |
| **Windows Print Services** | — | Dịch vụ quản lý máy in tập trung |
| **Group Policy (GPO)** | — | Tự động phân phối máy in |
| **HP Universal Print Driver (PCL6)** | 7.9.0.26347 | Driver máy in |
| **VMware Workstation** | v21 | Ảo hóa môi trường lab |
| **PowerShell** | 5.1+ | Scripts quản trị & kiểm tra hệ thống |
| **Windows 8.1 Enterprise x64** | Build 9600 | Client OS (VM) |

---

## ✨ Tính Năng Chính

### 1. 🗂️ Quản Lý Active Directory
- Tạo và quản lý **Organizational Units (OU)** theo cấu trúc doanh nghiệp
- Quản lý **Groups** theo phòng ban: `GRP_Design`, `GRP_Office`, `GRP_Accounting`
- Quản lý **User Accounts**: `user_design`, `user_office`, `user_accounting`

### 2. 🖨️ Quản Lý Máy In Tập Trung

| Máy In | Nhóm | Loại | Độ Ưu Tiên |
|--------|------|------|------------|
| PRN_Design_01 | GRP_Design | In màu | 50 |
| PRN_Design_02 | GRP_Design | In màu | 50 |
| PRN_Office_01 | GRP_Office | In B/W | 25 |
| PRN_Office_02 | GRP_Office | In B/W | 25 |
| PRN_Office_03 | GRP_Office | In B/W | 25 |
| PRN_Pool_Office | GRP_Office | Pool B/W | 30 |
| PRN_Accounting_01 | GRP_Accounting | In hóa đơn | 75 |
| PRN_Accounting_02 | GRP_Accounting | In hóa đơn | 75 |

### 3. 🔄 Printer Pool
- Cấu hình **PRN_Pool_Office** với nhiều cổng TCP/IP (IP_192.168.244.20x)
- Tự động cân bằng tải công việc in giữa các máy in trong pool
- Tăng tính sẵn sàng (High Availability) cho phòng Văn Phòng

### 4. 🔐 Phân Quyền Truy Cập (ACL)
- Mỗi nhóm AD chỉ được phép truy cập máy in của phòng ban mình
- Quản trị viên có quyền quản lý toàn bộ hệ thống in ấn
- Không cho phép cross-department printing

### 5. 📊 Độ Ưu Tiên (Print Priority)
```
Kế Toán (75) > Thiết Kế (50) > Pool VP (30) > Văn Phòng (25)
```
- Công việc in của Kế Toán luôn được ưu tiên xử lý trước
- Hỗ trợ SLA (Service Level Agreement) cho từng phòng ban

### 6. 📜 Group Policy Objects (GPO)
- Tự động deploy máy in đến các client khi đăng nhập domain
- Đặt máy in mặc định (Default Printer) theo nhóm người dùng
- Áp dụng chính sách in ấn cho toàn bộ domain

### 7. 📋 Logging & Monitoring
- Ghi log hoạt động in tại `C:\PrintLogs`
- Theo dõi trạng thái hàng đợi in (Print Queue) theo thời gian thực
- Kiểm tra kết nối mạng tới các máy in qua Ping

---

## ⚙️ Cấu Hình Hệ Thống

### Server Configuration
```
Domain Name  : ABC.local
Server Name  : SRV-PRINT
Network      : 192.168.244.0/24
Print Driver : HP Universal PCL6 v7.9.0.26347
Spooler      : Running (Auto Start)
DNS          : Running (Auto Start)
AD DS (NTDS) : Running (Auto Start)
```

### Virtual Machines (VMware)
```
VM Server    : Windows Server (Domain Controller + Print Server)
VM Client 1  : user_design  — Windows 8.1 x64 — 2048MB RAM
VM Client 2  : user_office  — Windows 8.1 x64 — 2048MB RAM
VM Client 3  : user_design  — Windows 8.1 x64 — 2048MB RAM (Clone)
VM Client 4  : user_office  — Windows 8.1 x64 — 2048MB RAM (Clone)
Network Mode : NAT
```

### Print Ports (TCP/IP)
- Dải IP máy in: `192.168.244.201` → `192.168.244.210`
- Protocol: **Standard TCP/IP Port**
- Port Name format: `IP_192.168.244.2xx`

---

## 📁 Cấu Trúc Thư Mục

```
QTHTM/
├── 📄 README.md                          # Tài liệu dự án (file này)
├── 📄 Kịch bản demo Server.txt           # Script PowerShell demo hệ thống (12 phần)
├── 📦 upd-pcl6-x64-7.9.0.26347.zip      # HP Universal Print Driver PCL6 (x64)
├── 📁 upd-pcl6-x64-7.9.0.26347/         # Driver đã giải nén
│
├── 📁 52300221_52300240_52300253_N6/     # Tài liệu báo cáo (bản gốc)
│   ├── 📄 *_Baocao.pdf                  # Báo cáo đầy đủ dự án
│   └── 📄 *_Trinhbay.pdf               # Slide trình bày
│
├── 📁 QTM_N1_B6/                        # Tài liệu báo cáo (bản chính)
│   ├── 📄 *_Baocao.pdf                  # Báo cáo chi tiết
│   ├── 📄 *_Trinhbay.pdf               # Slide trình bày
│   └── 📁 Source Latex/                # Source code LaTeX của báo cáo
│       ├── 📁 Bài Báo Cáo/            # Source LaTeX báo cáo
│       └── 📁 Trình Bày/              # Source LaTeX slide
│
├── 📁 Drivers/                          # Driver máy in
│   └── 📁 Dot4/AMD64/                  # IEEE 1284.4 USB Driver (x64)
│
└── 📁 user/                             # VMware VMs (Client Machines)
    ├── 🖥️ Clone of Windows 8.x x64 (2).vmx   # VM user_ofice config
    ├── 💾 Windows 8.x x64-cl[1-4]*.vmdk       # VM Disk (4 clients)
    ├── 🖥️ user_design.vmx                      # VM Thiết kế config
    ├── 💾 user_design-s00*.vmdk                # VM Disk - Design
    └── 🖥️ user_office.vmx                      # VM Văn phòng config
```

---

## 🚀 Hướng Dẫn Triển Khai

### Yêu Cầu Hệ Thống

- **Host Machine**: Windows 10/11 với ít nhất 16GB RAM, 100GB disk trống
- **VMware Workstation**: Phiên bản 16+ hoặc VMware Player
- **Phần mềm**: PowerShell 5.1+, RSAT Tools

### Bước 1: Cài Đặt Domain Controller

```powershell
# Cài đặt AD DS Role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Tạo Domain mới
Install-ADDSForest `
    -DomainName "ABC.local" `
    -DomainNetbiosName "ABC" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -Force:$true
```

### Bước 2: Cài Đặt Print Server

```powershell
# Cài đặt Print Services
Install-WindowsFeature -Name Print-Services -IncludeManagementTools

# Kiểm tra dịch vụ Spooler
Get-Service -Name Spooler | Start-Service
Set-Service -Name Spooler -StartupType Automatic
```

### Bước 3: Cài Đặt Driver HP PCL6

```powershell
# Giải nén và cài đặt driver
Add-PrinterDriver -Name "HP Universal Printing PCL 6" -InfPath ".\upd-pcl6-x64-7.9.0.26347\*.inf"
```

### Bước 4: Tạo Máy In & Cấu Hình Port

```powershell
# Tạo TCP/IP Port
Add-PrinterPort -Name "IP_192.168.244.201" -PrinterHostAddress "192.168.244.201"

# Tạo máy in và chia sẻ
Add-Printer -Name "PRN_Design_01" `
            -PortName "IP_192.168.244.201" `
            -DriverName "HP Universal Printing PCL 6" `
            -Shared:$true `
            -ShareName "PRN_Design_01"

# Đặt độ ưu tiên
Set-Printer -Name "PRN_Design_01" -Priority 50
```

### Bước 5: Tạo Groups & Users trong AD

```powershell
# Tạo Groups
New-ADGroup -Name "GRP_Design"     -GroupScope Global -Path "OU=Groups,DC=ABC,DC=local"
New-ADGroup -Name "GRP_Office"     -GroupScope Global -Path "OU=Groups,DC=ABC,DC=local"
New-ADGroup -Name "GRP_Accounting" -GroupScope Global -Path "OU=Groups,DC=ABC,DC=local"

# Tạo Users
New-ADUser -Name "user_design"     -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Enabled $true
New-ADUser -Name "user_office"     -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Enabled $true
New-ADUser -Name "user_accounting" -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Enabled $true

# Thêm vào Groups
Add-ADGroupMember -Identity "GRP_Design"     -Members "user_design"
Add-ADGroupMember -Identity "GRP_Office"     -Members "user_office"
Add-ADGroupMember -Identity "GRP_Accounting" -Members "user_accounting"
```

### Bước 6: Cấu Hình Printer Pool

```powershell
# Tạo Pool Printer (nhiều port cho 1 máy in)
Add-PrinterPort -Name "IP_192.168.244.205" -PrinterHostAddress "192.168.244.205"
Add-PrinterPort -Name "IP_192.168.244.206" -PrinterHostAddress "192.168.244.206"

Add-Printer -Name "PRN_Pool_Office" `
            -PortName "IP_192.168.244.205,IP_192.168.244.206" `
            -DriverName "HP Universal Printing PCL 6" `
            -Shared:$true -ShareName "PRN_Pool_Office"
Set-Printer -Name "PRN_Pool_Office" -Priority 30
```

---

## 📺 Demo Script

File `Kịch bản demo Server.txt` chứa script PowerShell kiểm tra toàn bộ hệ thống gồm **12 phần**:

| Phần | Nội Dung |
|------|----------|
| **Phần 1** | Kiểm tra Domain Controller & cấu trúc OU |
| **Phần 2** | Kiểm tra Groups & Users Active Directory |
| **Phần 3** | Kiểm tra dịch vụ Print Spooler, DNS, NTDS |
| **Phần 4** | Liệt kê máy in, cổng TCP/IP, Printer Pool |
| **Phần 5** | Bảng phân quyền máy in theo phòng ban |
| **Phần 6** | Danh sách độ ưu tiên in ấn theo phòng ban |
| **Phần 7** | Kiểm tra Group Policy Objects (GPO) |
| **Phần 8** | Kiểm tra Logging & Event Log |
| **Phần 9** | Hàng đợi in hiện tại (Print Queue) |
| **Phần 10** | Ping kiểm tra kết nối mạng tới máy in |
| **Phần 11** | Danh sách máy tính đã join domain |
| **Phần 12** | Tổng hợp trạng thái toàn bộ hệ thống |

**Cách chạy:**
```powershell
# Chạy trên Domain Controller (với quyền Admin)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.\KichBanDemoServer.ps1
```

---

## 📊 Kết Quả Đạt Được

- ✅ Hệ thống Domain `ABC.local` hoạt động ổn định
- ✅ **8 máy in** được cấu hình và chia sẻ qua mạng
- ✅ **3 nhóm phòng ban** với phân quyền độc lập
- ✅ **Printer Pool** tự động cân bằng tải
- ✅ Độ ưu tiên in ấn được phân cấp rõ ràng
- ✅ GPO tự động phân phối máy in khi client login
- ✅ Log hoạt động in ấn được ghi nhận đầy đủ
- ✅ **4 VM Client** đã join domain và hoạt động bình thường

---

## 👥 Nhóm Thực Hiện

| MSSV | Họ và Tên | Vai Trò |
|------|-----------|---------|
| **52300221** | Thành viên 1 | Cấu hình Domain Controller & AD DS |
| **52300240** | Thành viên 2 | Cấu hình Print Server & GPO |
| **52300253** | Thành viên 3 | Cấu hình Client, Test & Báo cáo |

> **Nhóm 6** — Môn: Quản Trị Hệ Thống Mạng

---

## 📚 Tài Liệu Tham Khảo

- [Microsoft Docs — Print and Document Services](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/print-and-document-services)
- [Microsoft Docs — Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [HP Universal Print Driver — Download](https://support.hp.com/us-en/drivers/hp-universal-print-driver-series-for-windows)
- [VMware Workstation Documentation](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)

---

<div align="center">

**© 2025 Nhóm 6 — Quản Trị Hệ Thống Mạng**

*Dự án được thực hiện cho mục đích học thuật tại trường đại học.*

</div>
