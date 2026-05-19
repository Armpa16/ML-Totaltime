# Data Dictionary — WMS-ML

อธิบายความหมายของคอลัมน์ในไฟล์ข้อมูลหลักที่ใช้ในโปรเจกต์นี้

---

## 1. PalletLogs.csv

**แหล่งที่มา:** ระบบ WMS (Warehouse Management System) โรงงาน SCG Roofing  
**จำนวนแถว:** ~399,233 รายการ  
**คำอธิบาย:** Log การเคลื่อนย้าย Pallet (พาเลท) แต่ละใบในโกดัง ทุก Action ที่พนักงานหรือรถยกทำกับ Pallet จะถูกบันทึกที่นี่

| คอลัมน์ | ประเภทข้อมูล | ความหมาย | ตัวอย่าง |
|---|---|---|---|
| `PalletID` | string | รหัสประจำตัว Pallet แต่ละใบ (unique per pallet) | `SB1-1LS3690515P025` |
| `Action` | string | ประเภทการกระทำที่บันทึก ในชุดข้อมูลนี้มีค่าเดียวคือ `POSTTOCHECKERLOCATION` = การนำ Pallet จากสต็อกไปที่ลานจ่ายสินค้า | `POSTTOCHECKERLOCATION` |
| `PalletStatus` | string | สถานะของ Pallet ณ ขณะบันทึก | ดูตาราง **PalletStatus** ด้านล่าง |
| `PackSize` | int | ขนาดมาตรฐานของ Pallet (จำนวนแผ่นหรือชิ้นต่อพาเลทตามสูตรการแพ็ก) | `320`, `180`, `240` |
| `Amount` | int | จำนวนสินค้าจริงบนพาเลท (อาจน้อยกว่า PackSize ถ้าเป็น Pallet แตก/เศษ) | `320`, `180` |
| `LotNo` | string | รหัส Lot การผลิต | `3690515P` |
| `MaterialCode` | string | รหัสสินค้าใน SAP | `ZCB10STDA002000A14` |
| `NameThai` | string | ชื่อสินค้าภาษาไทย | `ก/บ คอนกรีต SCG เอลาบานา สีน้ำตาลโอ๊คแดง` |
| `ProductType` | string | ประเภทสินค้า | ดูตาราง **ProductType** ด้านล่าง |
| `Model` | string | แบรนด์/รุ่นสินค้า | `CPAC`, `PRESTIGE` |
| `Profile` | string | รูปทรงของกระเบื้อง | `ELA`, `RB` |
| `Curve` | string | ชื่อ Curve ของกระเบื้อง | `ELABANA`, `CLASSIC` |
| `Color` | float | รหัสสีสินค้า | |
| `Coat` | string | ประเภทผิวเคลือบ | `Pigmented GEP`, `Plain` |
| `ProductClass` | string | ชั้นของสินค้า (A=ปกติ, B/C=ด้อยคุณภาพ) | `A`, `B`, `C` |
| `StampDateTime` | string | วันและเวลาที่บันทึก Log (format: `DD/MM/YYYY HH:MM:SS`) | `18/05/2026 10:51:20` |
| `FromLocationType` | string | ประเภทตำแหน่งต้นทางของ Pallet | ดูตาราง **LocationType** ด้านล่าง |
| `FromLocationCode` | string | รหัสช่องเก็บต้นทาง | `LCR20200700048` |
| `FromLocationName` | string | ชื่อช่องเก็บต้นทาง เช่น `A-48 C23 F1` = แถว A-48, คอลัมน์ 23, ชั้น 1 | `A-48 C23 F1` |
| `FromRowNo` | float | หมายเลขแถวของตำแหน่งต้นทาง | `48` |
| `FromColNo` | int | หมายเลขคอลัมน์ของตำแหน่งต้นทาง | `23` |
| `FromFloorNo` | int | หมายเลขชั้นของตำแหน่งต้นทาง | `1` |
| `ToLocationType` | string | ประเภทตำแหน่งปลายทาง ในชุดข้อมูลนี้เป็น `POSTLOCATION` ทั้งหมด = ลานจ่ายสินค้า | `POSTLOCATION` |
| `ToLocationCode` | string | รหัสช่องลานจ่ายปลายทาง | `PL20100001`, `COM22090001` |
| `ToLocationName` | string | ชื่อช่องลานจ่ายปลายทาง | `ลานจ่าย 1 ช่องจ่าย 1 (CPAC)` |
| `ToRowNo` | float | หมายเลขแถวของตำแหน่งปลายทาง | |
| `ToColNo` | int | หมายเลขคอลัมน์ของตำแหน่งปลายทาง | |
| `ToFloorNo` | int | หมายเลขชั้นของตำแหน่งปลายทาง | |
| `UserCreate` | string | ชื่อพนักงานที่บันทึกการเคลื่อนย้าย | `ชัยณรงค์ เพ็ชรนิล` |
| `UserCreateCode` | int | รหัสพนักงาน | `2102000002` |
| `PlantCode` | float | รหัสโรงงาน (ในชุดข้อมูลนี้ยังไม่มีค่า — null ทั้งหมด) | |

### PalletStatus

| ค่า | ความหมาย |
|---|---|
| `FOLKPOSTEND` | รถยกนำ Pallet ไปวางที่ลานจ่ายแล้ว |
| `GOOD` | Pallet สถานะปกติ/ดี |
| `POSTCOMPLETED` | การ Post Pallet เสร็จสมบูรณ์ |
| `PACK` | Pallet อยู่ระหว่างการแพ็กสินค้า |

### ProductType

| ค่า | ความหมาย |
|---|---|
| `CPAC-Tile` | กระเบื้องคอนกรีต แบรนด์ CPAC |
| `CPAC-Fitting` | อุปกรณ์ประกอบ แบรนด์ CPAC (ครอบ, โครงหลังคา ฯลฯ) |
| `PRESTIGE-Tile` | กระเบื้องคอนกรีต แบรนด์ PRESTIGE |
| `PRESTIGE-Fitting` | อุปกรณ์ประกอบ แบรนด์ PRESTIGE |
| `DURA-Fitting` | อุปกรณ์ประกอบ แบรนด์ DURA |
| `ACCESSORIES` | อุปกรณ์เสริม/อื่นๆ |

### LocationType (FromLocationType)

| ค่า | ความหมาย |
|---|---|
| `NORMAL` | ช่องเก็บสินค้าปกติในโกดัง |
| `BUFFER` | พื้นที่พักสินค้าชั่วคราว (Buffer Zone) |
| `FRACTION` | ช่องเก็บ Pallet แตก (สินค้าไม่ครบ Pack) |
| `PALLETSOURCE` | แหล่งที่มาเป็น Pallet ว่าง / ยังไม่ระบุตำแหน่งเดิม |

---

## 2. vwTimeStampDashboard_v3.csv

**แหล่งที่มา:** View รวมข้อมูลจาก WMS สำหรับ Dashboard การวิเคราะห์การจ่ายสินค้า  
**จำนวนแถว:** ~32,505 รายการ  
**คำอธิบาย:** ข้อมูลระดับ PickList (ใบสั่งจ่าย) หนึ่งแถว = รถบรรทุกหนึ่งคัน บันทึก Timestamp สำคัญตลอด flow การจ่ายสินค้า ตั้งแต่รถมาถึงจนออกจากโรงงาน

### คอลัมน์หลัก

| คอลัมน์ | ประเภทข้อมูล | ความหมาย | ตัวอย่าง |
|---|---|---|---|
| `PlantName` | string | ชื่อโรงงาน | `SB1` |
| `PickListType` | string | ประเภทการจ่ายสินค้า | ดูตาราง **PickListType** ด้านล่าง |
| `PickDate` | datetime | วันที่สร้าง PickList | `2025-01-07 11:10:04` |
| `TruckSeqNo` | float | ลำดับคิวรถในวันนั้น | `8`, `26` |
| `CarType` | float | รหัสประเภทรถ (SAP material number) | `1000000003`, `1000000008` |
| `CarNo` | string | ทะเบียนรถ | `89-0471` |
| `PackListNo` | string | รหัส PickList / ใบสั่งจ่าย | `SB1PL250107022` |
| `CustomerName` | string | ชื่อลูกค้า | `บ.จำหน่ายวัตถุก่อสร้าง จก. สาขา 3` |
| `PostLocationName` | string | ชื่อลานจ่ายที่รับสินค้า | `ลานจ่าย 1 ช่องจ่าย 2 (CPAC)` |
| `PrepareForward` | string | รถมา Forward (จองคิวล่วงหน้า) หรือไม่: `Y`=ใช่, `N`=ไม่ใช่ | `N`, `Y` |

### คอลัมน์ Timestamp (ลำดับเวลา flow การจ่ายสินค้า)

```
QueueTime → CreateDate → PickingTime → OperatorCarConfirm → CarConfirm
    → PostingTime → FirstPostPallet → LastPostPallet
```

| คอลัมน์ | ความหมาย |
|---|---|
| `QueueTime` | เวลาที่รถมาถึงและเข้าคิว |
| `TruckReceiveDate` | วันที่กำหนดรับรถ (อาจต่างจาก QueueTime ถ้าจอง Forward) |
| `TruckReceiveHour` | ชั่วโมงที่กำหนดรับรถ |
| `TruckReceiveMinute` | นาทีที่กำหนดรับรถ |
| `CreateDate` | เวลาที่สร้าง PickList ในระบบ |
| `PickingTime` | เวลาที่เริ่มหยิบสินค้า (พนักงานกด Start) |
| `OperatorCarConfirm` | เวลาที่พนักงานยืนยันรถ (Operator confirm) |
| `CarConfirm` | เวลาที่ระบบยืนยันรถเข้าลาน |
| `PostingTime` | เวลาที่ Post สินค้าเสร็จทั้งหมด (งาน WMS เสร็จ) |
| `FirstPostPallet` | เวลาที่ Post Pallet ใบแรก |
| `LastPostPallet` | เวลาที่ Post Pallet ใบสุดท้าย |

### คอลัมน์ Timestamp แยกตามประเภทสินค้า

| คอลัมน์ | ความหมาย |
|---|---|
| `TileStart` | เวลาเริ่มหยิบกระเบื้อง (Tile) |
| `TileEnd` | เวลาเสร็จหยิบกระเบื้อง |
| `FittingStart` | เวลาเริ่มหยิบอุปกรณ์ประกอบ (Fitting) |
| `FittingEnd` | เวลาเสร็จหยิบ Fitting |
| `AccStart` | เวลาเริ่มหยิบอุปกรณ์เสริม (Accessories) |
| `AccEnd` | เวลาเสร็จหยิบ Accessories |

### คอลัมน์ปริมาณสินค้า (SAP Amount)

จำนวนสินค้าแต่ละประเภทใน PickList นี้ (หน่วย: แผ่น/ชิ้น)

| คอลัมน์ | ความหมาย |
|---|---|
| `CPACTileSapAmount` | จำนวนกระเบื้อง CPAC |
| `PRESTIGETileSapAmount` | จำนวนกระเบื้อง PRESTIGE |
| `NEUSTILETileSapAmount` | จำนวนกระเบื้อง NEUSTILE |
| `CPACFittingSapAmount` | จำนวน Fitting CPAC |
| `PRESTIGEFittingSapAmount` | จำนวน Fitting PRESTIGE |
| `NEUSTILEFittingSapAmount` | จำนวน Fitting NEUSTILE |
| `DURAFittingSapAmount` | จำนวน Fitting DURA |
| `ACCESSORIESSapAmount` | จำนวน Accessories |

### คอลัมน์อื่น

| คอลัมน์ | ความหมาย |
|---|---|
| `TruckStatus` | สถานะปัจจุบันของรถ: `Loading`=กำลังโหลด |
| `PackListStatus` | สถานะใบจ่าย: `OPERATORCOMPLETED`=เสร็จแล้ว, `Loading`=กำลังดำเนินการ |
| `TruckOverTimeName` | ชื่อเหตุผลที่รถ Overtime (ถ้ามี) |
| `TruckOverTimeRemark` | หมายเหตุ Overtime (ถ้ามี) |

### PickListType

| ค่า | ความหมาย |
|---|---|
| `Walk-in` | รถที่มาโดยไม่ได้จองคิวล่วงหน้า |
| `SmartQ` | รถที่จองคิวผ่านระบบ SmartQ (Forward queue) |

---

## หมายเหตุคุณภาพข้อมูล

| ไฟล์ | ปัญหาที่พบ |
|---|---|
| **PalletLogs.csv** | `PlantCode` เป็น null ทั้งหมด / Pallet ที่มาจาก `PALLETSOURCE` ไม่มีข้อมูล From Location |
| **vwTimeStampDashboard_v3.csv** | บางแถวมีค่า `TruckStatus`, `CarType`, `PrepareForward` เป็น Excel serial number (เช่น `45681.42`) แทนที่จะเป็น datetime หรือ string — เกิดจาก encoding ผิดพลาดตอน export / `FirstPostPallet` และ `LastPostPallet` มี null ~1% / `TileStart`/`TileEnd` null ~17% (รถบางคันอาจรับเฉพาะ Fitting) / `Unnamed: 39` เป็นคอลัมน์ว่างสามารถ drop ได้ |
