# IDA Tricks

---

# Phần 1: Định dạng dữ liệu & Hệ thống thanh ghi (Data Formats & Registers)

### 1. Kích thước dữ liệu và hậu tố lệnh

Trong x86-64, kích thước của kiểu dữ liệu được xác định thông qua hậu tố của lệnh (AT&T) hoặc các từ khóa chỉ định kích thước bộ nhớ (Intel).

*   **Byte**: 1 byte (`char`)
*   **Word**: 2 bytes (`short`)
*   **Double Word**: 4 bytes (`int`, `float`)
*   **Quad Word**: 8 bytes (`long`, `double`, `char*`)

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

### -> RETYPE TRONG IDA

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

# Phần 2: Chế độ địa chỉ & Toán tử con trỏ (Addressing Modes & Pointers)

Trong x86-64, việc truy cập bộ nhớ luôn tuân theo một công thức tổng quát để tính toán "Địa chỉ hiệu dụng" (Effective Address). Công thức này là:
$$Addr = Imm + R[r_b] + R[r_i] \cdot s$$

Trong đó:
*   **$Imm$ (Immediate)**: Một hằng số offset (thường dùng để xác định vị trí biến cục bộ hoặc trường trong struct).
*   **$r_b$ (Base register)**: Thanh ghi cơ sở.
*   **$r_i$ (Index register)**: Thanh ghi chỉ số (thường là biến chạy của vòng lặp).
*   **$s$ (Scale factor)**: Hệ số nhân, bắt buộc phải là **1, 2, 4, hoặc 8** (tương ứng với kích thước các kiểu dữ liệu cơ bản).

### 1. Ví dụ về tính toán địa chỉ và con trỏ

Giả sử ta có một mảng số nguyên `E` kiểu `int` (mỗi phần tử 4 byte). Địa chỉ bắt đầu của mảng nằm trong `%rdx` và chỉ số `i` nằm trong `%rcx`.

**Bảng đối chiếu các biểu thức C và mã máy tương ứng:**

| Biểu thức C | Kiểu dữ liệu | Giá trị địa chỉ / dữ liệu |
| :--- | :--- | :--- |
| `E` | `int *` | $x_E$ (Địa chỉ bắt đầu) |
| `E[0]` | `int` | $M[x_E]$ (Giá trị phần tử đầu) |
| `E[i]` | `int` | $M[x_E + 4i]$ |
| `&E[2]` | `int *` | $x_E + 8$ |
| `E + i - 1` | `int *` | $x_E + 4i - 4$ |

**Mã Assembly tương ứng (Dữ liệu trả về lưu vào thanh ghi A):**

**1. Lấy địa chỉ của mảng (Biểu thức C: `E`)**
*Dùng để gán con trỏ hoặc truyền tham số.*

*   **AT&T:**
```movq %rdx, %rax```
*   **Intel:**
```mov rax, rdx```
*   **Kết quả:** `%rax` chứa địa chỉ $x_E$.

**2. Truy cập phần tử đầu tiên (Biểu thức C: `E[0]`)**
*Đọc giá trị từ bộ nhớ.*

*   **AT&T:**
```movl (%rdx), %eax```
*   **Intel:**
```mov eax, dword ptr [rdx]```
*   **Kết quả:** `%eax` chứa giá trị tại $M[x_E]$.

**3. Truy cập phần tử thứ i (Biểu thức C: `E[i]`)**
*Sử dụng hệ số tỷ lệ (Scale factor) là 4.*

*   **AT&T:**
```movl (%rdx, %rcx, 4), %eax```
*   **Intel:**
```mov eax, dword ptr [rdx + rcx * 4]```
*   **Kết quả:** `%eax` chứa giá trị tại $M[x_E + 4i]$.

**4. Lấy địa chỉ của phần tử thứ 2 (Biểu thức C: `&E[2]`)**
*Tính toán địa chỉ mà không truy cập bộ nhớ.*

*   **AT&T:**
```leaq 8(%rdx), %rax```
*   **Intel:**
```lea rax, [rdx + 8]```
*   **Kết quả:** `%rax` chứa địa chỉ $x_E + 8$.

**5. Toán tử con trỏ phức tạp (Biểu thức C: `E + i - 1`)**
*Thường xuất hiện khi trình biên dịch tối ưu hóa vòng lặp.*

*   **AT&T:**
```leaq -4(%rdx, %rcx, 4), %rax```
*   **Intel:**
```lea rax, [rdx + rcx * 4 - 4]```
*   **Kết quả:** `%rax` chứa địa chỉ $x_E + 4i - 4$.


### 2. Lệnh LEA (Load Effective Address) - "Vũ khí" tính toán

Lệnh `lea` (Intel) hay `leaq` (AT&T) nhìn bề ngoài giống lệnh di chuyển dữ liệu (`mov`), nhưng nó **không truy cập bộ nhớ**. Nó chỉ dùng bộ máy tính toán địa chỉ của CPU để thực hiện các phép tính số học nhanh.

**Ví dụ:** Nếu `%rdx` đang chứa giá trị `x`.

**Cú pháp AT&T:**
```asm
leaq    7(%rdx, %rdx, 4), %rax   ; Tính rax = x + 4*x + 7 = 5x + 7
```

**Cú pháp Intel (Trong IDA):**
```asm
lea     rax, [rdx + rdx * 4 + 7] ; Tính rax = 5x + 7
```

---

### -> RETYPE TRONG IDA

Khi bạn thấy các lệnh truy cập bộ nhớ dạng phức tạp trong IDA, hãy sử dụng các manh mối sau để Retype:

1.  **Nhận diện kích thước phần tử mảng (Scale factor)**:
    *   Nếu thấy `[base + index * 1]`: Thường là mảng **`char`** (1 byte).
    *   Nếu thấy `[base + index * 2]`: Thường là mảng **`short`** (2 byte).
    *   Nếu thấy `[base + index * 4]`: Thường là mảng **`int`** hoặc **`float`** (4 byte).
    *   Nếu thấy `[base + index * 8]`: Thường là mảng **`long`**, **`double`**, hoặc **mảng các con trỏ** (8 byte).

2.  **Phân biệt `lea` và `mov` trong Intel Syntax**:
    *   `mov eax, dword ptr [rdx + rcx * 4]`: Đây là thao tác lấy **giá trị** từ mảng. Trong C sẽ là `E[i]`.
    *   `lea rax, [rdx + rcx * 4]`: Đây là thao tác lấy **địa chỉ** của phần tử. Trong C sẽ là `&E[i]` hoặc dùng trong toán tử con trỏ `E + i`.
    *   Nếu bạn thấy IDA Decompiler dịch ra các phép tính lạ như `v5 = 5 * v2 + 7`, hãy kiểm tra lại cửa sổ Assembly xem có phải là lệnh `lea rax, [rdx + rdx * 4 + 7]` không. Nếu có, đó có thể chỉ là một phép tính số học bình thường, không phải địa chỉ bộ nhớ.

3.  **Xác định kiểu dữ liệu qua hằng số Offset (Imm)**:
    *   Khi thấy `mov eax, [rdi + 12]`: Nếu `rdi` là một con trỏ tới Struct, thì `12` chính là offset của một trường (field) bên trong Struct đó. Bạn có thể dựa vào kích thước 4 byte (`eax`) để Retype trường tại offset 12 là `int`.

---
