# IDA Tricks

---

# Phần 1: Định dạng dữ liệu & Hệ thống thanh ghi (Data Formats & Registers)

### 1. Kích thước dữ liệu và hậu tố lệnh

Trong x86-64, kích thước của kiểu dữ liệu được xác định thông qua hậu tố của lệnh (AT&T) hoặc các từ khóa chỉ định kích thước bộ nhớ (Intel).

*   **Byte**: 1 byte (C: `char`)
*   **Word**: 2 bytes (C: `short`)
*   **Double Word**: 4 bytes (C: `int`, `float`)
*   **Quad Word**: 8 bytes (C: `long`, `double`, `char*`)

### 2. Cấu trúc các thanh ghi số nguyên

Mỗi thanh ghi 64-bit có thể được truy cập dưới dạng các phần nhỏ hơn để tương ứng với các kiểu dữ liệu C khác nhau.

*   **64-bit**: `%rax`, `%rbx`, `%rcx`, `%rdx`, `%rsi`, `%rdi`, `%rbp`, `%rsp`, `%r8`-%r15
*   **32-bit**: `%eax`, `%ebx`, `%ecx`, `%edx`, `%esi`, `%edi`, `%ebp`, `%esp`, `%r8d`-%r15d
*   **16-bit**: `%ax`, `%bx`, `%cx`, `%dx`, `%si`, `%di`, `%bp`, `%sp`, `%r8w`-%r15w
*   **8-bit**: `%al`, `%bl`, `%cl`, `%dl`, `%sil`, `%dil`, `%bpl`, `%spl`, `%r8b`-%r15b

### 3. Ví dụ biên dịch: Hàm di chuyển dữ liệu (Data Movement)

Dưới đây là ví dụ về cách trình biên dịch chuyển đổi từ mã C sang Assembly cho một hàm thao tác với con trỏ và giá trị.

**Mã nguồn C:**
```c
long exchange(long *xp, long y)
{
    long x = *xp;
    *xp = y;
    return x;
}
```

**Mã Assembly (AT&T Syntax - Trong sách):**
```asm
; xp in %rdi, y in %rsi
exchange:
    movq    (%rdi), %rax    ; Get x at xp. Set as return value.
    movq    %rsi, (%rdi)    ; Store y at xp.
    ret                     ; Return.
```

**Mã Assembly (Intel Syntax - Trong IDA):**
```asm
; xp in rdi, y in rsi
exchange:
    mov     rax, [rdi]      ; Get x at xp. Set as return value.
    mov     [rdi], rsi      ; Store y at xp.
    retn                    ; Return.
```

### 4. Một ví dụ khác về việc thay đổi kích thước dữ liệu

Khi di chuyển dữ liệu từ kích thước nhỏ sang lớn, x86-64 sử dụng các lệnh mở rộng (`movz` cho số không dấu và `movs` cho số có dấu).

**Mã Assembly (AT&T Syntax):**
```asm
movabsq $0x0011223344556677, %rax  ; %rax = 0011223344556677
movb    $-1, %al                    ; %rax = 00112233445566FF
movw    $-1, %ax                    ; %rax = 001122334455FFFF
movl    $-1, %eax                   ; %rax = 00000000FFFFFFFF (Xóa bit cao)
movq    $-1, %rax                   ; %rax = FFFFFFFFFFFFFFFF
```

**Mã Assembly (Intel Syntax):**
```asm
movabs  rax, 11223344556677h       ; rax = 0011223344556677
mov     al, 0FFh                    ; rax = 00112233445566FF
mov     ax, 0FFFFh                  ; rax = 001122334455FFFF
mov     eax, 0FFFFFFFFh             ; rax = 00000000FFFFFFFF (Xóa bit cao)
mov     rax, 0FFFFFFFFFFFFFFFFh     ; rax = FFFFFFFFFFFFFFFF
```

---

### TỔNG KẾT VỀ VIỆC RETYPE TRONG IDA

Dựa trên các khối mã trên, đây là những quy tắc bạn cần nhớ khi thực hiện Retype (`phím Y`) trong IDA:

1.  **Xác định kích thước biến (Variable Size)**:
    *   Nếu thấy lệnh sử dụng thanh ghi 8-bit (`al`, `bl`, `dl`, `cl`...) hoặc từ khóa `byte ptr`: Hãy Retype biến đó về `char` (hoặc `_BYTE`).
    *   Nếu thấy thanh ghi 32-bit (`eax`, `edi`, `esi`...) hoặc `dword ptr`: Retype về `int` (hoặc `_DWORD`). Lưu ý lệnh `mov` vào thanh ghi 32-bit sẽ tự động xóa 32 bit cao của thanh ghi 64-bit tương ứng.
    *   Nếu thấy thanh ghi 64-bit (`rax`, `rdi`...) hoặc `qword ptr`: Retype về `long`, `long long` hoặc con trỏ (Pointer).

2.  **Nhận diện con trỏ (Pointer Recognition)**:
    *   Bất cứ khi nào bạn thấy một thanh ghi nằm trong ngoặc vuông `[rdi]`, `[rax]`, `[rbp+var_X]`: Thanh ghi đó đang chứa một địa chỉ.
    *   Trong hàm `exchange`, vì lệnh `mov rax, [rdi]` lấy dữ liệu 8 byte, ta có thể khẳng định `rdi` là một con trỏ kiểu `long *`.

3.  **Nhận diện tính chất dấu (Signedness)**:
    *   Lệnh `movz` (Move with Zero Extend): Thường dùng cho kiểu dữ liệu không dấu (`unsigned`).
    *   Lệnh `movs` (Move with Sign Extend): Thường dùng cho kiểu dữ liệu có dấu (`signed`).

---
*Tôi đã xong Phần 1. Nếu bạn đã nắm chắc cách nhìn thanh ghi để đoán kiểu dữ liệu, hãy ra lệnh để tôi sang **Phần 2: Chế độ địa chỉ (Addressing Modes)** - Phần này cực kỳ quan trọng để bạn nhận diện Mảng (Array) và biến cục bộ trong IDA.*
