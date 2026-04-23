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
```asm
movq %rdx, %rax
```
*   **Intel:**
```asm
mov rax, rdx
```
*   **Kết quả:** `%rax` chứa địa chỉ $x_E$.

**2. Truy cập phần tử đầu tiên (Biểu thức C: `E[0]`)**
*Đọc giá trị từ bộ nhớ.*

*   **AT&T:**
```asm
movl (%rdx), %eax
```
*   **Intel:**
```asm
mov eax, dword ptr [rdx]
```
*   **Kết quả:** `%eax` chứa giá trị tại $M[x_E]$.

**3. Truy cập phần tử thứ i (Biểu thức C: `E[i]`)**
*Sử dụng hệ số tỷ lệ (Scale factor) là 4.*

*   **AT&T:**
```asm
movl (%rdx, %rcx, 4), %eax
```
*   **Intel:**
```asm
mov eax, dword ptr [rdx + rcx * 4]
```
*   **Kết quả:** `%eax` chứa giá trị tại $M[x_E + 4i]$.

**4. Lấy địa chỉ của phần tử thứ 2 (Biểu thức C: `&E[2]`)**
*Tính toán địa chỉ mà không truy cập bộ nhớ.*

*   **AT&T:**

```asm
leaq 8(%rdx), %rax
```

*   **Intel:**

```asm
lea rax, [rdx + 8]
```

*   **Kết quả:** `%rax` chứa địa chỉ $x_E + 8$.

**5. Toán tử con trỏ phức tạp (Biểu thức C: `E + i - 1`)**
*Thường xuất hiện khi trình biên dịch tối ưu hóa vòng lặp.*

*   **AT&T:**
```asm
leaq -4(%rdx, %rcx, 4), %rax
```

*   **Intel:**
```asm
lea rax, [rdx + rcx * 4 - 4]
```

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

Tiếp tục với mục tiêu hỗ trợ bạn sử dụng IDA Pro, chúng ta sẽ bước sang một phần cực kỳ quan trọng: **Luồng điều khiển**.

Trong IDA, việc hiểu luồng điều khiển không chỉ giúp bạn nắm được logic của chương trình mà còn là "manh mối" duy nhất để bạn xác định xem một biến là **có dấu (signed)** hay **không dấu (unsigned)** — điều mà IDA Decompiler thường xuyên nhận diện sai.

---

# Phần 3: Luồng điều khiển (Control Flow)

Để thay đổi luồng thực thi, CPU sử dụng các **Cờ trạng thái (Condition Codes)** được cập nhật sau mỗi phép toán.

*   **ZF (Zero Flag):** Kết quả phép toán bằng 0.
*   **SF (Sign Flag):** Kết quả phép toán âm (bit cao nhất là 1).
*   **CF (Carry Flag):** Có nhớ ra ngoài bit cao nhất (dành cho số không dấu).
*   **OF (Overflow Flag):** Xảy ra tràn số bù 2 (dành cho số có dấu).

### 1. Phép so sánh và Nhận diện kiểu dữ liệu (Signed vs Unsigned)

Đây là phần quan trọng nhất để **Retype** trong IDA. Trình biên dịch sẽ chọn lệnh nhảy dựa trên kiểu dữ liệu của biến.

**Ví dụ: So sánh hai biến `a` và `b`**

| Cú pháp Intel (IDA) | Ý nghĩa logic | Kiểu dữ liệu tương ứng |
| :--- | :--- | :--- |
| `cmp rax, rdx` | So sánh $a$ và $b$ | (Bước đệm) |
| **`jg`** (Jump Greater) | Nhảy nếu $a > b$ | **Signed** (`int`, `long`) |
| **`ja`** (Jump Above) | Nhảy nếu $a > b$ | **Unsigned** (`unsigned int`, `char *`) |
| **`jl`** (Jump Less) | Nhảy nếu $a < b$ | **Signed** |
| **`jb`** (Jump Below) | Nhảy nếu $a < b$ | **Unsigned** |

**Kinh nghiệm IDA:** Nếu bạn thấy Decompiler hiện `if ( a > b )` nhưng mã Assembly bên dưới dùng lệnh **`ja`**, hãy Retype `a` và `b` sang `unsigned`.

---

### 2. Cấu trúc If-Else (Mẫu cơ bản)

Dưới đây là cách trình biên dịch chuyển đổi một khối `if-else` (Ví dụ từ Mục 3.6.5).

**Mã nguồn C:**
```c
long absdiff(long x, long y) {
    long result;
    if (x < y)
        result = y - x;
    else
        result = x - y;
    return result;
}
```

**Mã Assembly (Cú pháp Intel trong IDA):**
```asm
; x in rdi, y in rsi
absdiff:
    cmp     rdi, rsi        ; So sánh x : y
    jge     short loc_else  ; Nếu x >= y, nhảy tới phần Else
    mov     rax, rsi        ; --- Bắt đầu phần Then ---
    sub     rax, rdi        ; result = y - x
    retn
loc_else:                   ; --- Bắt đầu phần Else ---
    mov     rax, rdi
    sub     rax, rsi        ; result = x - y
    retn
```

---

### 3. Cấu trúc Vòng lặp (Loops)

Trình biên dịch thường chuyển đổi các vòng lặp `while` hoặc `for` về dạng `do-while` để tối ưu số lần nhảy.

**Mã nguồn C (Vòng lặp `while`):**
```c
long fact_while(long n) {
    long result = 1;
    while (n > 1) {
        result *= n;
        n = n - 1;
    }
    return result;
}
```

**Mã Assembly (Cú pháp Intel trong IDA - Dạng Jump-to-Middle):**
```asm
; n in rdi
fact_while:
    mov     eax, 1          ; result = 1
    jmp     short loc_test  ; Nhảy xuống kiểm tra điều kiện trước
loc_loop:                   ; --- Thân vòng lặp ---
    imul    rax, rdi        ; result *= n
    sub     rdi, 1          ; n = n - 1
loc_test:                   ; --- Kiểm tra điều kiện ---
    cmp     rdi, 1          ; So sánh n : 1
    jg      short loc_loop  ; Nếu n > 1, quay lại vòng lặp
    retn
```

---

### 4. Switch Case và Jump Table (Cực kỳ quan trọng cho IDA)

Khi một lệnh `switch` có nhiều trường hợp (thường > 4) và các giá trị gần nhau, trình biên dịch tạo ra một **Jump Table** (Bảng nhảy). Đây là nơi bạn cần tạo Struct hoặc Array địa chỉ trong IDA.

**Mã nguồn C:**
```c
switch (n) {
    case 100: val *= 13; break;
    case 102: val += 10; /* Fall through */
    case 103: val += 11; break;
    default:  val = 0;
}
```

**Mã Assembly (Cú pháp Intel trong IDA):**
```asm
; n in rsi (sau khi đã trừ đi 100 để đưa về dải 0-6)
    cmp     rsi, 6          ; Kiểm tra xem n có nằm trong dải 0-6 không
    ja      loc_default     ; Nếu n > 6, nhảy tới trường hợp default
    jmp     ds:jpt_4006F8[rsi*8] ; Nhảy gián tiếp dựa vào bảng địa chỉ (Jump Table)

; --- Jump Table (Bảng nhảy nằm trong phân đoạn .rodata) ---
jpt_4006F8:
    dq offset loc_case100   ; n = 100 (index 0)
    dq offset loc_default   ; n = 101 (index 1 - missing)
    dq offset loc_case102   ; n = 102 (index 2)
    ...
```

---

### -> RETYPE & LOGIC

Khi phân tích luồng điều khiển trong IDA để phục vụ Retype:

1.  **Xác định Signed/Unsigned**: 
    *   Để ý kỹ lệnh nhảy: 
        *   `jg`, `jl`, `jge`, `jle` -> Biến chắc chắn là **có dấu (signed)**.
        *   `ja`, `jb`, `jae`, `jbe` -> Biến chắc chắn là **không dấu (unsigned)**.
    *   Hãy sửa lại kiểu dữ liệu ngay (`Y`) khi thấy Decompiler hiển thị so sánh sai bản chất.

2.  **Khôi phục Switch Case**:
    *   Nếu Decompiler không nhận diện được `switch`, bạn sẽ thấy lệnh `jmp ds:addr[rsi*8]`.
    *   **Retype**: Click vào `ds:addr`, xác định đây là một mảng các địa chỉ (kiểu `void *` hoặc `code *`).
    *   Việc tính toán `index = n - 100` giúp bạn xác định giá trị khởi đầu của `switch`.

3.  **Nhận diện "Fall-through"**:
    *   Trong ví dụ `case 102`, nếu không có lệnh `jmp` cuối block để thoát ra ngoài, nó sẽ thực thi thẳng vào block tiếp theo. IDA Decompiler thường xử lý tốt cái này, nhưng khi đọc mã ASM, sự thiếu vắng của lệnh nhảy thoát là dấu hiệu của "Fall-through".

---
