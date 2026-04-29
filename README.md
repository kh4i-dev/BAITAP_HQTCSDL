# BÀI KIỂM TRA SỐ 2 – HỆ QUẢN TRỊ CSDL

## Thông tin sinh viên

- Họ tên: TRẦN VĂN KHẢI
- Mã SV: K235480106035
- Chủ đề: Quản lý kho bán gear cho game thủ

---

# PHẦN 1: THIẾT KẾ DATABASE

## Ảnh 1: Tạo Database

- Lệnh SQL:

```sql
CREATE DATABASE [QuanLyGear_K235480106035];
GO
USE [QuanLyGear_K235480106035];
GO
```

- Mục đích: tạo cơ sở dữ liệu cho hệ thống quản lý gear
- Kết quả: database được tạo thành công

![Ảnh 1](pic_1.png)

---

## Ảnh 2: Tạo bảng LoaiSanPham

- Lệnh SQL:

```sql
CREATE TABLE [LoaiSanPham] (
    [MaLoai] INT PRIMARY KEY,
    [TenLoai] NVARCHAR(100) NOT NULL
);
```

- Mục đích: lưu loại sản phẩm (chuột, bàn phím,...)
- Kết quả: bảng được tạo thành công

![Ảnh 2](pic_2.png)

---

## Ảnh 3: Tạo bảng SanPham

```sql
CREATE TABLE [SanPham] (
    [MaSP] INT PRIMARY KEY,
    [TenSP] NVARCHAR(150) NOT NULL,
    [Gia] MONEY CHECK ([Gia] > 0),
    [SoLuongTon] INT CHECK ([SoLuongTon] >= 0),
    [MaLoai] INT
);
```

- Mục đích: lưu thông tin sản phẩm
- Ràng buộc:
  - PK: MaSP
  - CHECK: Giá > 0

![Ảnh 3](pic_3.png)

---

## Ảnh 4: Tạo khóa ngoại

```sql
ALTER TABLE [SanPham]
ADD CONSTRAINT FK_SP_Loai
FOREIGN KEY ([MaLoai]) REFERENCES [LoaiSanPham]([MaLoai]);
```

- Mục đích: liên kết sản phẩm với loại

![Ảnh 4](pic_4.png)

---

# PHẦN 2: FUNCTION

## Ảnh 5: Built-in Function

```sql
SELECT GETDATE();
SELECT SUM(Gia) FROM SanPham;
```

- Ý nghĩa:
  - GETDATE(): lấy ngày hiện tại
  - SUM(): tính tổng

    ![Ảnh 5](pic_5.png)

---

## Ảnh 6: Scalar Function – Tính tổng tiền đơn hàng

```sql
CREATE FUNCTION fn_TongTienDon (@MaDH INT)
RETURNS MONEY
AS
BEGIN
    DECLARE @tong MONEY;

    SELECT @tong = SUM(SoLuong * DonGia)
    FROM ChiTietDonHang
    WHERE MaDH = @MaDH;

    RETURN @tong;
END;
GO
```

## Ảnh 7: Gọi Function

```sql
SELECT dbo.fn_TongTienDon(1);
```

- Mục đích: tính tổng tiền đơn hàng
- Kết quả: trả về tổng tiền

![Ảnh 7](pic_7.png)

---

# PHẦN 3: STORED PROCEDURE

## Ảnh 8: SP thêm sản phẩm

```sql
CREATE PROCEDURE sp_ThemSanPham
    @MaSP INT,
    @TenSP NVARCHAR(150),
    @Gia MONEY,
    @SoLuong INT,
    @MaLoai INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM SanPham WHERE MaSP = @MaSP)
        PRINT N'Sản phẩm đã tồn tại';
    ELSE
        INSERT INTO SanPham
        VALUES (@MaSP, @TenSP, @Gia, @SoLuong, @MaLoai);
END;
GO
```

## Ảnh 9: Gọi SP

```sql
EXEC sp_ThemSanPham 3, N'Bàn phím cơ AKKO', 1500000, 10, 2;
```

![Ảnh 9](pic_9.png)

---

# PHẦN 4: TRIGGER

## Ảnh 10: Trigger trừ kho

```sql
CREATE TRIGGER trg_BanHang
ON ChiTietDonHang
AFTER INSERT
AS
BEGIN
    UPDATE SanPham
    SET SoLuongTon = SoLuongTon - i.SoLuong
    FROM SanPham sp
    JOIN inserted i ON sp.MaSP = i.MaSP;
END;
```

- Mục đích: tự động cập nhật tồn kho khi bán hàng

![Ảnh 10](pic_10.png)

---

# PHẦN 5: CURSOR

## Ảnh 11: Cursor duyệt đơn hàng

```sql
DECLARE cur CURSOR FOR
SELECT MaDH FROM DonHang;

DECLARE @MaDH INT;

OPEN cur;
FETCH NEXT FROM cur INTO @MaDH;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT @MaDH;
    FETCH NEXT FROM cur INTO @MaDH;
END;

CLOSE cur;
DEALLOCATE cur;
```

![Ảnh 11](pic_11.png)

---

# NHẬN XÉT

- Cursor xử lý từng dòng → chậm
- SQL thuần (SELECT) nhanh hơn
- Trigger giúp tự động hóa xử lý dữ liệu

---

# KẾT LUẬN

- Đã xây dựng đầy đủ hệ thống quản lý kho gear
- Áp dụng Function, Procedure, Trigger, Cursor
- Hiểu rõ cách tổ chức và xử lý dữ liệu trong SQL Server
