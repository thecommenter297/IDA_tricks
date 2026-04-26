# Machine Level Represntation

---

# 3.2 Program Encodings

### Lệnh biên dịch chuẩn
```bash
linux> gcc -Og -o p p1.c p2.c
```
*   **`-Og`**: Mức tối ưu hóa phục vụ việc học. Giữ nguyên cấu trúc mã nguồn C khi dịch sang mã máy.
*   **`-O1`, `-O2`**: Tối ưu hóa thực tế. Mã máy bị biến đổi mạnh, rất khó đối chiếu với mã nguồn C ban đầu.
*   **Lưu ý**: Nếu dùng bản GCC cũ (< 4.8), thay `-Og` bằng `-O1`.

### Chuỗi tiến trình biên dịch (Compilation Sequence)
1.  **C Preprocessor**: Xử lý `#include` và `#define` (Mở rộng mã).
2.  **Compiler**: Dịch C sang **Assembly** (`.s`).
3.  **Assembler**: Dịch Assembly sang **Object-code** (`.o`).
    *   Là mã máy nhị phân.
    *   *Lưu ý:* Các địa chỉ của biến toàn cục (global values) vẫn chưa được xác định.
4.  **Linker**: Hợp nhất các file `.o` và các hàm thư viện (ví dụ `printf`) thành **Mã thực thi** (`p`).

---

## 3.2.1 Machine-Level Code

Lập trình mức máy dựa trên 2 sự trừu tượng hóa quan trọng:

#### 1. Instruction Set Architecture (ISA)
*   **Định nghĩa**: Quy định trạng thái bộ xử lý (processor state), định dạng các lệnh, và tác động của chúng lên hệ thống.
*   **Nguyên tắc**: Mô tả việc thực thi là **tuần tự** (xong lệnh này mới tới lệnh kia).
*   **Thực tế phần cứng**: Chạy song song nhiều lệnh cùng lúc nhưng có cơ chế bảo vệ để kết quả cuối cùng luôn khớp với logic tuần tự của ISA.

#### 2. Virtual addresses (Địa chỉ ảo)
*   Chương trình mức máy không nhìn thấy địa chỉ vật lý của RAM.
*   Nó sử dụng địa chỉ ảo để quản lý bộ nhớ như một mảng byte khổng lồ... 
*   Là sự kết hợp phức tạp giữa bộ nhớ phần cứng (RAM) và phần mềm hệ điều hành (OS).

#### Vai trò của Trình biên dịch (Compiler)
*   Chuyển đổi các mô hình thực thi trừu tượng của C thành các **chỉ thị cơ bản (elementary instructions)** mà bộ xử lý có thể hiểu được.
*   **Assembly**: Là định dạng văn bản (textual format) của mã máy, giúp con người có thể đọc được. Đây là bước đệm then chốt để hiểu cách máy tính vận hành.

---

#### Trạng thái Bộ xử lý x86-64 (Processor State)
Khác với C, mã máy x86-64 làm lộ diện các thành phần phần cứng mà lập trình viên C thường không thấy:

| Thành phần | Tên gọi/Đặc điểm | Chức năng trong IDA Pro & Assembly |
| :--- | :--- | :--- |
| **Program Counter (PC)** | `%rip` (x86-64) | Lưu địa chỉ của **lệnh tiếp theo** sẽ được thực thi. |
| **Integer Register File** | Gồm 16 thanh ghi (64-bit) | Lưu địa chỉ (con trỏ C) hoặc dữ liệu số nguyên. Dùng để truyền tham số, lưu biến cục bộ và giá trị trả về. |
| **Condition Code Registers** | Các flag trạng thái | Lưu thông tin về kết quả của lệnh số học/logic gần nhất. Dùng để thực hiện rẽ nhánh (`if`, `while`). |
| **Vector Registers** | Các thanh ghi bổ sung | Lưu một hoặc nhiều giá trị số nguyên hoặc số thực dấu phẩy động (floating-point). |

---

#### Cách nhìn nhận Bộ nhớ: C vs. Mã máy

1.  **Trong C:** Dữ liệu có kiểu (Types) rõ ràng: `int`, `float`, `struct`, `array`... được cấp phát vùng nhớ riêng biệt.
2.  **Trong Mã máy:** 
    *   Bộ nhớ chỉ là một **mảng byte có thể đánh địa chỉ (byte-addressable array)**.
    *   Dữ liệu phức hợp (Array, Struct) chỉ là các khối byte liền kề (contiguous bytes).
    *   **Không phân biệt** giữa số nguyên có dấu/không dấu, không phân biệt các loại con trỏ, thậm chí không phân biệt con trỏ với số nguyên.

---

#### Quản lý Bộ nhớ Chương trình (Program Memory)
Bộ nhớ ảo chứa các thành phần:
*   **Mã thực thi (Executable code)** của chương trình.
*   **Thông tin hệ điều hành** cần thiết.
*   **Run-time Stack**: Quản lý việc gọi hàm và trả về (procedure calls/returns).
*   **Heap**: Các khối bộ nhớ do người dùng cấp phát (ví dụ: `malloc`).

#### Giới hạn địa chỉ (Addressing)
*   Mặc dù địa chỉ là 64-bit, nhưng thực tế hiện nay x86-64 chỉ sử dụng **48 bit** (16 bit cao phải luôn bằng 0).
*   **Phạm vi**: Có thể đánh địa chỉ tới $2^{48}$ byte (64 Terabytes).
*   **Thực tế**: Các chương trình thông thường chỉ truy cập vài Megabytes hoặc Gigabytes. Hệ điều hành sẽ quản lý để ánh xạ địa chỉ ảo này vào bộ nhớ vật lý thật.

---

## 3.2.2 Code Examples (Các ví dụ về mã)

### Lưu ý về sự biến đổi của mã (Aside)
*   Mã máy được tạo ra bởi GCC sẽ thay đổi tùy thuộc vào phiên bản trình biên dịch và các tùy chọn dòng lệnh (optimization levels).
*   **Mục tiêu học tập:** Không phải là học thuộc lòng một đoạn mã cụ thể, mà là học cách xem xét mã assembly và ánh xạ (map) nó ngược lại các cấu trúc trong ngôn ngữ lập trình bậc cao (C).

---

### Ví dụ mã nguồn C (`mstore.c`)
Đoạn mã dưới đây định nghĩa hàm `multstore` thực hiện nhân hai số thông qua một hàm bổ trợ và lưu kết quả vào một con trỏ.

```c
long mult2(long, long);

void multstore(long x, long y, long *dest) {
    long t = mult2(x, y);
    *dest = t;
}
```

### Lệnh tạo file Assembly
Để xem mã assembly được tạo ra bởi trình biên dịch C, sử dụng cờ `-S`:

```bash
linux> gcc -Og -S mstore.c
```
*   **Kết quả:** GCC sẽ chạy trình biên dịch và dừng lại sau khi tạo ra file assembly `mstore.s`, không tiến hành dịch sang file object (`.o`).

---

### Phân tích mã Assembly (Bắt đầu)

File `mstore.s` chứa các khai báo và các dòng lệnh thực thi. Dưới đây là những dòng đầu tiên của hàm `multstore`:



<details>
   <summary>Cách hiển thị biểu diễn Byte của chương trình (Aside)</summary>
   
   ---
   
Để xem mã đối tượng nhị phân (binary object code), ta sử dụng công cụ **disassembler**. Ví dụ với hàm `multstore` dài 14 bytes, ta có thể dùng trình gỡ lỗi GDB với lệnh:
`x/14xb multstore`
*   **`x`**: Examine (kiểm tra bộ nhớ).
*   **`14`**: Số lượng đơn vị cần xem.
*   **`x`**: Định dạng hiển thị Hexadecimal.
*   **`b`**: Đơn vị là Byte.
</details>

---

### Phân tích mã Assembly (Tiếp theo)

Dưới đây là các dòng lệnh còn lại của hàm `multstore`:

| AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích ý nghĩa |
| :--- | :--- | :--- |
| `movq %rdx, %rbx` | `mov rbx, rdx` | Sao chép giá trị con trỏ `dest` (đang ở `%rdx`) sang `%rbx`. |
| `call mult2` | `call mult2` | Gọi hàm `mult2`. Kết quả trả về sẽ được lưu vào `%rax`. |
| `movq %rax, (%rbx)` | `mov [rbx], rax` | **Dereference**: Lưu kết quả từ `%rax` vào địa chỉ bộ nhớ mà `%rbx` đang trỏ tới (`*dest = t`). |
| `popq %rbx` | `pop rbx` | Khôi phục lại giá trị cũ của thanh ghi `%rbx` từ Stack. |
| `ret` | `retn` | Trở về hàm đã gọi (caller). |

**Đặc điểm quan trọng:**
*   Mỗi dòng lệnh tương ứng với một chỉ thị máy đơn nhất.
*   Tất cả thông tin về tên biến cục bộ hoặc kiểu dữ liệu đã bị loại bỏ hoàn toàn ở mức này.

---

### Mã đối tượng (Object Code)

Để biên dịch và dịch chuyển (assemble) mã mà không liên kết, ta dùng cờ `-c`:
```bash
linux> gcc -Og -c mstore.c
```
Thao tác này tạo ra file `mstore.o` chứa mã máy ở định dạng nhị phân. Chuỗi 14 bytes thực tế của hàm `multstore` trông như sau (hệ thập lục phân):

`53 48 89 d3 e8 00 00 00 00 48 89 03 5b c3`

**Bài học then chốt:**
Chương trình thực thi bởi máy tính chỉ đơn giản là một chuỗi các byte mã hóa một loạt các chỉ thị. Máy tính có rất ít thông tin về mã nguồn gốc đã tạo ra các chỉ thị này.

---

### Trình nghịch đảo mã (Disassemblers)

Để kiểm tra nội dung của các file mã máy, các chương trình thuộc lớp **disassemblers** là cực kỳ vô giá. Chúng tạo ra một định dạng tương tự mã assembly từ mã máy nhị phân.
Trên hệ thống Linux, công cụ `OBJDUMP` (viết tắt của "object dump") với cờ `-d` thực hiện vai trò này:

```bash
linux> objdump -d mstore.o
```

Kết quả được trình bày như sau:

---

### Kết quả nghịch đảo mã (Disassembly) của hàm `multstore`

Dưới đây là bảng phân tích từ file mã đối tượng `mstore.o` thông qua lệnh `objdump -d`:

| Offset | Bytes (Mã máy) | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| `0:` | `53` | `pushq %rbx` | `push rbx` | Lưu `%rbx` vào Stack. |
| `1:` | `48 89 d3` | `movq %rdx, %rbx` | `mov rbx, rdx` | Sao chép `dest` vào `%rbx`. |
| `4:` | `e8 00 00 00 00` | `callq 9 <multstore+0x9>` | `call mult2` | Gọi hàm (Địa chỉ tạm thời là 0). |
| `9:` | `48 89 03` | `movq %rax, (%rbx)` | `mov [rbx], rax` | Lưu kết quả vào `*dest`. |
| `c:` | `5b` | `popq %rbx` | `pop rbx` | Khôi phục `%rbx`. |
| `d:` | `c3` | `retq` | `retn` | Trở về hàm gọi. |

---

### Các đặc điểm quan trọng của mã máy và bản nghịch đảo mã:

1.  **Độ dài lệnh:** Các lệnh x86-64 có độ dài thay đổi từ **1 đến 15 bytes**. Những lệnh thường dùng hoặc có ít toán hạng sẽ tốn ít byte hơn.
2.  **Thiết kế định dạng lệnh:** Từ một vị trí bắt đầu xác định, mỗi chuỗi byte chỉ có **duy nhất một cách giải mã** thành lệnh assembly. Ví dụ: chỉ có lệnh `pushq %rbx` mới bắt đầu bằng byte `53`.
3.  **Cơ chế của Disassembler:** Trình nghịch đảo mã xác định các lệnh assembly thuần túy dựa trên chuỗi byte trong file mã máy. Nó không cần truy cập vào mã nguồn C hay các khai báo assembly gốc.
4.  **Quy ước đặt tên (Suffixes):** Trình nghịch đảo mã (`objdump`) sử dụng quy ước đặt tên hơi khác so với GCC:
    *   Nó lược bỏ hậu tố `q` (quadword) ở nhiều lệnh (ví dụ: `push` thay vì `pushq`). Các hậu tố này chỉ kích thước và thường có thể lược bỏ.
    *   Ngược lại, nó thêm hậu tố `q` vào lệnh `callq` và `retq`. Trong IDA Pro, bạn thường sẽ thấy `call` và `retn`.

---

### Ví dụ về chương trình hoàn chỉnh

Để tạo ra mã thực thi thực tế, ta cần chạy trình liên kết (linker) trên tập hợp các file mã đối tượng. File này phải chứa một hàm `main`.

**Mã nguồn file `main.c`:**
```c
#include <stdio.h>

void multstore(long, long, long *);

int main() {
    long d;
    multstore(2, 3, &d);
    printf("2 * 3 --> %ld\n", d);
    return 0;
}
```

**Mã nguồn hàm `mult2` (trong một file khác):**
```c
long mult2(long a, long b) {
    long s = a * b;
    return s;
}
```

Để tạo ra chương trình thực thi `prog` từ hai file nguồn:
```bash
linux> gcc -Og -o prog main.c mstore.c
```
*   **Kích thước file:** File `prog` tăng lên khoảng 8,655 bytes vì nó không chỉ chứa mã máy của các hàm ta viết, mà còn bao gồm các đoạn mã để khởi động, kết thúc chương trình và tương tác với hệ điều hành.

Để xem mã máy của file thực thi này, ta sử dụng:
```bash
linux> objdump -d prog
```

---

### Phân tích hàm `multstore` trong file thực thi `prog`

Trình nghịch đảo mã sẽ trích xuất đoạn mã sau (địa chỉ đã được điền đầy đủ):

| Địa chỉ (Address) | Bytes (Mã máy) | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| `400540:` | `53` | `push %rbx` | `push rbx` | Lưu `%rbx` |
| `400541:` | `48 89 d3` | `mov %rdx, %rbx` | `mov rbx, rdx` | `dest` -> `%rbx` |
| `400544:` | `e8 42 00 00 00` | `callq 40058b <mult2>` | `call mult2` | Gọi hàm `mult2` tại `40058b` |
| `400549:` | `48 89 03` | `mov %rax, (%rbx)` | `mov [rbx], rax` | `*dest = t` |
| `40054c:` | `5b` | `pop %rbx` | `pop rbx` | Khôi phục `%rbx` |
| `40054d:` | `c3` | `retq` | `retn` | Trở về |
| `40054e:` | `90` | `nop` | `nop` | Lệnh không hoạt động (Padding) |
| `40054f:` | `90` | `nop` | `nop` | Lệnh không hoạt động (Padding) |

---

### Sự khác biệt sau khi Liên kết (Linking)

Mã này gần như giống hệt với mã được tạo ra từ file `mstore.o`, nhưng có 3 điểm khác biệt quan trọng cần lưu ý:

1.  **Thay đổi địa chỉ (Address shifting):** Trình liên kết đã di chuyển mã của hàm này sang một dải địa chỉ khác (`400540`).
2.  **Điền địa chỉ gọi hàm (Relocation):** Trong file `.o`, lệnh `call` có địa chỉ tạm thời là `00 00 00 00`. Sau khi liên kết, trình liên kết đã điền địa chỉ thực tế của hàm `mult2` (ở đây là khoảng cách tương đối `0x42` dẫn đến địa chỉ `40058b`).
3.  **Lệnh đệm (Padding - NOP):** Bạn thấy xuất hiện hai lệnh `nop` (No-Operation) ở cuối hàm (dòng 8 và 9). 
    *   **Mục đích:** Kéo dài kích thước hàm lên đúng 16 bytes. 
    *   **Lợi ích:** Giúp việc sắp xếp khối mã tiếp theo trong bộ nhớ đạt hiệu suất tối ưu hơn cho hệ thống.

---

## 3.2.3 Notes on Formatting (Ghi chú về định dạng)

Mã assembly do GCC tạo ra thường khó đọc đối với con người. Một mặt, nó chứa nhiều thông tin bổ trợ không quá quan trọng; mặt khác, nó không cung cấp mô tả về logic chương trình.

Để xem toàn bộ nội dung file assembly được tạo ra từ lệnh:
```bash
linux> gcc -Og -S mstore.c
```
Nội dung đầy đủ của file `mstore.s` (sau khi lược bỏ các chỉ thị rác) sẽ được trình bày như sau:

```assembly
    .file   "010-mstore.c"
    .text
    .globl  multstore
    .type   multstore, @function
multstore:
    pushq   %rbx
    movq    %rdx, %rbx
    call    mult2
    movq    %rax, (%rbx)
    popq    %rbx
    ret
    .size   multstore, .-multstore
    .ident  "GCC: (Ubuntu 4.8.1-2ubuntu1-12.04) 4.8.1"
    .section    .note.GNU-stack,"",@progbits
```

**Giải thích về các chỉ thị (Directives):**
*   Tất cả các dòng bắt đầu bằng dấu **`.`** là các chỉ thị nhằm hướng dẫn trình dịch chuyển (assembler) và trình liên kết (linker). 
*   **Về cơ bản:** Chúng ta có thể bỏ qua các dòng này khi phân tích logic chương trình. Chúng không tạo ra các lệnh máy thực thi trực tiếp.

---

### Phiên bản Assembly rút gọn và Chú giải (Annotated Version)

Để giúp việc phân tích dễ dàng hơn, sách sẽ lược bỏ các chỉ thị rác và trình bày mã dưới dạng có chú thích. Đây cũng là cách bạn nên nhìn nhận khi soi mã trong IDA Pro.

**Ánh xạ tham số vào thanh ghi (Register Mapping):**
*   `x` nằm trong thanh ghi `%rdi`
*   `y` nằm trong thanh ghi `%rsi`
*   `dest` nằm trong thanh ghi `%rdx`

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Chú thích (Ý nghĩa logic) |
| :--- | :--- | :--- | :--- |
| 1 | `multstore:` | `multstore:` | Nhãn bắt đầu hàm. |
| 2 | `pushq %rbx` | `push rbx` | Lưu giá trị `%rbx` (Save %rbx). |
| 3 | `movq %rdx, %rbx` | `mov rbx, rdx` | Copy giá trị `dest` sang `%rbx`. |
| 4 | `call mult2` | `call mult2` | Gọi hàm `mult2(x, y)`. |
| 5 | `movq %rax, (%rbx)` | `mov [rbx], rax` | Lưu kết quả (`t`) vào địa chỉ `*dest`. |
| 6 | `popq %rbx` | `pop rbx` | Khôi phục giá trị `%rbx` (Restore %rbx). |
| 7 | `ret` | `retn` | Trở về (Return). |

**Lưu ý về cách trình bày:**
*   Các dòng mã được đánh số bên trái để tiện tham chiếu.
*   Phần chú giải bên phải mô tả ngắn gọn tác động của lệnh và mối liên hệ của nó với mã nguồn C.

---

### Các tài liệu bổ sung (Web Asides)

Sách cũng cung cấp các tài liệu mở rộng trên web cho những người đam mê chuyên sâu:
*   **IA32 Machine Code:** Kiến thức nền tảng về x86-64 giúp việc học kiến thức cũ (IA32) trở nên khá đơn giản.
*   **Kết hợp C và Assembly:** Giới thiệu các cách để nhúng mã assembly vào chương trình C. Đối với một số ứng dụng đặc thù, lập trình viên cần xuống mức thấp để tận dụng các tính năng phần cứng mà ngôn ngữ bậc cao không hỗ trợ. Một cách tiếp cận phổ biến là viết toàn bộ hàm bằng assembly và kết hợp chúng với các hàm C trong giai đoạn liên kết.

---

### So sánh cú pháp Assembly: ATT vs. Intel (Aside)

Sách (CS:APP) sử dụng định dạng **ATT** (đặt theo tên công ty AT&T). Tuy nhiên, các công cụ khác như **IDA Pro**, Microsoft Disassembler và tài liệu của Intel lại sử dụng định dạng **Intel**.

Để tạo mã máy cú pháp Intel bằng GCC, ta dùng lệnh:
```bash
linux> gcc -Og -S -masm=intel mstore.c
```

**Mã Assembly của hàm `multstore` (Cú pháp Intel/IDA Pro):**
```assembly
multstore:
    push    rbx
    mov     rbx, rdx
    call    mult2
    mov     QWORD PTR [rbx], rax
    pop     rbx
    ret
```

**Các điểm khác biệt then chốt (Cần nhớ để dùng IDA Pro):**

1.  **Hậu tố kích thước (Size suffixes):** Intel lược bỏ các hậu tố như `q` trong `pushq` hay `movq`. Ta chỉ thấy `push` và `mov`.
2.  **Ký hiệu thanh ghi:** Intel lược bỏ dấu `%` trước tên thanh ghi. Ví dụ: dùng `rbx` thay vì `%rbx`.
3.  **Biểu diễn bộ nhớ:** Intel sử dụng cách mô tả địa chỉ khác hẳn.
    *   **ATT:** `(%rbx)`
    *   **Intel:** `QWORD PTR [rbx]` (Sử dụng từ khóa như `QWORD PTR` và dấu ngoặc vuông `[]`).
4.  **Thứ tự toán hạng (Cực kỳ quan trọng):** 
    *   **ATT:** `Lệnh  Nguồn, Đích` (Ví dụ: `movq %rdx, %rbx` -> Copy từ rdx sang rbx).
    *   **Intel (IDA):** `Lệnh  Đích, Nguồn` (Ví dụ: `mov rbx, rdx` -> Copy từ rdx vào rbx).
    *   *Lưu ý:* Việc đảo ngược thứ tự này thường gây nhầm lẫn nhất khi bạn chuyển đổi giữa đọc sách và soi mã trên IDA.

---

## 3.3 Data Formats (Định dạng dữ liệu)

Do bắt nguồn từ kiến trúc 16-bit rồi mở rộng lên 32-bit và 64-bit, Intel sử dụng thuật ngữ **"word"** (từ) theo cách riêng:

*   **Word:** Đại diện cho kiểu dữ liệu **16-bit**.
*   **Double words:** Đại diện cho kiểu dữ liệu **32-bit** (kí hiệu là `l` trong ATT, viết tắt của "long word").
*   **Quad words:** Đại diện cho kiểu dữ liệu **64-bit** (kí hiệu là `q` trong ATT). Đây là kích thước mặc định cho các con trỏ và số nguyên `long` trong x86-64.

**Bảng tóm tắt các kiểu dữ liệu nguyên trong C trên x86-64:**

| Kiểu dữ liệu C | Cú pháp Intel (IDA) | Cú pháp ATT (Sách) | Kích thước (Bytes) |
| :--- | :--- | :--- | :--- |
| `char` | `Byte` | `b` | 1 |
| `short` | `Word` | `w` | 2 |
| `int` | `Double word` | `l` | 4 |
| `long` | `Quad word` | `q` | 8 |
| `char *` (Pointer) | `Quad word` | `q` | 8 |
| `float` | `Single precision` | `s` | 4 |
| `double` | `Double precision` | `l` | 8 |

*   **Lưu ý:** Trong kiến trúc x86-64, kiểu `long` được triển khai với 64 bit, cho phép dải giá trị rất rộng. Hầu hết các ví dụ trong chương này sẽ tập trung vào con trỏ và dữ liệu `long` (8 bytes).

---

### Kết hợp mã Assembly với chương trình C (Aside)

Mặc dù trình biên dịch C thực hiện rất tốt việc chuyển đổi logic sang mã máy, nhưng có một số tính năng của phần cứng mà C không thể truy cập trực tiếp.
*   **Ví dụ:** Flag trạng thái **PF (Parity Flag)**. Mỗi khi CPU thực hiện lệnh số học, nó đặt PF = 1 nếu 8 bit thấp của kết quả có số lượng bit 1 là số chẵn. Trong C, việc kiểm tra điều này tốn rất nhiều bước tính toán (dịch bit, masking), trong khi mã máy có thể đọc trực tiếp từ thanh ghi flag.
*   **Hai cách nhúng mã máy:**
    1.  Viết toàn bộ hàm trong một file `.s` riêng biệt rồi để trình liên kết (linker) kết hợp lại.
    2.  Sử dụng **inline assembly** của GCC bằng chỉ thị `asm`. Cách này giúp chèn các đoạn mã máy ngắn trực tiếp vào file `.c`.

---

### Hình 3.1: Kích thước các kiểu dữ liệu C trong x86-64

Đây là bảng ánh xạ quan trọng nhất để bạn nhận diện dữ liệu khi dùng IDA Pro. Tập lệnh x86-64 cung cấp đầy đủ các chỉ thị cho từng kích thước dữ liệu này.

| C declaration | Intel data type | Assembly suffix (ATT) | Size (bytes) |
| :--- | :--- | :--- | :--- |
| `char` | **Byte** | `b` | 1 |
| `short` | **Word** | `w` | 2 |
| `int` | **Double word** | `l` | 4 |
| `long` | **Quad word** | `q` | 8 |
| `char *` (Pointer) | **Quad word** | `q` | 8 |
| `float` | **Single precision** | `s` | 4 |
| `double` | **Double precision** | `l` | 8 |

**Lưu ý kỹ thuật về hậu tố (Suffixes):**
*   Hậu tố `l` được dùng cho cả số nguyên 4-byte (double word) và số thực 8-byte (double precision). Điều này không gây nhầm lẫn vì các lệnh xử lý số thực và số nguyên là hoàn toàn khác nhau.
*   Các con trỏ (ví dụ `char *`) luôn được coi là số nguyên 8-byte (quad words) trong kiến trúc 64-bit.

---

### Số thực dấu phẩy động (Floating-point)

Số thực trong x86-64 có hai định dạng chính tương ứng với C:
1.  **`float` (Single-precision):** 4 bytes.
2.  **`double` (Double-precision):** 8 bytes.

**Về `long double` (Số thực mở rộng):**
*   Dòng họ x86 có lịch sử hỗ trợ một định dạng số thực đặc biệt 80-bit (10 bytes).
*   Trong C, bạn có thể khai báo bằng `long double` (80-bit). Tuy nhiên, sách khuyến cáo **không nên sử dụng** định dạng này vì nó không có tính di động (portable) sang các dòng máy khác, và nó thường không được thực hiện bởi cùng một phần cứng hiệu suất cao như các loại đơn (single) và đôi (double).
*   **Hậu tố lệnh trong ATT:** Hầu hết các lệnh assembly có một hậu tố ký tự đơn để chỉ định kích thước của toán hạng.
    *   **`b` (byte):** 1 byte (ví dụ: `movb`).
    *   **`w` (word):** 2 bytes (ví dụ: `movw`).
    *   **`l` (double word):** 4 bytes (ví dụ: `movl`). Lưu ý: `l` còn dùng cho số thực 8 bytes.
    *   **`q` (quad word):** 8 bytes (ví dụ: `movq`).
    *   *Ghi chú:* Trong IDA Pro (Intel), kích thước này thường được xác định qua tên thanh ghi (ví dụ: `eax` là 4 bytes, `rax` là 8 bytes) hoặc từ khóa như `BYTE PTR`, `DWORD PTR`.

---

## 3.4 Accessing Information (Truy cập thông tin)

Một bộ xử lý x86-64 chứa một tập hợp gồm **16 thanh ghi đa năng (general-purpose registers)** lưu trữ các giá trị 64-bit. Các thanh ghi này dùng để lưu trữ cả dữ liệu số nguyên lẫn con trỏ.

### Sự tiến hóa của các thanh ghi
Tên gọi của các thanh ghi phản ánh lịch sử phát triển của kiến trúc Intel:
1.  **8086 (16-bit):** 8 thanh ghi gốc 16-bit: `%ax` đến `%bp`.
2.  **IA32 (32-bit):** Các thanh ghi được mở rộng thành 32-bit, ký hiệu là `%eax` đến `%ebp`.
3.  **x86-64 (64-bit):** 
    *   Các thanh ghi mở rộng thành 64-bit, ký hiệu là `%rax` đến `%rbp`.
    *   Bổ sung thêm 8 thanh ghi mới: `%r8` đến `%r15`.

### Truy cập các phần của thanh ghi (Nested Boxes)
Các lệnh có thể vận hành trên các kích thước dữ liệu khác nhau nằm trong các byte bậc thấp (low-order bytes) của thanh ghi:
*   **1 byte:** Truy cập 8 bit thấp nhất.
*   **2 bytes:** Truy cập 16 bit thấp nhất.
*   **4 bytes:** Truy cập 32 bit thấp nhất.
*   **8 bytes:** Truy cập toàn bộ thanh ghi.

---

### Quy tắc quan trọng về thay đổi dữ liệu (Cực kỳ quan trọng khi dùng IDA)

Khi một lệnh có đích đến là một thanh ghi, có hai quy ước xảy ra cho các byte còn lại:
1.  **Ghi 1 hoặc 2 bytes:** Các byte còn lại trong thanh ghi **không thay đổi**.
2.  **Ghi 4 bytes (Double-word):** Các byte bậc cao (4 bytes phía trên) của thanh ghi sẽ **bị đặt về 0 (zero-extended)**. 
    *   *Ví dụ:* Nếu bạn thực hiện `mov eax, 1`, thì toàn bộ 32 bit cao của `rax` sẽ tự động bị xóa về 0. Đây là quy ước được áp dụng khi mở rộng từ IA32 lên x86-64.

| Thao tác (ATT) | Thao tác (Intel/IDA) | Tác động lên thanh ghi 64-bit |
| :--- | :--- | :--- |
| `movb $1, %al` | `mov al, 1` | Chỉ thay đổi 8 bit thấp, các bit khác giữ nguyên. |
| `movw $1, %ax` | `mov ax, 1` | Chỉ thay đổi 16 bit thấp, các bit khác giữ nguyên. |
| `movl $1, %eax` | `mov eax, 1` | **Xóa 32 bit cao về 0**, nạp 1 vào 32 bit thấp. |
| `movq $1, %rax` | `mov rax, 1` | Thay đổi toàn bộ 64 bit. |

---

### Vai trò của các thanh ghi
Mặc dù được gọi là "đa năng", nhưng theo quy ước lập trình chuẩn (ABI), các thanh ghi đóng các vai trò khác nhau:
*   **`%rsp` (Stack Pointer):** Thanh ghi đặc biệt nhất, dùng để trỏ đến vị trí cuối cùng trong ngăn xếp (run-time stack). IDA Pro và các chương trình thường đọc/ghi thanh ghi này một cách thận trọng.
*   **15 thanh ghi còn lại:** Có tính linh hoạt cao hơn, nhưng việc sử dụng chúng phải tuân theo các quy ước để quản lý ngăn xếp, truyền tham số hàm, trả về giá trị hàm...

---

### Hình 3.2: Các thanh ghi số nguyên (Integer Registers)

Các phần bậc thấp của tất cả 16 thanh ghi có thể được truy cập dưới dạng các đại lượng: **8-bit (byte)**, **16-bit (word)**, **32-bit (double word)**, và **64-bit (quad word)**.

<img width="658" height="749" alt="image" src="https://github.com/user-attachments/assets/6989a3b6-5841-40aa-9437-e2002ccd4bee" />


| 64-bit (Sách/ATT) | 32-bit | 16-bit | 8-bit | IDA Pro (Intel) | Vai trò theo quy ước (Role) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `%rax` | `%eax` | `%ax` | `%al` | `rax/eax...` | **Return value** (Giá trị trả về) |
| `%rbx` | `%ebx` | `%bx` | `%bl` | `rbx/ebx...` | **Callee saved** (Hàm bị gọi phải lưu) |
| `%rcx` | `%ecx` | `%cx` | `%cl` | `rcx/ecx...` | **4th argument** (Tham số thứ 4) |
| `%rdx` | `%edx` | `%dx` | `%dl` | `rdx/edx...` | **3rd argument** (Tham số thứ 3) |
| `%rsi` | `%esi` | `%si` | `%sil` | `rsi/esi...` | **2nd argument** (Tham số thứ 2) |
| `%rdi` | `%edi` | `%di` | `%dil` | `rdi/edi...` | **1st argument** (Tham số thứ 1) |
| `%rbp` | `%ebp` | `%bp` | `%bpl` | `rbp/ebp...` | **Callee saved** (Thường dùng làm Frame Pointer) |
| `%rsp` | `%esp` | `%sp` | `%spl` | `rsp/esp...` | **Stack pointer** (Con trỏ ngăn xếp) |
| `%r8` | `%r8d` | `%r8w` | `%r8b` | `r8/r8d...` | **5th argument** (Tham số thứ 5) |
| `%r9` | `%r9d` | `%r9w` | `%r9b` | `r9/r9d...` | **6th argument** (Tham số thứ 6) |
| `%r10` | `%r10d` | `%r10w` | `%r10b` | `r10/r10d...` | **Caller saved** (Hàm gọi phải tự lưu) |
| `%r11` | `%r11d` | `%r11w` | `%r11b` | `r11/r11d...` | **Caller saved** (Hàm gọi phải tự lưu) |
| `%r12` | `%r12d` | `%r12w` | `%r12b` | `r12/r12d...` | **Callee saved** |
| `%r13` | `%r13d` | `%r13w` | `%r13b` | `r13/r13d...` | **Callee saved** |
| `%r14` | `%r14d` | `%r14w` | `%r14b` | `r14/r14d...` | **Callee saved** |
| `%r15` | `%r15d` | `%r15w` | `%r15b` | `r15/r15d...` | **Callee saved** |

---

### Các quy ước sử dụng thanh ghi (Register Conventions)

Như sơ đồ trên cho thấy, các thanh ghi khác nhau phục vụ các vai trò khác nhau trong các chương trình điển hình:

*   **Truyền tham số:** Các thanh ghi `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` được dùng để truyền 6 tham số đầu tiên của một hàm. Trong IDA Pro, khi bạn thấy dữ liệu được nạp vào `%rdi` ngay trước một lệnh `call`, đó chính là tham số thứ nhất của hàm đó.
*   **Giá trị trả về:** Thanh ghi `%rax` luôn chứa kết quả mà một hàm trả về cho hàm gọi nó.
*   **Quản lý Stack:** `%rsp` là con trỏ đặc biệt dùng để quản lý ranh giới hiện tại của ngăn xếp.
*   **Dữ liệu tạm thời & Biến cục bộ:** Các thanh ghi còn lại được dùng để lưu trữ dữ liệu tạm thời. 

**Quy tắc lưu trữ (Saved Registers):**
*   **Caller saved (Hàm gọi lưu):** Hàm gọi phải tự cất dữ liệu trong các thanh ghi này nếu muốn giữ lại giá trị trước khi gọi một hàm khác (vì hàm bị gọi có thể ghi đè lên chúng).
*   **Callee saved (Hàm bị gọi lưu):** Nếu một hàm muốn sử dụng các thanh ghi này, nó phải `push` giá trị cũ vào Stack và `pop` lại trước khi trả về để đảm bảo không làm hỏng dữ liệu của hàm đã gọi nó. (Đây là lý do bạn thường thấy lệnh `push rbx` ở đầu hàm trong IDA).

Các quy ước này sẽ được trình bày chi tiết hơn trong **Mục 3.7**, nơi mô tả cách triển khai các thủ tục (procedures).

### 3.4.1 Operand Specifiers (Các loại toán hạng)

Hầu hết các lệnh x86-64 có một hoặc nhiều toán hạng (operands), chỉ định các giá trị nguồn (source) để thực hiện phép toán và vị trí đích (destination) để lưu kết quả. Có 3 loại toán hạng chính:

#### 1. Immediate (Giá trị tức thời)
*   **Ký hiệu trong sách:** `$Imm` (ví dụ: `$-577` hoặc `$0x1F`).
*   **Cách hiểu:** Đây là các **hằng số** (constants) được nhúng trực tiếp vào lệnh.
*   **Trong IDA Pro:** Hiển thị số thuần túy (thường là hệ 16), ví dụ: `1Fh` hoặc `0FFFFFDC7h`.

#### 2. Register (Thanh ghi)
*   **Ký hiệu trong sách:** $r_a$ (ví dụ: `%rax`).
*   **Cách hiểu:** Giá trị đang nằm trong 1 trong 16 thanh ghi.
*   **Trong IDA Pro:** Tên thanh ghi như `rax`, `ebx`, `r10`.

#### 3. Memory Reference (Tham chiếu bộ nhớ)
Đây là phần gây rối nhất. Bản chất là chúng ta đang tính toán một **Địa chỉ hiệu dụng (Effective Address)** để truy cập vào RAM.

Hãy quên các ký hiệu $M[...]$ đi, đây là cách IDA Pro biểu diễn các chế độ địa chỉ (Addressing Modes) từ Figure 3.3:

| Loại địa chỉ | Sách (ATT) | **IDA Pro (Intel)** | Logic tính toán địa chỉ (Cách hiểu) |
| :--- | :--- | :--- | :--- |
| **Tuyệt đối (Absolute)** | `Imm` | `[0x12345]` | Truy cập thẳng vào địa chỉ `0x12345`. |
| **Gián tiếp (Indirect)** | `(%rax)` | `[rax]` | Lấy giá trị trong `rax` làm địa chỉ để tìm dữ liệu. |
| **Căn bản + Bù (Base + Displacement)** | `Imm(%rax)` | `[rax + 0x10]` | Lấy giá trị trong `rax` cộng thêm `0x10` làm địa chỉ. |
| **Chỉ số (Indexed)** | `(%rax, %rcx)` | `[rax + rcx]` | Cộng giá trị 2 thanh ghi lại để ra địa chỉ. |
| **Chỉ số có tỉ lệ (Scaled indexed)** | `( , %rcx, 4)` | `[rcx * 4]` | Lấy `rcx` nhân 4 để ra địa chỉ (thường dùng trong mảng). |
| **Đầy đủ nhất (General Form)** | `Imm(%rb, %ri, s)` | `[rb + ri*s + Imm]` | **Base + (Index * Scale) + Offset** |

---

### Giải mã "Công thức tổng quát" cực nhanh: `Imm(rb, ri, s)`

Khi bạn nhìn thấy một dòng lệnh phức tạp trong IDA như: `mov eax, [rbx + rcx*4 + 10h]`

1.  **`rb` (Base - Gốc):** Thanh ghi cơ sở (phải là thanh ghi 64-bit). Ở đây là `rbx`.
2.  **`ri` (Index - Chỉ số):** Thanh ghi chỉ số (phải là thanh ghi 64-bit). Ở đây là `rcx`.
3.  **`s` (Scale - Tỉ lệ):** Hệ số nhân, **bắt buộc** phải là 1, 2, 4, hoặc 8 (tương ứng với kích thước các kiểu dữ liệu `char`, `short`, `int`, `long`). Ở đây là `4`.
4.  **`Imm` (Displacement/Offset):** Một hằng số cộng thêm vào. Ở đây là `10h`.

**Tại sao phải phức tạp như vậy?**
Vì nó khớp hoàn hảo với cách lập trình:
*   `rb` là địa chỉ bắt đầu của một mảng (Array base).
*   `ri` là chỉ số phần tử (Index `i`).
*   `s` là kích thước của mỗi phần tử (ví dụ `int` là 4 bytes).
*   `Imm` là độ lệch để dịch chuyển đến một trường cụ thể trong Struct (nếu mảng đó chứa các Struct).

---


### Practice Problem 3.1: Tính toán giá trị toán hạng

<img width="651" height="512" alt="image" src="https://github.com/user-attachments/assets/226dfde6-9eda-4e1c-9185-608b6d1d6298" />


<details>
<summary><b>Nhấn để xem giải đề và phân tích logic</b></summary>

**Giả thuyết (Trạng thái hệ thống):**
*   **Bộ nhớ (Memory):**
    *   `0x100`: `0xFF` | `0x104`: `0xAB` | `0x108`: `0x13` | `0x10C`: `0x11`
*   **Thanh ghi (Registers):**
    *   `rax`: `0x100` | `rcx`: `0x1` | `rdx`: `0x3`

**Bảng tính toán giá trị:**

| Toán hạng (ATT) | Cách hiểu (Intel/IDA) | Phép tính (Logic) | Kết quả |
| :--- | :--- | :--- | :--- |
| `%rax` | `rax` | Giá trị trong thanh ghi `rax` | **`0x100`** |
| `0x104` | `[0x104]` | Giá trị tại địa chỉ bộ nhớ `0x104` | **`0xAB`** |
| `$0x108` | `0x108` | Hằng số (Immediate) | **`0x108`** |
| `(%rax)` | `[rax]` | Giá trị tại địa chỉ `0x100` | **`0xFF`** |
| `4(%rax)` | `[rax + 4]` | Giá trị tại địa chỉ `0x100 + 4 = 0x104` | **`0xAB`** |
| `9(%rax, %rdx)` | `[rax + rdx + 9]` | Giá trị tại `0x100 + 0x3 + 9 = 0x10C` | **`0x11`** |
| `260(%rcx, %rdx)` | `[rcx + rdx + 260]` | Lưu ý: 260 = `0x104`. Đ/c: `0x1 + 0x3 + 0x104 = 0x108` | **`0x13`** |
| `0xFC( , %rcx, 4)` | `[rcx * 4 + 0xFC]` | Địa chỉ: `(1 * 4) + 0xFC = 4 + 252 = 256` (`0x100`) | **`0xFF`** |
| `(%rax, %rdx, 4)` | `[rax + rdx * 4]` | Địa chỉ: `0x100 + (3 * 4) = 0x100 + 12` (`0x10C`) | **`0x11`** |

</details>

---

### 3.4.2 Data Movement Instructions (Các lệnh di chuyển dữ liệu)

Đây là nhóm lệnh được sử dụng nhiều nhất trong mọi chương trình. Chúng có nhiệm vụ sao chép dữ liệu từ vị trí này sang vị trí khác. 

#### Nhóm lệnh `MOV` (Instruction Class)
Nhóm lệnh này thực hiện việc sao chép dữ liệu từ **Nguồn (Source)** sang **Đích (Destination)** mà không thực hiện bất kỳ phép biến đổi nào. Có 4 biến thể dựa trên kích thước dữ liệu:

| Lệnh (ATT) | Lệnh (Intel/IDA) | Kích thước (Bytes) | Tên gọi |
| :--- | :--- | :--- | :--- |
| `movb` | `mov` (byte ptr) | 1 | Move byte |
| `movw` | `mov` (word ptr) | 2 | Move word |
| `movl` | `mov` (dword ptr) | 4 | Move double word |
| `movq` | `mov` (qword ptr) | 8 | Move quad word |

**Nguyên tắc cơ bản:**
1.  **Tính tổng quát:** Việc cho phép nhiều loại toán hạng (Immediate, Register, Memory) khiến một lệnh `mov` có thể diễn đạt rất nhiều tình huống trong mã nguồn C.
2.  **Sự tương đồng:** Cả 4 lệnh trên đều có hiệu ứng tương tự, chỉ khác nhau về số lượng byte mà chúng tác động (1, 2, 4, hoặc 8 bytes).

---

### Các quy tắc kết hợp Source/Destination

Trong kiến trúc x86-64, lệnh `MOV` có các giới hạn về việc kết hợp toán hạng mà bạn cần nhớ khi soi mã:

1.  **Source (Nguồn):** Có thể là giá trị tức thời (Immediate), thanh ghi (Register), hoặc bộ nhớ (Memory).
2.  **Destination (Đích):** Có thể là thanh ghi hoặc bộ nhớ.
3.  **Hạn chế quan trọng:** Không thể thực hiện sao chép trực tiếp từ **Memory sang Memory** trong một lệnh duy nhất.
    *   *Ví dụ:* Nếu muốn copy `*ptr1 = *ptr2`, máy tính phải tốn 2 bước: 
        *   Bước 1: Nạp từ bộ nhớ vào thanh ghi (`Memory -> Register`).
        *   Bước 2: Ghi từ thanh ghi vào bộ nhớ (`Register -> Memory`).

### Hình 3.4: Các lệnh di chuyển dữ liệu đơn giản

<img width="665" height="277" alt="image" src="https://github.com/user-attachments/assets/a9259c89-ca79-4b1a-9c03-ce3d35f5cb8a" />


| Lệnh (ATT) | Lệnh (Intel/IDA) | Hiệu ứng | Mô tả |
| :--- | :--- | :--- | :--- |
| `movb S, D` | `mov D, S` | $D \leftarrow S$ | Ghi 1 byte (Move byte) |
| `movw S, D` | `mov D, S` | $D \leftarrow S$ | Ghi 2 bytes (Move word) |
| `movl S, D` | `mov D, S` | $D \leftarrow S$ | Ghi 4 bytes (Move double word) |
| `movq S, D` | `mov D, S` | $D \leftarrow S$ | Ghi 8 bytes (Move quad word) |
| `movabsq I, R` | `movabs R, I` | $R \leftarrow I$ | Ghi hằng số 64-bit vào thanh ghi |

---

### Quy tắc về Toán hạng và Kích thước

1.  **Hạn chế Memory-to-Memory:** x86-64 không cho phép một lệnh `mov` có cả hai toán hạng đều là địa chỉ bộ nhớ. Để chép dữ liệu từ vùng nhớ này sang vùng nhớ khác, bạn phải dùng một thanh ghi trung gian (ví dụ: nạp vào `rax` rồi ghi từ `rax` ra đích).
2.  **Khớp kích thước:** Kích thước của thanh ghi phải khớp với hậu tố của lệnh (`b`, `w`, `l`, hoặc `q`).
    *   `movb` dùng với các thanh ghi 8-bit (`al`, `bl`, `cl`...).
    *   `movw` dùng với các thanh ghi 16-bit (`ax`, `bx`, `cx`...).
    *   `movl` dùng với các thanh ghi 32-bit (`eax`, `ebx`, `ecx`...).
    *   `movq` dùng với các thanh ghi 64-bit (`rax`, `rbx`, `rcx`...).

### Ngoại lệ quan trọng của `movl` (Zero-Extension)
Trong hầu hết các trường hợp, lệnh `mov` chỉ cập nhật đúng số byte được chỉ định. Tuy nhiên:
*   **Khi `movl` có đích đến là một thanh ghi:** Nó sẽ tự động đặt 4 byte cao (32-bit phía trên) của thanh ghi đó về **0**. 
*   **Tại sao?** Đây là quy ước của kiến trúc x86-64: mọi lệnh tạo ra giá trị 32-bit cho một thanh ghi đều sẽ xóa phần 32-bit cao về 0.

---

### Ví dụ về các tổ hợp lệnh `MOV`

Dưới đây là 5 cách kết hợp nguồn/đích phổ biến:

| STT | Mã ATT (Sách) | Mã Intel (IDA Pro) | Loại (Nguồn -> Đích) | Kích thước |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `movl $0x4050, %eax` | `mov eax, 4050h` | Immediate -> Register | 4 bytes |
| 2 | `movw %bp, %sp` | `mov sp, bp` | Register -> Register | 2 bytes |
| 3 | `movb (%rdi, %rcx), %al` | `mov al, [rdi + rcx]` | Memory -> Register | 1 byte |
| 4 | `movb $-17, (%rsp)` | `mov byte ptr [rsp], 0EFh` | Immediate -> Memory | 1 byte |
| 5 | `movq %rax, -12(%rbp)` | `mov [rbp - 12], rax` | Register -> Memory | 8 bytes |

---

### Lệnh `movabsq` (Move Absolute Quadword)

Lệnh `movq` thông thường chỉ có thể nhận giá trị tức thời (immediate) tối đa là **32-bit** (sau đó nó sẽ thực hiện *sign-extend* để thành 64-bit).

*   **Khi nào dùng `movabsq`?** Khi bạn cần nạp một hằng số 64-bit cực lớn (arbitrary 64-bit immediate) trực tiếp vào thanh ghi.
*   **Đặc điểm:** 
    *   Nguồn: Phải là một giá trị tức thời 64-bit.
    *   Đích: **Bắt buộc** phải là một thanh ghi (không thể ghi trực tiếp vào bộ nhớ).
*   **Trong IDA Pro:** IDA thường tự nhận diện và hiển thị là `mov rax, <giá trị_64_bit>`.

---

Tiếp theo (Hình 3.5 và 3.6), chúng ta sẽ xem xét hai lớp lệnh `MOV` dùng để sao chép dữ liệu từ một nguồn nhỏ (ví dụ 1 byte) sang một đích lớn hơn (ví dụ 8 byte), bao gồm các cơ chế bù không (zero-extension) và bù dấu (sign-extension).

---

### Ví dụ minh họa: Cách lệnh di chuyển dữ liệu thay đổi thanh ghi đích (Aside)

Để hiểu rõ hơn về quy tắc cập nhật các byte bậc cao (upper bytes) của thanh ghi, hãy xem xét chuỗi lệnh sau đây. Giả sử ban đầu thanh ghi `%rax` được nạp một giá trị mẫu:

| STT | Lệnh (ATT) | Lệnh (Intel/IDA) | Giá trị của `%rax` (Dạng Hex) | Giải thích tác động |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `movabsq $0x0011223344556677, %rax` | `mov rax, 11223344556677h` | `0011223344556677` | Khởi tạo giá trị 64-bit ban đầu. |
| 2 | `movb $-1, %al` | `mov al, 0FFh` | `00112233445566FF` | **Chỉ đổi 1 byte thấp** (`al`). Các byte khác giữ nguyên. |
| 3 | `movw $-1, %ax` | `mov ax, 0FFFFh` | `001122334455FFFF` | **Chỉ đổi 2 byte thấp** (`ax`). Các byte khác giữ nguyên. |
| 4 | `movl $-1, %eax` | `mov eax, 0FFFFFFFFh` | `00000000FFFFFFFF` | **Ghi 4 byte thấp VÀ xóa 4 byte cao về 0**. (Quy tắc Zero-extend). |
| 5 | `movq $-1, %rax` | `mov rax, 0FFFFFFFFFFFFFFFFh`| `FFFFFFFFFFFFFFFF` | **Ghi toàn bộ 8 byte**. |

**Điểm cần lưu ý đặc biệt khi dùng IDA Pro:**
*   Lệnh `movl` (dòng 4) thực hiện quy ước của x86-64: bất kỳ lệnh nào ghi vào phần 32-bit của thanh ghi (ví dụ `eax`) đều sẽ tự động xóa sạch phần 32-bit cao của thanh ghi 64-bit tương ứng (`rax`) về 0.

---

### Hình 3.5: Các lệnh di chuyển dữ liệu bù không (Zero-extending)

<img width="847" height="321" alt="image" src="https://github.com/user-attachments/assets/9bb10a4f-0c8a-4a4a-9853-1793be47b699" />


Khi bạn muốn copy dữ liệu từ một nguồn nhỏ (1 hoặc 2 bytes) sang một đích lớn hơn (thanh ghi), bạn cần quyết định xem các byte còn lại của đích sẽ được lấp đầy như thế nào. Lớp lệnh **`MOVZ`** sẽ lấp đầy các byte còn lại bằng số **0**.

| Lệnh (ATT) | Lệnh (Intel/IDA) | Hiệu ứng | Mô tả |
| :--- | :--- | :--- | :--- |
| `movzbw S, R` | `movzx r16, r/m8` | $R \leftarrow \text{ZeroExtend}(S)$ | Chép 1 byte sang 2 bytes. |
| `movzbl S, R` | `movzx r32, r/m8` | $R \leftarrow \text{ZeroExtend}(S)$ | Chép 1 byte sang 4 bytes. |
| `movzwl S, R` | `movzx r32, r/m16`| $R \leftarrow \text{ZeroExtend}(S)$ | Chép 2 bytes sang 4 bytes. |
| `movzbq S, R` | `movzx r64, r/m8` | $R \leftarrow \text{ZeroExtend}(S)$ | Chép 1 byte sang 8 bytes. |
| `movzwq S, R` | `movzx r64, r/m16`| $R \leftarrow \text{ZeroExtend}(S)$ | Chép 2 bytes sang 8 bytes. |

**Quy tắc đặt tên lệnh:**
*   Trong cú pháp ATT (sách), hai ký tự cuối chỉ định kích thước: Ký tự thứ nhất là **Nguồn**, ký tự thứ hai là **Đích**.
    *   `b`: byte (1 byte)
    *   `w`: word (2 bytes)
    *   `l`: double word (4 bytes)
    *   `q`: quad word (8 bytes)
*   Trong IDA Pro (Intel), lệnh này được gọi là **`MOVZX`** (Move with Zero-eXtend).

---

### Phân tích bổ sung

1.  **Nguồn và Đích:** Nguồn (`S`) có thể là thanh ghi hoặc địa chỉ bộ nhớ. Đích (`R`) **bắt buộc phải là thanh ghi**.
2.  **Tại sao không có lệnh `movzlq`?** 
    *   Sách chỉ ra rằng: Chúng ta không cần lệnh để bù không từ 4 byte sang 8 byte (double word to quad word). 
    *   Lý do: Như đã thấy ở ví dụ trên, lệnh `movl` bình thường khi ghi vào thanh ghi 32-bit đã tự động thực hiện việc bù không cho 4 byte cao của thanh ghi 64-bit rồi.
3.  **Lớp lệnh đối nghịch:** Ngoài `movz` (bù không), chúng ta còn lớp lệnh `movs` (bù dấu - sign extension) để xử lý các số nguyên có dấu, sẽ được trình bày ngay sau đây.

---

### Hình 3.6: Các lệnh di chuyển dữ liệu bù dấu (Sign-extending)

Lớp lệnh **`MOVS`** sao chép dữ liệu từ nguồn nhỏ sang đích lớn hơn, nhưng khác với `MOVZ`, nó lấp đầy các byte còn lại bằng cách **sao chép bit dấu** (bit quan trọng nhất - MSB) của giá trị nguồn.

| Lệnh (ATT) | Lệnh (Intel/IDA) | Hiệu ứng | Mô tả |
| :--- | :--- | :--- | :--- |
| `movsbw` | `movsx r16, r/m8` | $R \leftarrow \text{SignExtend}(S)$ | Chép 1 byte sang 2 bytes (có dấu). |
| `movsbl` | `movsx r32, r/m8` | $R \leftarrow \text{SignExtend}(S)$ | Chép 1 byte sang 4 bytes (có dấu). |
| `movswl` | `movsx r32, r/m16`| $R \leftarrow \text{SignExtend}(S)$ | Chép 2 bytes sang 4 bytes (có dấu). |
| `movsbq` | `movsx r64, r/m8` | $R \leftarrow \text{SignExtend}(S)$ | Chép 1 byte sang 8 bytes (có dấu). |
| `movswq` | `movsx r64, r/m16`| $R \leftarrow \text{SignExtend}(S)$ | Chép 2 bytes sang 8 bytes (có dấu). |
| `movslq` | **`movsxd r64, r/m32`** | $R \leftarrow \text{SignExtend}(S)$ | Chép 4 bytes sang 8 bytes (có dấu). |
| **`cltq`** | **`cdqe`** | `%rax` $← SignExtend$ (`%eax`) | Mở rộng dấu `%eax` vào `%rax`. |

**Lưu ý quan trọng:**
*   Trong IDA Pro (Intel), hậu tố `x` thường được thêm vào (`movsx`). Riêng trường hợp từ 4 byte (double word) lên 8 byte (quad word), Intel sử dụng lệnh **`movsxd`**.
*   **Sự thiếu vắng của `movzlq`:** Như đã nhắc ở trang trước, không cần lệnh bù không 4-sang-8 vì lệnh `movl` bình thường đã làm việc đó rồi. Tuy nhiên, với bù dấu, ta **bắt buộc** phải có lệnh `movslq` (hoặc `movsxd` trong IDA).

---

### Lệnh `cltq` (Convert Long to Quadword)

Đây là một lệnh không có toán hạng, nó luôn tác động cố định trên hai thanh ghi `%eax` và `%rax`.
*   **Chức năng:** Nó thực hiện chính xác việc mở rộng dấu từ `%eax` lên `%rax`.
*   **So sánh:** Lệnh này có kết quả y hệt như lệnh `movslq %eax, %rax` (ATT) hay `movsxd rax, eax` (Intel).
*   **Tại sao tồn tại?** Nó có cách mã hóa (encoding) gọn nhẹ hơn (chỉ tốn ít byte hơn trong mã máy), nên thường được các trình biên dịch ưu tiên sử dụng.
*   **Trong IDA Pro:** Bạn sẽ thấy lệnh này hiển thị là **`cdqe`**.

---

### Practice Problem 3.2: Xác định hậu tố lệnh dựa trên toán hạng

<img width="533" height="210" alt="image" src="https://github.com/user-attachments/assets/6d2db509-2a77-49f6-83e3-5412d60c7ad6" />


<details>
<summary><b>Nhấn để xem giải đề và phân tích kích thước</b></summary>

Dựa trên kích thước của các thanh ghi đích hoặc nguồn, ta xác định hậu tố (`b`, `w`, `l`, `q`) cho lệnh `mov`:

| Lệnh Assembly (Chưa có hậu tố) | Hậu tố đúng | Lý do (Kích thước) |
| :--- | :--- | :--- |
| `mov %eax, (%rsp)` | **`movl`** | `%eax` là 4 bytes. |
| `mov (%rax), %dx` | **`movw`** | `%dx` là 2 bytes. |
| `mov $0xFF, %bl` | **`movb`** | `%bl` là 1 byte. |
| `mov (%rsp,%rdx,4), %dl` | **`movb`** | `%dl` là 1 byte. |
| `mov (%rdx), %rax` | **`movq`** | `%rax` là 8 bytes. |
| `mov %dx, (%rax)` | **`movw`** | `%dx` là 2 bytes. |

</details>

---

### So sánh các lệnh di chuyển Byte (Aside)

Ví dụ sau minh họa sự khác biệt về cách các lệnh `movb`, `movsbq`, và `movzbq` thay đổi các byte bậc cao của thanh ghi đích. 
Giả sử ban đầu: `%rax = 0011223344556677` và `%dl = AA` (hệ nhị phân của `AA` là `10101010`, có bit dấu là **1**).

| STT | Lệnh (ATT) | Lệnh (Intel/IDA) | Giá trị của `%rax` (Hex) | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `movb %dl, %al` | `mov al, dl` | `00112233445566AA` | **Không đổi** các byte khác, chỉ đổi 1 byte thấp. |
| 2 | `movsbq %dl, %rax` | `movsx rax, dl` | `FFFFFFFFFFFFFFAA` | **Bù dấu**: Vì bit cao nhất của `AA` là 1, nên 7 byte còn lại được lấp đầy bởi `F`. |
| 3 | `movzbq %dl, %rax` | `movzx rax, dl` | `00000000000000AA` | **Bù không**: 7 byte còn lại được lấp đầy bởi số 0. |

---

### Practice Problem 3.3: Chẩn đoán lỗi mã Assembly

<img width="539" height="235" alt="image" src="https://github.com/user-attachments/assets/391175f3-1a93-440e-b63c-896bb8931923" />


<details>
<summary><b>Nhấn để xem giải thích các lỗi sai</b></summary>

Mỗi dòng lệnh dưới đây đều gây lỗi khi biên dịch. Dưới đây là lý do:

| Lệnh lỗi (ATT) | Lý do sai |
| :--- | :--- |
| `movb $0xF, (%ebx)` | Không thể dùng thanh ghi 32-bit (`%ebx`) làm địa chỉ trong chế độ 64-bit (phải dùng `%rbx`). |
| `movl %rax, (%rsp)` | Sai kích thước: `%rax` là 8 bytes, nhưng `movl` chỉ dành cho 4 bytes. |
| `movw (%rax), 4(%rsp)` | Lỗi **Memory-to-Memory**: Không thể copy trực tiếp từ vùng nhớ này sang vùng nhớ khác. |
| `movb %al, %sl` | Tên thanh ghi sai: Thanh ghi 8-bit bậc thấp của `%rsi` phải là `%sil`, không phải `%sl`. |
| `movq %rax, $0x123` | Đích đến không thể là một hằng số (Immediate). |
| `movl %eax, %rdx` | Sai kích thước: Đích của lệnh 4-byte (`movl`) phải là thanh ghi 32-bit (ví dụ `%edx`). |
| `movb %si, 8(%rbp)` | Sai kích thước: `%si` là 2 bytes, nhưng lệnh `movb` chỉ dành cho 1 byte. |

</details>

---

### 3.4.3 Data Movement Example (Ví dụ di chuyển dữ liệu)

Hãy xem xét hàm `exchange` dưới đây. Đây là một ví dụ điển hình về việc sử dụng các lệnh di chuyển dữ liệu để hoán đổi giá trị.

**Quy ước quan trọng trước khi phân tích:**
1.  **Tham số:** Các tham số được truyền qua thanh ghi.
    *   Tham số thứ 1 (`xp` - con trỏ): nằm trong `%rdi`.
    *   Tham số thứ 2 (`y` - giá trị): nằm trong `%rsi`.
2.  **Giá trị trả về:** Được lưu vào thanh ghi `%rax`.
3.  **Lệnh `ret`:** Dùng để quay trở về hàm đã gọi.

#### (a) Mã nguồn C
```c
long exchange(long *xp, long y)
{
    long x = *xp;
    *xp = y;
    return x;
}
```

#### Phân tích sơ bộ:
Hàm `exchange` được triển khai chỉ với 3 lệnh: hai lệnh di chuyển dữ liệu (`movq`) và một lệnh trả về (`ret`).

### Phân tích chi tiết hàm `exchange`

#### (b) Mã Assembly (Đối chiếu ATT và Intel/IDA Pro)

Dựa trên quy ước tham số: `xp` nằm trong `%rdi`, `y` nằm trong `%rsi`.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `exchange:` | `exchange:` | Nhãn bắt đầu hàm. |
| 2 | `movq (%rdi), %rax` | `mov rax, [rdi]` | **Đọc từ bộ nhớ**: Lấy giá trị tại địa chỉ `xp` lưu vào `%rax` (tương ứng `x = *xp`). |
| 3 | `movq %rsi, (%rdi)` | `mov [rdi], rsi` | **Ghi vào bộ nhớ**: Chép giá trị `y` vào địa chỉ mà `xp` trỏ tới (tương ứng `*xp = y`). |
| 4 | `ret` | `retn` | Trả về. Lúc này `%rax` đã chứa `x`, nên kết quả trả về đúng là `x`. |

---

### Các bài học quan trọng về dịch ngược:

1.  **Bản chất của con trỏ:** "Con trỏ" trong C thực chất chỉ là các **địa chỉ bộ nhớ**. Việc giải mã con trỏ (dereferencing) trong Assembly đơn giản là đặt thanh ghi chứa địa chỉ đó vào trong dấu ngoặc vuông (Intel: `[rdi]`, ATT: `(%rdi)`).
2.  **Biến cục bộ:** Các biến cục bộ (như `long x`) thường được trình biên dịch giữ lại trong **thanh ghi** thay vì cấp phát trên bộ nhớ (Stack) nếu có thể. Điều này giúp tốc độ truy cập nhanh hơn nhiều so với RAM.
3.  **Giá trị trả về:** Thay vì tốn thêm lệnh để di chuyển dữ liệu, trình biên dịch khôn ngoan nạp thẳng giá trị `*xp` vào `%rax` ngay từ đầu. Vì `%rax` là thanh ghi mặc định cho giá trị trả về, logic `return x` được thực hiện "miễn phí".

---

### Practice Problem 3.4: Chuyển đổi kiểu dữ liệu và Di chuyển dữ liệu

<img width="429" height="819" alt="image" src="https://github.com/user-attachments/assets/1677ba08-8d44-4fe6-9ea4-5452d3330067" />

<br>

> Do cái đề quá dài và đề hơi mờ nên mình để cả đề dưới dạng tiếng Việt ở dưới

**Đề bài:**
Giả sử ta có hai biến con trỏ `sp` và `dp` được khai báo như sau:
```c
src_t *sp;
dest_t *dp;
```
Trong đó `src_t` và `dest_t` là các kiểu dữ liệu được định nghĩa bằng `typedef`. Chúng ta cần sử dụng **hai lệnh** di chuyển dữ liệu phù hợp để thực hiện phép gán:
`*dp = (dest_t) *sp;`

**Giả định hệ thống:**
*   Giá trị của `sp` (địa chỉ nguồn) nằm trong thanh ghi `%rdi`.
*   Giá trị của `dp` (địa chỉ đích) nằm trong thanh ghi `%rsi`.
*   **Lệnh 1:** Đọc từ bộ nhớ vào một phần phù hợp của thanh ghi `%rax` (`%rax`, `%eax`, `%ax`, hoặc `%al`) và thực hiện chuyển đổi kiểu dữ liệu (bù không hoặc bù dấu nếu cần).
*   **Lệnh 2:** Ghi phần tương ứng đó của thanh ghi `%rax` vào bộ nhớ tại địa chỉ đích.
*   **Quy tắc chuyển đổi (Section 2.2.6):** Khi thay đổi cả **kích thước** và **tính có dấu (signedness)**, phép toán phải thực hiện **thay đổi kích thước trước**.

<details>
<summary><b>Nhấn để xem phân tích các trường hợp</b></summary>

---

Giả sử `sp` (nguồn) nằm trong `%rdi` và `dp` (đích) nằm trong `%rsi`. 
**Quy tắc:** Khi thay đổi cả kích thước và tính có dấu (signness), hãy thực hiện **thay đổi kích thước trước**.

| Kiểu nguồn (`src_t`) | Kiểu đích (`dest_t`) | Lệnh 1 (Đọc & Chuyển đổi) | Lệnh 2 (Ghi vào đích) | Giải thích (IDA/Intel) |
| :--- | :--- | :--- | :--- | :--- |
| `long` | `long` | `movq (%rdi), %rax` | `movq %rax, (%rsi)` | `mov rax, [rdi]` / `mov [rsi], rax` |
| `char` | `int` | `movsbl (%rdi), %eax` | `movl %eax, (%rsi)` | **Bù dấu** (1 byte signed -> 4 bytes) |
| `char` | `unsigned` | `movsbl (%rdi), %eax` | `movl %eax, (%rsi)` | Chuyển kích thước trước (bù dấu), sau đó coi là unsigned. |
| `unsigned char` | `long` | `movzbq (%rdi), %rax` | `movq %rax, (%rsi)` | **Bù không** (1 byte unsigned -> 8 bytes) |
| `int` | `char` | `movl (%rdi), %eax` | `movb %al, (%rsi)` | Đọc 4 byte, chỉ ghi lại 1 byte thấp (`al`). |
| `unsigned` | `unsigned char` | `movl (%rdi), %eax` | `movb %al, (%rsi)` | Tương tự trên, cắt cụt (truncation). |
| `char` | `short` | `movsbw (%rdi), %ax` | `movw %ax, (%rsi)` | **Bù dấu** (1 byte -> 2 bytes). |

---


</details>

---

### Kiến thức bổ sung: Ví dụ về con trỏ trong C (Aside)

Hàm `exchange` cung cấp một minh họa tốt về cách sử dụng con trỏ. 
*   **Giải mã con trỏ (Dereferencing):** 
    *   Lệnh `long x = *xp;` là một thao tác **đọc** từ bộ nhớ. Ta lấy giá trị tại địa chỉ mà `xp` trỏ tới và lưu vào biến `x`.
    *   Lệnh `*xp = y;` là một thao tác **ghi** vào bộ nhớ. Ta viết giá trị của `y` vào địa chỉ mà `xp` đang nắm giữ.
*   **Toán tử lấy địa chỉ (`&`):** Tạo ra một con trỏ trỏ đến vị trí của biến.

**Ví dụ thực tế:**
```c
long a = 4;
long b = exchange(&a, 3);
printf("a = %ld, b = %ld\n", a, b);
```
**Kết quả:** `a = 3, b = 4`. 
Việc truyền con trỏ `&a` vào hàm cho phép hàm `exchange` thay đổi giá trị của biến `a` nằm ở một vùng nhớ từ xa (ngoài phạm vi của hàm).

---

### Practice Problem 3.5

<img width="578" height="404" alt="image" src="https://github.com/user-attachments/assets/0a7e10db-ab7f-44e2-9212-f3c847ff9246" />


<details>
   <summary>Bấm vào để xem lời giải</summary>
   <br>
   
Mã Assembly tương ứng được tạo ra như sau:

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Phân tích bước đi của dữ liệu |
| :--- | :--- | :--- | :--- |
| 1 | `movq (%rdi), %r8` | `mov r8, [rdi]` | Đọc `*xp` vào thanh ghi `%r8`. |
| 2 | `movq (%rsi), %rcx` | `mov rcx, [rsi]` | Đọc `*yp` vào thanh ghi `%rcx`. |
| 3 | `movq (%rdx), %rax` | `mov rax, [rdx]` | Đọc `*zp` vào thanh ghi `%rax`. |
| 4 | `movq %r8, (%rsi)` | `mov [rsi], r8` | Ghi giá trị (cũ của `*xp`) vào địa chỉ `yp`. |
| 5 | `movq %rcx, (%rdx)` | `mov [rdx], rcx` | Ghi giá trị (cũ của `*yp`) vào địa chỉ `zp`. |
| 6 | `movq %rax, (%rdi)` | `mov [rdi], rax` | Ghi giá trị (cũ của `*zp`) vào địa chỉ `xp`. |
| 7 | `ret` | `retn` | Trả về. |

---

### Phân tích logic từng bước (Step-by-Step)

Để viết lại mã C, ta hãy đặt tên các biến cục bộ tương ứng với các thanh ghi tạm thời mà trình biên dịch đã sử dụng:

1.  **Lấy dữ liệu nguồn:**
    *   `long temp_x = *xp;` (Lệnh 1 sử dụng `%r8`)
    *   `long temp_y = *yp;` (Lệnh 2 sử dụng `%rcx`)
    *   `long temp_z = *zp;` (Lệnh 3 sử dụng `%rax`)

2.  **Hoán đổi vị trí (Cyclic Swap):**
    *   `*yp = temp_x;` (Lệnh 4: lấy giá trị cũ của `xp` ghi vào `yp`)
    *   `*zp = temp_y;` (Lệnh 5: lấy giá trị cũ của `yp` ghi vào `zp`)
    *   `*xp = temp_z;` (Lệnh 6: lấy giá trị cũ của `zp` ghi vào `xp`)

---

### Lời giải mã C hoàn chỉnh

Dựa trên phân tích luồng dữ liệu ở trên, đây là nội dung hàm `decode1` trong C:

```c
void decode1(long *xp, long *yp, long *zp) 
{
    long x = *xp;
    long y = *yp;
    long z = *zp;

    *yp = x;
    *zp = y;
    *xp = z;
}
```

---

### IDA Pro Insights (Góc nhìn dịch ngược)

Khi bạn gặp một đoạn mã như thế này trong IDA Pro, hãy chú ý các đặc điểm sau:
*   **Ba tham số đầu tiên:** IDA sẽ tự nhận diện `%rdi`, `%rsi`, `%rdx` là `arg_0`, `arg_1`, `arg_2` (hoặc tên thực nếu có thông tin debug).
*   **Thanh ghi tạm:** Việc trình biên dịch sử dụng `%r8`, `%rcx`, `%rax` cho thấy đây là 3 biến cục bộ kiểu `long` (8-byte).
*   **Mẫu hình (Pattern):** Đây là mẫu hình của một phép **hoán đổi vòng tròn (cyclic swap)**. Dữ liệu được "nhấc" hết lên các thanh ghi trước, sau đó mới "đặt" ngược lại vào các địa chỉ bộ nhớ theo thứ tự mới. Điều này giúp tránh việc ghi đè làm mất dữ liệu trước khi kịp sao lưu.

---

</details>

---

### 3.4.4 Pushing and Popping Stack Data (Đẩy vào và Lấy ra khỏi Ngăn xếp)

Ngăn xếp (Stack) đóng vai trò sống còn trong việc xử lý các lời gọi thủ tục (procedure calls). Trong x86-64, ngăn xếp là một vùng nhớ hoạt động theo nguyên lý **LIFO** (Vào sau, Ra trước - Last-In, First-Out).

#### Đặc điểm của Stack trong x86-64:
*   **Hướng phát triển:** Ngăn xếp phát triển **ngược xuống (downward)**, nghĩa là các phần tử mới sẽ được đẩy vào các địa chỉ bộ nhớ thấp hơn.
*   **Đỉnh ngăn xếp (Stack Top):** Là phần tử có địa chỉ **thấp nhất** trong tất cả các phần tử thuộc ngăn xếp.
*   **Thanh ghi `%rsp` (Stack Pointer):** Luôn lưu trữ địa chỉ của phần tử nằm ở đỉnh ngăn xếp.

---

### Hình 3.8: Các lệnh Push và Pop

<img width="753" height="247" alt="image" src="https://github.com/user-attachments/assets/54e911ef-fd91-4d07-886f-21652d83b034" />


Mỗi lệnh này chỉ nhận một toán hạng duy nhất: nguồn dữ liệu để đẩy vào (push) hoặc đích đến để lấy ra (pop).

| Lệnh (ATT) | Lệnh (Intel/IDA) | Hiệu ứng (Logic bên dưới) | Mô tả |
| :--- | :--- | :--- | :--- |
| `pushq S` | `push S` | `R[%rsp] ← R[%rsp] - 8;`<br>`M[R[%rsp]] ← S` | Giảm `%rsp` đi 8, sau đó ghi giá trị `S` vào địa chỉ mới của `%rsp`. |
| `popq D` | `pop D` | `D ← M[R[%rsp]];`<br>`R[%rsp] ← R[%rsp] + 8` | Đọc giá trị tại đỉnh ngăn xếp vào `D`, sau đó tăng `%rsp` thêm 8. |

---

### Hình 3.9: Minh họa hoạt động của Ngăn xếp

<img width="1002" height="757" alt="image" src="https://github.com/user-attachments/assets/28fa2e01-82ed-4840-8d1a-c87b19c911e5" />


Theo quy ước, chúng ta vẽ ngăn xếp "ngược", với phần đỉnh (top) nằm ở dưới cùng của hình vẽ.

1.  **Trạng thái ban đầu:** `%rsp` đang trỏ tại địa chỉ `0x108`.
2.  **Lệnh `pushq %rax` (với `%rax = 0x123`):**
    *   Thanh ghi `%rsp` giảm từ `0x108` xuống `0x100`.
    *   Giá trị `0x123` được ghi vào địa chỉ `0x100`.
3.  **Lệnh `popq %rdx`:**
    *   Giá trị `0x123` được đọc từ địa chỉ `0x100` và lưu vào thanh ghi `%rdx`.
    *   Thanh ghi `%rsp` tăng lại từ `0x100` lên `0x108`.

**Lưu ý quan trọng:**
*   Giá trị `0x123` vẫn còn tồn tại ở địa chỉ bộ nhớ `0x100` cho đến khi nó bị ghi đè bởi một thao tác `push` khác. Tuy nhiên, sau lệnh `pop`, địa chỉ đó không còn được coi là thuộc về ngăn xếp nữa vì `%rsp` đã trỏ đi nơi khác.

---

### IDA Pro Insights (Góc nhìn dịch ngược)

Khi quan sát trong IDA Pro, bạn cần lưu ý:
*   **Bản chất lệnh:** Lệnh `push rax` thực chất là một cách viết tắt của cặp lệnh:
    ```assembly
    sub rsp, 8
    mov [rsp], rax
    ```
*   **Bản chất lệnh Pop:** Lệnh `pop rdx` thực chất là viết tắt của:
    ```assembly
    mov rdx, [rsp]
    add rsp, 8
    ```
*   **Function Prologue/Epilogue:** IDA thường hiển thị rất nhiều lệnh `push` ở đầu hàm (để sao lưu các thanh ghi *callee-saved*) và các lệnh `pop` tương ứng ở cuối hàm (để khôi phục trạng thái bộ xử lý trước khi `ret`).
*   **Địa chỉ ảo:** Trong IDA, bạn có thể nhấn vào `rsp` hoặc xem cửa sổ "Stack View" để thấy các giá trị đang nằm trên ngăn xếp được biểu diễn trực quan tương tự như Hình 3.9.

---

### Phân tích sâu về cơ chế Push và Pop

Lệnh `pushq` và `popq` thực chất là các phím tắt cho chuỗi lệnh tính toán con trỏ và di chuyển dữ liệu.

#### 1. Cơ chế của `pushq %rbp`
Hành vi của lệnh này tương đương với cặp lệnh sau:

| AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Mô tả bước thực hiện |
| :--- | :--- | :--- |
| `subq $8, %rsp` | `sub rsp, 8` | Giảm con trỏ ngăn xếp (Tạo chỗ trống). |
| `movq %rbp, (%rsp)` | `mov [rsp], rbp` | Lưu giá trị vào đỉnh ngăn xếp mới. |

*   **Lợi ích:** Lệnh `pushq` chỉ tốn **1 byte** trong mã máy, trong khi cặp lệnh tương đương tốn tới 8 bytes.

#### 2. Cơ chế của `popq %rax`
Hành vi của lệnh này tương đương với cặp lệnh sau:

| AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Mô tả bước thực hiện |
| :--- | :--- | :--- |
| `movq (%rsp), %rax` | `mov rax, [rsp]` | Đọc giá trị từ đỉnh ngăn xếp vào thanh ghi. |
| `addq $8, %rsp` | `add rsp, 8` | Tăng con trỏ ngăn xếp (Giải phóng chỗ). |

#### 3. Truy cập ngăn xếp tùy ý
Vì ngăn xếp nằm trong cùng một vùng bộ nhớ với mã chương trình và dữ liệu khác, chúng ta có thể truy cập bất kỳ vị trí nào trong ngăn xếp bằng các phương pháp tham chiếu bộ nhớ tiêu chuẩn.
*   **Ví dụ:** Lệnh `movq 8(%rsp), %rdx` (ATT) hoặc `mov rdx, [rsp+8]` (Intel) sẽ sao chép phần tử thứ hai của ngăn xếp vào thanh ghi `%rdx` mà không làm thay đổi giá trị của `%rsp`.

---

## 3.5 Arithmetic and Logical Operations (Các phép toán số học và logic)

Hình 3.10 (ở trang sau) liệt kê các phép toán số nguyên và logic trong x86-64. Hầu hết các phép toán này đều có các biến thể kích thước khác nhau (ngoại trừ `leaq`).

Các phép toán được chia thành 4 nhóm:
1.  **Load effective address:** Tính toán địa chỉ hiệu dụng.
2.  **Unary:** Các phép toán một toán hạng (đơn phân).
3.  **Binary:** Các phép toán hai toán hạng (nhị phân).
4.  **Shifts:** Các phép toán dịch bit.

### 3.5.1 Load Effective Address (Nạp địa chỉ hiệu dụng - `leaq`)

Lệnh **`leaq`** (Load Effective Address) thực chất là một biến thể của lệnh `movq`. 
*   **Định dạng:** Nó có dạng giống như một lệnh đọc từ bộ nhớ vào thanh ghi.
*   **Điểm khác biệt cốt lõi:** Thay vì đọc dữ liệu từ địa chỉ được tính toán, nó **nạp chính địa chỉ đó** vào thanh ghi đích.

**IDA Pro Insight:** 
*   Trong cú pháp Intel, lệnh này là **`lea`**.
*   Khi bạn thấy `lea rax, [rdi+rsi*4]`, bộ xử lý không hề truy cập vào RAM tại địa chỉ đó. Nó chỉ thực hiện phép toán `rax = rdi + (rsi * 4)` và lưu kết quả vào `rax`. Trình biên dịch thường dùng lệnh này như một "mẹo" để thực hiện các phép toán cộng và nhân nhanh chóng mà không cần dùng các lệnh số học nặng nề.

---

### Practice Problem 3.6: Tính toán giá trị với lệnh `leaq`

Giả sử thanh ghi `%rbx` lưu giá trị $p$ và thanh ghi `%rdx` lưu giá trị $q$.

<img width="720" height="790" alt="image" src="https://github.com/user-attachments/assets/f6778857-8a50-41b1-ad82-584699103788" />


<details>
<summary><b>Nhấn để xem bảng giải mã công thức toán học</b></summary>

| Lệnh Assembly (ATT) | Lệnh Intel (IDA Pro) | Công thức tính toán (Kết quả lưu vào `%rax`) |
| :--- | :--- | :--- |
| `leaq 9(%rdx), %rax` | `lea rax, [rdx + 9]` | $q + 9$ |
| `leaq (%rdx,%rbx), %rax` | `lea rax, [rdx + rbx]` | $q + p$ |
| `leaq (%rdx,%rbx,3), %rax` | `lea rax, [rdx + rbx*2 + rbx]` | $q + 3p$ (Lưu ý: tỉ lệ 3 không hợp lệ, thực tế là $q + p \cdot 3$) |
| `leaq 2(%rbx,%rbx,7), %rax` | `lea rax, [rbx + rbx*7 + 2]` | $p + 7p + 2 = \mathbf{8p + 2}$ |
| `leaq 0xE(,%rdx,3), %rax` | `lea rax, [rdx*2 + rdx + 14]` | $3q + 14$ (Với $0xE = 14$ thập phân) |
| `leaq 6(%rbx,%rdx,7), %rax` | `lea rax, [rbx + rdx*7 + 6]` | $p + 7q + 6$ |

*Ghi chú kỹ thuật:* Trong x86-64 thực tế, hệ số tỉ lệ ($s$) chỉ có thể là 1, 2, 4, hoặc 8. Để nhân với 3, 7 hoặc các số khác, trình biên dịch kết hợp `Base + Index * Scale`. Ví dụ: `x * 3` được tính là `x + x * 2`.

</details>

---

### Ví dụ minh họa: Sử dụng `leaq` trong mã thực tế

Xét hàm C thực hiện tính toán biểu thức: $x + 4y + 12z$.

```c
long scale(long x, long y, long z) {
    long t = x + 4 * y + 12 * z;
    return t;
}
```

Khi biên dịch, thay vì dùng các lệnh nhân (`imul`) đắt đỏ, trình biên dịch GCC sử dụng chuỗi 3 lệnh `leaq` vô cùng khôn ngoan để đạt được kết quả:

| Dòng | Lệnh ATT (Sách) | Lệnh Intel (IDA Pro) | Phép tính logic | Trạng thái dữ liệu |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `leaq (%rdi,%rsi,4), %rax` | `lea rax, [rdi + rsi*4]` | $x + 4y$ | `%rax` = $x + 4y$ |
| 2 | `leaq (%rdx,%rdx,2), %rdx` | `lea rdx, [rdx + rdx*2]` | $z + 2z$ | `%rdx` = $3z$ |
| 3 | `leaq (%rax,%rdx,4), %rax` | `lea rax, [rax + rdx*4]` | $(x + 4y) + 4(3z)$ | `%rax` = $x + 4y + 12z$ |
| 4 | `ret` | `retn` | Trả về kết quả | Kết quả nằm trong `%rax` |

**Phân tích của trình biên dịch:**
*   **Dòng 1:** Tính $x + 4y$ và lưu tạm vào `%rax`.
*   **Dòng 2:** Nhân $z$ với 3 bằng cách lấy $z + 2z$, cập nhật lại vào `%rdx`.
*   **Dòng 3:** Lấy giá trị ở `%rax` cộng với $4 \times (\text{giá trị mới ở } \%rdx)$. Tức là: $(x + 4y) + 4 \times (3z) = x + 4y + 12z$.

---

### IDA Pro Insights (Góc nhìn tối ưu hóa)

*   **Lệnh LEA không phải là lệnh bộ nhớ:** Khi bạn soi IDA và thấy `lea`, hãy nhớ bộ xử lý đang dùng đơn vị tính toán địa chỉ (Address Generation Unit - AGU) để làm toán nhanh thay vì dùng đơn vị số học (ALU).
*   **Nhận diện biểu thức:** Trong IDA, nếu bạn thấy một chuỗi các lệnh `lea` liên tiếp tác động lên các thanh ghi tham số (`rdi`, `rsi`, `rdx`), đó là dấu hiệu của một biểu thức toán học đa biến.
*   **Tốc độ:** Khả năng thực hiện phép cộng và một dạng nhân giới hạn (nhân với 2, 4, 8) trong một chu trình đơn giúp `lea` cực kỳ hiệu quả trong các biểu thức số học đơn giản như ví dụ trên.

---

### Practice Problem 3.7: Dịch ngược biểu thức toán học phức tạp

<img width="960" height="607" alt="image" src="https://github.com/user-attachments/assets/91b43a68-cdfe-4c7b-87ae-89dc03f2de02" />


**Đề bài:**
Xét đoạn mã C dưới đây, trong đó biểu thức tính toán đã bị lược bỏ:

```c
short scale3(short x, short y, short z) {
    short t = ________________;
    return t;
}
```

Khi biên dịch hàm này bằng GCC, ta thu được mã Assembly sau:

**Ánh xạ thanh ghi:**
*   `x` nằm trong `%rdi`
*   `y` nằm trong `%rsi`
*   `z` nằm trong `%rdx`

**Mã Assembly:**

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Phép tính logic (Từng bước) |
| :--- | :--- | :--- | :--- |
| 1 | `leaq (%rsi,%rsi,2), %rdx` | `lea rdx, [rsi + rsi*2]` | `%rdx = y + 2*y = 3*y` |
| 2 | `leaq (%rdx,%rdi,4), %rax` | `lea rax, [rdx + rdi*4]` | `%rax = (3*y) + (4*x)` |
| 3 | `leaq (%rax,%rdx,2), %rax` | `lea rax, [rax + rdx*2]` | `%rax = (3*y + 4*x) + 2*(3*y)` |
| 4 | `ret` | `retn` | Trả về kết quả | Kết quả nằm trong `%rax` |

---

<details>
   <summary>Phân tích và Giải mã biểu thức C</summary>

Để tìm biểu thức của `t`, ta đi theo luồng biến đổi của các thanh ghi:

1.  **Bước 1 (Lệnh 1):** Trình biên dịch tính `3 * y` và lưu vào thanh ghi `%rdx`. Lưu ý rằng giá trị gốc của tham số `z` ban đầu nằm ở `%rdx` đã bị ghi đè hoàn toàn. Điều này có nghĩa là tham số `z` **không được sử dụng** trong biểu thức này.
2.  **Bước 2 (Lệnh 2):** Lấy kết quả vừa tính (`3 * y`) cộng với `4 * x`. Kết quả tạm thời là `4 * x + 3 * y`, lưu vào `%rax`.
3.  **Bước 3 (Lệnh 3):** Lấy giá trị ở `%rax` cộng với `2 * %rdx`. Vì `%rdx` đang giữ giá trị `3 * y`, phép toán sẽ là:
    $t = (4 * x + 3 * y) + 2 * (3 * y)$
    $t = 4 * x + 3 * y + 6 * y$
    $t = 4 * x + 9 * y$

### Lời giải mã C hoàn chỉnh:

Dựa trên phân tích trên, biểu thức còn thiếu trong mã C là:

```c
short scale3(short x, short y, short z) {
    short t = 4 * x + 9 * y;
    return t;
}
```

</details>

---

### IDA Pro Insights (Lưu ý về tối ưu hóa)

*   **Tham số bị bỏ rơi:** Khi dịch ngược trong IDA, nếu bạn thấy một thanh ghi tham số (như `%rdx` cho tham số thứ 3) bị ghi đè ngay dòng đầu tiên mà chưa được sử dụng, bạn có thể kết luận hàm đó không dùng đến tham số đó (unused parameter).
*   **Sử dụng lại kết quả trung gian:** Thay vì tính `9 * y` một cách trực tiếp (không thể làm được chỉ với 1 lệnh `lea` vì hệ số tỉ lệ tối đa là 8), trình biên dịch đã chọn cách tính `3 * y` trước, sau đó lấy `(3 * y) + 2 * (3 * y)` để ra `9 * y`. Đây là một kỹ thuật tối ưu hóa phổ biến để tiết kiệm số lượng lệnh thực thi.
*   **Kích thước dữ liệu:** Mặc dù kiểu dữ liệu là `short` (16-bit), trình biên dịch vẫn sử dụng `leaq` (64-bit) để tính toán nhằm tận dụng tối đa tốc độ của bộ nạp địa chỉ, sau đó kết quả trả về sẽ tự động được hiểu là 16-bit thấp của `%rax`.

---

### 3.5.2 Unary and Binary Operations (Phép toán đơn phân và nhị phân)

#### 1. Unary Operations (Phép toán một toán hạng)
Các phép toán trong nhóm này chỉ có một toán hạng duy nhất, đóng vai trò là **cả nguồn và đích**.
*   **Toán hạng:** Có thể là một thanh ghi hoặc một vị trí bộ nhớ.
*   **Ví dụ:** Lệnh `incq (%rsp)` (ATT) sẽ làm tăng giá trị của phần tử 8-byte nằm ở đỉnh ngăn xếp thêm 1 đơn vị. 
*   **Tương ứng trong C:** Cú pháp này gợi nhớ đến các toán tử tăng (`++`) và giảm (`--`) trong ngôn ngữ C.

| Lệnh (ATT) | Lệnh (Intel/IDA) | Tương ứng trong C |
| :--- | :--- | :--- |
| `inc D` | `inc D` | `D++` |
| `dec D` | `dec D` | `D--` |

#### 2. Binary Operations (Phép toán hai toán hạng)
Trong nhóm này, toán hạng thứ hai đóng vai trò là **cả nguồn và đích**.
*   **Tương ứng trong C:** Cú pháp này giống với các toán tử gán phức hợp như `x -= y`.
*   **Thứ tự toán hạng (Cực kỳ quan trọng):** 
    *   Trong **ATT (Sách)**: Toán hạng nguồn đứng trước, đích đứng sau. `subq %rax, %rdx` có nghĩa là lấy `%rdx` trừ đi `%rax` rồi lưu lại vào `%rdx`. (Hãy đọc là: "Trừ `%rax` khỏi `%rdx`").
    *   Trong **Intel (IDA Pro)**: Thứ tự đảo ngược. `sub rdx, rax` có nghĩa là `%rdx = %rdx - %rax`.

| Loại toán hạng | ATT Syntax (Sách) | Intel Syntax (IDA) | Logic tính toán |
| :--- | :--- | :--- | :--- |
| **Nguồn (Source)** | Đứng trước | Đứng sau | Có thể là hằng số, thanh ghi, hoặc bộ nhớ. |
| **Đích (Destination)** | Đứng sau | Đứng trước | Có thể là thanh ghi hoặc bộ nhớ. |

**Hạn chế và Cơ chế vận hành:**
1.  **Hạn chế Memory-to-Memory:** Tương tự lệnh `MOV`, cả hai toán hạng không thể cùng là vị trí bộ nhớ.
2.  **Chu kỳ Bộ nhớ:** Khi toán hạng thứ hai là một vị trí bộ nhớ, bộ xử lý phải thực hiện chuỗi thao tác: **Đọc** giá trị từ bộ nhớ -> **Thực hiện** phép toán -> **Ghi** kết quả ngược lại bộ nhớ.

---

### IDA Pro Insights (Lưu ý về phép Trừ và Chia)

*   **Tính không giao hoán (Non-commutative):** Với phép cộng (`add`) hay phép nhân (`imul`), thứ tự toán hạng không làm thay đổi kết quả. Nhưng với phép trừ (`sub`), việc nhầm lẫn giữa ATT và Intel sẽ khiến bạn hiểu sai hoàn toàn logic (đổi ngược số bị trừ và số trừ).
    *   *Sách (ATT):* `sub S, D` $\rightarrow D = D - S$
    *   *IDA (Intel):* `sub D, S` $\rightarrow D = D - S$
*   **Cập nhật biến toàn cục/cục bộ:** Trong IDA, nếu bạn thấy lệnh `add [rbp+var_8], rax`, đây chính là hiện thân của biểu thức `v1 += x;` trong C, trong đó `v1` là một biến cục bộ nằm trên Stack.

---

### Practice Problem 3.8: Tác động của các phép toán Số học

<img width="800" height="619" alt="image" src="https://github.com/user-attachments/assets/401919da-e9d7-478d-b0e5-dce1643347ab" />


**Trạng thái ban đầu hệ thống:**

| Bộ nhớ (Address) | Giá trị (Value) | Thanh ghi (Register) | Giá trị (Value) |
| :--- | :--- | :--- | :--- |
| `0x100` | `0xFF` | `%rax` | `0x100` |
| `0x108` | `0xAB` | `%rcx` | `0x1` |
| `0x110` | `0x13` | `%rdx` | `0x3` |
| `0x118` | `0x11` | | |

---

<details>
<summary><b>Nhấn để xem giải đề và phân tích chi tiết từng lệnh</b></summary>

Điền vào bảng dưới đây vị trí đích (thanh ghi hoặc địa chỉ bộ nhớ) bị cập nhật và giá trị mới sau khi thực hiện lệnh:

| Lệnh (ATT Syntax) | Lệnh (Intel Syntax/IDA) | Logic tính toán | Vị trí đích (Destination) | Giá trị mới (Value) |
| :--- | :--- | :--- | :--- | :--- |
| `addq %rcx, (%rax)` | `add qword ptr [rax], rcx` | Lấy giá trị tại `0x100` (`0xFF`) cộng `%rcx` (`0x1`) | **Address `0x100`** | **`0x100`** |
| `subq %rdx, 8(%rax)` | `sub qword ptr [rax+8], rdx` | Lấy giá trị tại `0x108` (`0xAB`) trừ `%rdx` (`0x3`) | **Address `0x108`** | **`0xA8`** |
| `imulq $16, (%rax,%rdx,8)`| `imul qword ptr [rax+rdx*8], 10h`| Lấy giá trị tại `0x118` (`0x11`) nhân với 16 (`0x10`) | **Address `0x118`** | **`0x110`** |
| `incq 16(%rax)` | `inc qword ptr [rax+10h]` | Tăng giá trị tại `0x110` (`0x13`) thêm 1 | **Address `0x110`** | **`0x14`** |
| `decq %rcx` | `dec rcx` | Giảm thanh ghi `%rcx` (`0x1`) đi 1 | **Thanh ghi `%rcx`** | **`0x0`** |
| `subq %rdx, %rax` | `sub rax, rdx` | Lấy thanh ghi `%rax` (`0x100`) trừ `%rdx` (`0x3`) | **Thanh ghi `%rax`** | **`0xFD`** |

</details>

---

### Phân tích kỹ thuật (Insights cho IDA Pro)

1.  **Thứ tự toán hạng (Reminder):**
    *   Hãy nhìn lệnh `subq %rdx, %rax`. Trong sách (ATT), nó là `%rax = %rax - %rdx`. 
    *   Trong IDA Pro (Intel), bạn sẽ thấy `sub rax, rdx`. May mắn là dù viết ngược lại, logic vẫn là **Đích = Đích - Nguồn**. Tuy nhiên, phải cực kỳ cẩn thận với lệnh `sub` và `cmp`.

2.  **Kích thước dữ liệu:**
    *   Tất cả các lệnh trong bài tập này đều có hậu tố `q` (quadword), tương đương với `qword ptr` trong IDA. Nghĩa là chúng ta đang thao tác trên 8 byte dữ liệu.
    *   Khi tính toán địa chỉ như `8(%rax)`, bạn lấy `0x100 + 8 = 0x108`. Trong IDA, nếu đây là một mảng kiểu `long`, bạn có thể thấy nó hiển thị dưới dạng `[rax + 8]`.

3.  **Phép nhân `imulq $16, ...`**:
    *   Giá trị `0x11` (thập phân là 17). 
    *   $17 \times 16 = 272$.
    *   Đổi $272$ sang Hex: $272 = 256 + 16 = 1 \times 16^2 + 1 \times 16^1 + 0 \times 16^0 = 0x110$.
    *   Đây là cách trình biên dịch xử lý các phép toán số nguyên cơ bản trên các phần tử mảng.

---

### 3.5.3 Shift Operations (Các phép dịch bit)

Nhóm cuối cùng trong các phép toán số học là các phép dịch bit. Trong cú pháp ATT (sách), **lượng dịch (shift amount)** được ghi trước, và **giá trị cần dịch** được ghi sau.

#### 1. Lượng dịch (Shift Amount)
Lượng dịch có thể được chỉ định bằng hai cách:
1.  **Giá trị tức thời (Immediate):** Một hằng số.
2.  **Thanh ghi đơn byte `%cl`:** Đây là một điểm cực kỳ đặc biệt, x86-64 chỉ cho phép sử dụng duy nhất thanh ghi `%cl` (8-bit thấp của `%rcx`) làm toán hạng lượng dịch.

**Cơ chế Masking (Mặt nạ bit):**
Khi dịch một giá trị có độ dài $w$ bits, lượng dịch thực tế được tính từ $m$ bits thấp nhất của thanh ghi `%cl`, trong đó $2^m = w$. Các bit cao hơn bị bỏ qua.
*   **Ví dụ:** Nếu `%cl` có giá trị `0xFF` (255 thập phân):
    *   Lệnh `salb` (8-bit): Dịch 7 vị trí (lấy 3 bit thấp: $2^3=8$).
    *   Lệnh `salw` (16-bit): Dịch 15 vị trí (lấy 4 bit thấp: $2^4=16$).
    *   Lệnh `sall` (32-bit): Dịch 31 vị trí (lấy 5 bit thấp: $2^5=32$).
    *   Lệnh **`salq`** (64-bit): Dịch **63** vị trí (lấy 6 bit thấp: $2^6=64$).

#### 2. Các loại phép dịch

**Dịch trái (Left Shift):**
*   **Lệnh:** **`SAL`** (Shift Arithmetic Left) và **`SHL`** (Shift Logical Left).
*   **Đặc điểm:** Cả hai có hiệu ứng **như nhau**. Các vị trí trống bên phải luôn được lấp đầy bằng số **0**.

**Dịch phải (Right Shift):**
*   **`SAR` (Shift Arithmetic Right):** Dịch phải **số học**. Lấp đầy các vị trí trống bên trái bằng bản sao của **bit dấu** (MSB). Giúp bảo toàn dấu của số nguyên. (Ký hiệu: $>>_A$).
*   **`SHR` (Shift Logical Right):** Dịch phải **logic**. Lấp đầy các vị trí trống bên trái bằng số **0**. Dùng cho dữ liệu không dấu. (Ký hiệu: $>>_L$).

---

### Bảng đối chiếu lệnh (ATT vs Intel/IDA Pro)

| Loại dịch | ATT (Sách) | Intel (IDA Pro) | Hiệu ứng (Logic) |
| :--- | :--- | :--- | :--- |
| **Dịch trái** | `salq $k, D` | `shl D, k` | $D = D \ll k$ |
| **Dịch phải số học** | `sarq %cl, D` | `sar D, cl` | $D = D \gg_A cl$ |
| **Dịch phải logic** | `shrq $k, D` | `shr D, k` | $D = D \gg_L k$ |

---

### IDA Pro Insights (Lưu ý khi dịch ngược)

1.  **Thanh ghi `%cl`:** Nếu bạn thấy một lệnh dịch bit mà lượng dịch là một thanh ghi, IDA chắc chắn sẽ hiển thị đó là `cl`. Đừng ngạc nhiên nếu trước đó chương trình nạp một giá trị lớn vào `rcx`, vì bộ xử lý sẽ chỉ lấy vài bit thấp nhất của nó để thực hiện dịch.
2.  **Phép nhân/chia tối ưu:** 
    *   `shl rax, 3` thường là cách trình biên dịch thực hiện `rax * 8`.
    *   `sar rax, 2` thường là cách thực hiện `rax / 4` (với `rax` là số có dấu).
3.  **Tính có dấu trong C:** 
    *   Trong C, toán tử `>>` trên kiểu `unsigned` sẽ được dịch thành `SHR`.
    *   Toán tử `>>` trên kiểu `int` hoặc `long` (có dấu) thường được dịch thành `SAR`. Đây là điểm quan trọng để bạn xác định kiểu dữ liệu của biến khi đọc mã assembly.

---

### Practice Problem 3.9: Hoàn thiện mã Assembly cho phép dịch bit

<img width="1049" height="820" alt="image" src="https://github.com/user-attachments/assets/eec38aa7-9f0f-47d0-8fef-994c5ea3ced4" />


**Đề bài:**
Giả sử chúng ta muốn tạo mã assembly cho hàm C sau:

```c
long shift_left4_rightn(long x, long n)
{
    x <<= 4;   // Dịch trái 4 bit
    x >>= n;   // Dịch phải n bit (số học)
    return x;
}
```

Dưới đây là một phần của mã assembly thực hiện các phép dịch này và để lại giá trị cuối cùng trong thanh ghi `%rax`. Hai lệnh then chốt đã bị lược bỏ.

**Quy ước:**
*   Tham số `x` nằm trong `%rdi`.
*   Tham số `n` nằm trong `%rsi`.

**Mã Assembly (còn thiếu):**

| Dòng | Mã Assembly (ATT) | Chú thích (Annotations) |
| :--- | :--- | :--- |
| 1 | `movq %rdi, %rax` | Lấy giá trị `x` đưa vào `%rax`. |
| 2 | `________________` | **Thực hiện x <<= 4** |
| 3 | `movl %esi, %ecx` | Lấy giá trị `n` (4 bytes thấp) đưa vào `%ecx`. |
| 4 | `________________` | **Thực hiện x >>= n** (Dịch phải số học) |

---

<details>
<summary><b>Nhấn để xem lời giải và phân tích các lệnh bị thiếu</b></summary>

Để hoàn thiện bài toán này, ta cần chọn đúng lệnh dịch bit dựa trên kích thước dữ liệu (`long` - 8 bytes) và quy ước về thanh ghi lượng dịch.

**1. Điền vào dòng 2 (x <<= 4):**
*   **ATT Syntax:** `salq $4, %rax` (hoặc `shlq $4, %rax`).
*   **Intel Syntax (IDA):** `shl rax, 4`.
*   *Giải thích:* Ta dịch trái thanh ghi chứa giá trị `x` (`%rax`) với hằng số là 4. Hậu tố `q` chỉ định đây là phép toán trên 8 byte.

**2. Điền vào dòng 4 (x >>= n):**
*   **ATT Syntax:** `sarq %cl, %rax`.
*   **Intel Syntax (IDA):** `sar rax, cl`.
*   *Giải thích:* Đề bài yêu cầu dịch phải **số học** (arithmetically), do đó phải dùng lệnh **SAR**. Lượng dịch `n` đã được nạp vào `%ecx` ở dòng 3. Theo quy định của x86-64, lượng dịch khi dùng thanh ghi bắt buộc phải là `%cl`.

**Mã hoàn chỉnh (ATT):**
```assembly
shift_left4_rightn:
    movq    %rdi, %rax      ; Copy x sang rax
    salq    $4, %rax        ; x <<= 4
    movl    %esi, %ecx      ; Copy n sang ecx (lấy phần thấp là cl)
    sarq    %cl, %rax       ; x >>= n (Arithmetic shift)
    ret
```

</details>

---

### IDA Pro Insights (Phân tích thực tế)

*   **Tại sao lại là `movl %esi, %ecx`?** 
    *   Mặc dù `n` là kiểu `long` (8 byte), nhưng lượng dịch bit tối đa cho một thanh ghi 64-bit chỉ là 63. Do đó, trình biên dịch chỉ cần lấy 4 byte thấp (`%esi`) của `n` đưa vào `%ecx`. 
    *   Thực tế, lệnh `sarq %cl, %rax` chỉ quan tâm đến 6 bit thấp nhất của thanh ghi `%rcx` (tức là nằm trong `%cl`).
*   **Nhận diện kiểu dữ liệu:** Khi soi mã trong IDA, nếu bạn thấy lệnh `sar` (Arithmetic Right Shift), bạn có thể tự tin đoán rằng biến trong mã nguồn C là kiểu số nguyên **có dấu** (`int` hoặc `long`). Nếu là `shr` (Logical Right Shift), khả năng cao biến đó là **không dấu** (`unsigned`).

---

### Hình 3.11: Mã C và mã Assembly cho hàm số học `arith`

#### (a) Mã nguồn C
```c
long arith(long x, long y, long z)
{
    long t1 = x ^ y;
    long t2 = z * 48;
    long t3 = t1 & 0x0F0F0F0F;
    long t4 = t2 - t3;
    return t4;
}
```

#### (b) Mã Assembly (Đối chiếu ATT và Intel/IDA Pro)
**Ánh xạ ban đầu:** `x` trong `%rdi`, `y` trong `%rsi`, `z` trong `%rdx`.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Phép tính logic | Trạng thái biến |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `xorq %rsi, %rdi` | `xor rdi, rsi` | `rdi = x ^ y` | `%rdi` giữ `t1` |
| 2 | `leaq (%rdx,%rdx,2), %rax` | `lea rax, [rdx + rdx*2]` | `rax = 3 * z` | `%rax` giữ `3z` |
| 3 | `salq $4, %rax` | `shl rax, 4` | `rax = rax << 4` | `%rax` giữ `t2` ($3z \times 16 = 48z$) |
| 4 | `andl $252645135, %edi` | `and edi, 0F0F0F0Fh` | `edi = t1 & 0x0F0F0F0F` | `%rdi` giữ `t3` |
| 5 | `subq %rdi, %rax` | `sub rax, rdi` | `rax = t2 - t3` | `%rax` giữ `t4` (Kết quả) |
| 6 | `ret` | `retn` | Trả về kết quả | |

*Ghi chú: Số `252645135` trong hệ thập phân chính là `0x0F0F0F0F` trong hệ hexa.*

---

### 3.5.4 Discussion (Thảo luận)

Hầu hết các lệnh số học được liệt kê ở Hình 3.10 có thể sử dụng cho cả số nguyên không dấu (unsigned) và số nguyên bù hai (signed). Chỉ có phép dịch phải (`SAR` vs `SHR`) là cần phân biệt rõ ràng giữa dữ liệu có dấu và không dấu. Đây là lý do tại sao số học bù hai là cách ưu tiên để triển khai số học số nguyên có dấu.

#### Phân tích luồng tối ưu hóa trong ví dụ `arith`:

1.  **Tối ưu phép nhân (Dòng 2 & 3):**
    *   Biểu thức C yêu cầu `z * 48`.
    *   Thay vì dùng lệnh `imul`, trình biên dịch kết hợp `leaq` để tính $3z$ (`z + 2z`), sau đó dùng `salq` để dịch trái 4 bit (tương đương nhân với $2^4 = 16$). 
    *   Kết quả: $3z \times 16 = 48z$. Đây là cách tính toán nhanh hơn nhiều so với lệnh nhân thông thường.

2.  **Tối ưu hóa thanh ghi:**
    *   Trình biên dịch sử dụng chính các thanh ghi tham số để lưu trữ kết quả trung gian. Ví dụ: `%rdi` ban đầu chứa `x`, sau dòng 1 nó chứa `t1`, và sau dòng 4 nó chứa `t3`.
    *   Việc tái sử dụng thanh ghi giúp giảm thiểu việc phải truy cập bộ nhớ (Stack) và tiết kiệm tài nguyên bộ xử lý.

3.  **Sử dụng lệnh 32-bit cho dữ liệu 64-bit (Dòng 4):**
    *   Lưu ý lệnh `andl` hoạt động trên `%edi` (32-bit). 
    *   Vì hằng số `0x0F0F0F0F` chỉ có 32 bit, trình biên dịch sử dụng phiên bản lệnh `l` (long/32-bit). Quy ước của x86-64 sẽ tự động xóa các bit cao của `%rdi` về 0, điều này hoàn toàn phù hợp với logic của phép toán `&` với một hằng số nhỏ.

---

### IDA Pro Insights (Kỹ năng đọc Code tối ưu)

*   **Nhận diện hằng số Hex:** Khi thấy một số thập phân lớn trong sách (như $252645135$), hãy luôn chuyển nó sang Hex trong IDA (nhấn phím `H`). Bạn sẽ thấy nó thường là các mẫu bit đẹp như `0F0F0F0Fh`.
*   **Phân rã phép nhân:** Nếu bạn thấy một lệnh `lea` theo sau bởi một lệnh `shl` trên cùng một thanh ghi, hãy ngay lập tức nghĩ đến một phép nhân hằng số.
    *   Công thức: `(x * Scale_của_LEA) << Lượng_dịch_SHL`.
*   **Thứ tự trả về:** Hãy luôn theo dõi thanh ghi `%rax`. Mọi con đường tính toán cuối cùng đều phải đổ về `%rax` trước lệnh `ret`.

---

### Practice Problem 3.10: Dịch ngược hàm `arith3`

<img width="950" height="715" alt="image" src="https://github.com/user-attachments/assets/967afaa4-e119-401c-b1a9-90ac234c70ee" />


**Đề bài:**
Xét đoạn mã C dưới đây, trong đó các biểu thức đã bị lược bỏ:

```c
short arith3(short x, short y, short z) {
    short p1 = ________;
    short p2 = ________;
    short p3 = ________;
    short p4 = ________;
    return p4;
}
```

**Mã Assembly tương ứng (ATT Syntax):**
*Quy ước: x trong %rdi, y trong %rsi, z trong %rdx*

| Dòng | Lệnh Assembly (Sách) | Lệnh Intel (IDA Pro) | Phân tích logic |
| :--- | :--- | :--- | :--- |
| 1 | `orq %rsi, %rdx` | `or rdx, rsi` | `rdx = z | y` (Đây là `p1`) |
| 2 | `sarq $9, %rdx` | `sar rdx, 9` | `rdx = p1 >> 9` (Đây là `p2`) |
| 3 | `notq %rdx` | `not rdx` | `rdx = ~p2` (Đây là `p3`) |
| 4 | `movq %rdx, %rax` | `mov rax, rdx` | Chuẩn bị trả về giá trị `p3` |
| 5 | `subq %rsi, %rax` | `sub rax, rsi` | `rax = p3 - y` (Đây là `p4`) |
| 6 | `ret` | `retn` | Trả về `p4` |

<details>

<summary>Lời giải mã C hoàn chỉnh:</summary>

```c
short arith3(short x, short y, short z) {
    short p1 = z | y;
    short p2 = p1 >> 9;
    short p3 = ~p2;
    short p4 = p3 - y;
    return p4;
}
```

</details>

---

### Practice Problem 3.11: Idiom "XOR tự thân"

<img width="1072" height="478" alt="image" src="https://github.com/user-attachments/assets/d061eea9-99e5-478a-a4fe-2865393c7b37" />


<details>
   <summary>Nhấn để xem phân tích và lời giải</summary>
Trong mã máy tạo ra từ C, rất phổ biến việc bắt gặp các dòng lệnh có dạng:
`xorq %rcx, %rcx` (ATT) hoặc `xor rcx, rcx` (Intel/IDA)
ngay cả khi mã nguồn C không hề có phép toán XOR nào.

**A. Giải thích tác dụng của lệnh này và phép toán nó thực hiện:**
*   **Tác dụng:** Lệnh này dùng để **đặt giá trị của thanh ghi về 0**. 
*   **Logic:** Theo tính chất của phép toán logic XOR, bất kỳ giá trị nào XOR với chính nó cũng sẽ cho kết quả là 0 ($x \oplus x = 0$).

**B. Cách viết nào trực quan hơn để thực hiện thao tác này?**
*   Cách viết trực quan hơn là dùng lệnh move hằng số 0 vào thanh ghi:
    `movq $0, %rcx` (ATT) hoặc `mov rcx, 0` (Intel/IDA).

**C. So sánh kích thước byte để mã hóa hai cách trên:**
*   **`xorq %rcx, %rcx`**: Chỉ tốn **3 bytes** mã máy.
*   **`movq $0, %rcx`**: Tốn tới **7 bytes** mã máy (vì phải chứa cả hằng số 0 32-bit hoặc 64-bit bên trong lệnh).
*   **Kết luận:** Trình biên dịch sử dụng `xor` thay vì `mov` vì nó giúp file thực thi nhỏ gọn hơn và bộ xử lý thực thi lệnh này cũng cực kỳ nhanh.

</details>

---

### IDA Pro Insights (Mẹo cho người dịch ngược)

1.  **Nhận diện "Zeroing":** Khi bạn thấy bất kỳ lệnh `xor reg, reg` nào trong IDA (ví dụ `xor eax, eax`), hãy ngay lập tức đọc nó là `reg = 0`. Đây là cách chuẩn để xóa thanh ghi trong thế giới mã máy.
2.  **Lệnh 32-bit trong hàm 64-bit:** Ở bài 3.11, trình biên dịch thường dùng `xor eax, eax` (2 bytes) thay vì `xor rax, rax` (3 bytes). Nhờ quy ước *Zero-extension*, lệnh 32-bit này vẫn xóa sạch toàn bộ 64-bit của thanh ghi `rax` về 0, giúp tiết kiệm thêm 1 byte nữa.
3.  **Tối ưu hóa biểu thức (Bài 3.10):** Trình biên dịch có xu hướng sử dụng thanh ghi 64-bit (`orq`, `sarq`) cho các biến `short` (16-bit). Điều này giúp tận dụng tối đa kiến trúc CPU. Khi dịch ngược, nếu thấy `rax` là kết quả cuối cùng của các phép toán trên `rsi`, `rdx`, hãy nhìn vào prototype hàm để xác định kích thước thực tế của biến.

---

### 3.5.5 Special Arithmetic Operations (Các phép toán số học đặc biệt)

Như đã học, việc nhân hai số nguyên 64-bit có thể tạo ra một kết quả cần tới 128-bit để biểu diễn. Kiến trúc x86-64 cung cấp các lệnh hỗ trợ hạn chế cho các con số 128-bit (16 bytes). Intel gọi đại lượng 16-byte này là một **oct word**.

### Hình 3.12: Các phép toán số học đặc biệt (128-bit)

Trong các phép toán này, cặp thanh ghi **`%rdx`** và **`%rax`** được xem như một thanh ghi 128-bit duy nhất (ký hiệu là `rdx:rax`).

| Lệnh (ATT) | Lệnh (Intel/IDA) | Hiệu ứng (Logic) | Mô tả |
| :--- | :--- | :--- | :--- |
| `imulq S` | `imul S` | `rdx:rax = rax * S` | Nhân 128-bit (có dấu) |
| `mulq S` | `mul S` | `rdx:rax = rax * S` | Nhân 128-bit (không dấu) |
| `cqto` | **`cqo`** | `rdx:rax = SignExtend(rax)` | Mở rộng dấu `rax` vào `rdx` |
| `idivq S` | `idiv S` | `rax = rdx:rax / S`<br>`rdx = rdx:rax % S` | Chia 128-bit (có dấu) |
| `divq S` | `div S` | `rax = rdx:rax / S`<br>`rdx = rdx:rax % S` | Chia 128-bit (không dấu) |

---

### Phân tích phép Nhân (Multiplication)

Lệnh nhân `imulq` có hai dạng khác nhau:

1.  **Dạng hai toán hạng (Dạng phổ biến):** Đã học ở Hình 3.10. Nó nhân hai giá trị 64-bit và tạo ra kết quả 64-bit (chấp nhận rủi ro bị tràn số).
2.  **Dạng một toán hạng (Dạng đặc biệt):** 
    *   Sử dụng để tính toán **kết quả đầy đủ 128-bit**.
    *   **Toán hạng ngầm định:** Một thừa số bắt buộc phải nằm sẵn trong thanh ghi **`%rax`**.
    *   **Toán hạng nguồn (`S`):** Là thừa số thứ hai (thanh ghi hoặc bộ nhớ).
    *   **Kết quả:** 64 bit cao được lưu vào **`%rdx`**, 64 bit thấp lưu vào **`%rax`**.

---

### Phân tích phép Chia (Division)

Phép chia trong x86-64 phức tạp hơn vì nó luôn coi số bị chia là một số 128-bit nằm trong cặp thanh ghi `rdx:rax`.

1.  **Chuẩn bị số bị chia:** 
    *   Nếu bạn chỉ có một số 64-bit trong `%rax` và muốn chia nó, bạn phải "lấp đầy" `%rdx`. 
    *   Với số có dấu, ta dùng lệnh **`cqto`** (trong IDA là **`cqo`**) để copy bit dấu của `%rax` vào toàn bộ `%rdx`.
    *   Với số không dấu, ta phải xóa thanh ghi `%rdx` về 0 (ví dụ dùng `xor edx, edx`).
2.  **Thực hiện phép chia:** Lệnh `idivq S` hoặc `divq S` thực hiện chia `rdx:rax` cho toán hạng `S`.
3.  **Kết quả:**
    *   Thương số (Quotient) lưu vào **`%rax`**.
    *   Số dư (Remainder) lưu vào **`%rdx`**.

---

### IDA Pro Insights (Mẹo nhận diện phép toán 128-bit)

*   **Nhận diện phép Chia/Lấy dư:** Khi bạn thấy lệnh `idiv` hoặc `div`, hãy luôn nhìn lên phía trên nó. Nếu có lệnh `cqo` (hoặc `xor edx, edx`), đó là bước chuẩn bị số bị chia. 
    *   Sau lệnh `div`, nếu mã nguồn sử dụng kết quả từ `rax`, đó là phép `/`.
    *   Nếu mã nguồn sử dụng kết quả từ `rdx`, đó là phép `%` (mod).
*   **Phép nhân full:** Nếu bạn thấy lệnh `imul` hoặc `mul` chỉ có **duy nhất một toán hạng**, hãy nhớ ngay quy tắc: kết quả đang bị "xẻ đôi" nằm ở cả `rdx` và `rax`.
*   **Tính nhất quán:** Trình biên dịch thường sử dụng `imul` dạng hai toán hạng cho các phép nhân thông thường (vì nhanh hơn), và chỉ dùng dạng một toán hạng khi thực sự cần xử lý các con số khổng lồ hoặc để kiểm tra tràn số.

---

### Ví dụ về phép nhân 128-bit: Hàm `store_uprod`

Đoạn mã C dưới đây minh họa việc tạo ra kết quả 128-bit từ tích của hai số 64-bit không dấu `x` và `y`. Vì chuẩn C thông thường không quy định kiểu dữ liệu 128-bit, chúng ta sử dụng phần mở rộng của GCC là `__int128`.

#### (a) Mã nguồn C
```c
#include <inttypes.h>

typedef unsigned __int128 uint128_t;

void store_uprod(uint128_t *dest, uint64_t x, uint64_t y) {
    *dest = (uint128_t) x * y;
}
```

#### (b) Phân tích mã Assembly (Ánh xạ ATT và Intel/IDA Pro)
**Quy ước:** `dest` trong `%rdi`, `x` trong `%rsi`, `y` trong `%rdx`.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `movq %rsi, %rax` | `mov rax, rsi` | Copy `x` vào thanh ghi `%rax` (số bị nhân). |
| 2 | `mulq %rdx` | `mul rdx` | Nhân với `y`. Kết quả 128-bit nằm ở `%rdx:%rax`. |
| 3 | `movq %rax, (%rdi)` | `mov [rdi], rax` | Lưu 8 byte thấp của kết quả vào địa chỉ `dest`. |
| 4 | `movq %rdx, 8(%rdi)` | `mov [rdi+8], rdx` | Lưu 8 byte cao của kết quả vào địa chỉ `dest + 8`. |
| 5 | `ret` | `retn` | Trả về. |

**Quan sát kỹ thuật:**
*   **Lưu trữ 128-bit:** Việc lưu trữ kết quả đòi hỏi hai lệnh `movq`: một cho 8 byte bậc thấp và một cho 8 byte bậc cao.
*   **Little-endian:** Do x86-64 là kiến trúc Little-endian, các byte bậc cao được lưu trữ tại địa chỉ bộ nhớ cao hơn (`8(%rdi)`).

---

### Chi tiết về phép Chia (Division)

Phép chia cũng sử dụng cơ chế một toán hạng tương tự như phép nhân. Lệnh chia có dấu `idivq` coi số bị chia là một đại lượng 128-bit nằm trong cặp thanh ghi `%rdx` (64 bit cao) và `%rax` (64 bit thấp).

#### 1. Chuẩn bị số bị chia (Dividend Preparation)
Đối với hầu hết các ứng dụng của phép chia 64-bit, số bị chia ban đầu được đưa vào thanh ghi `%rax`. Lúc này, ta cần thiết lập các bit của `%rdx` sao cho phù hợp:
*   **Phép chia không dấu (`divq`):** Thanh ghi `%rdx` phải được đặt về **0** hoàn toàn.
*   **Phép chia có dấu (`idivq`):** Thanh ghi `%rdx` phải được lấp đầy bởi **bit dấu** của `%rax`.

#### 2. Lệnh `cqto` (Convert Quad to Oct-word)
Để thực hiện việc bù dấu cho phép chia có dấu, x86-64 cung cấp lệnh đặc biệt **`cqto`**.
*   **Cơ chế:** Lệnh này không có toán hạng. nó đọc bit dấu từ `%rax` và sao chép nó ra toàn bộ thanh ghi `%rdx`.
*   **Trong IDA Pro:** Lệnh này thường hiển thị dưới tên gọi **`cqo`**.

#### 3. Kết quả phép toán
Sau khi thực thi lệnh `idivq S` hoặc `divq S`:
*   **Thương số (Quotient):** Được lưu vào thanh ghi **`%rax`**.
*   **Số dư (Remainder):** Được lưu vào thanh ghi **`%rdx`**.

---

### IDA Pro Insights (Kỹ năng nhận diện phép chia)

*   **Dấu hiệu nhận biết:** Khi bạn thấy một lệnh `idiv` hoặc `div` trong IDA, hãy nhìn lên trên:
    1.  Nếu thấy `cqo` (hoặc `cqto`): Đây chắc chắn là phép chia số **có dấu** (`signed /` hoặc `signed %`).
    2.  Nếu thấy `xor edx, edx`: Đây là phép chia số **không dấu** (`unsigned`).
*   **Phân biệt `/` và `%` trong C:**
    *   Nếu sau đó code sử dụng giá trị từ `rax` $\rightarrow$ Mã nguồn C dùng toán tử `/`.
    *   Nếu sau đó code sử dụng giá trị từ `rdx` $\rightarrow$ Mã nguồn C dùng toán tử `%`.

---

### Ví dụ thực tế: Hàm `remdiv` (Tính thương và số dư)

Hàm dưới đây tính toán cả thương số và số dư của hai số nguyên 64-bit có dấu:

#### (a) Mã nguồn C
```c
void remdiv(long x, long y, long *qp, long *rp) {
    long q = x / y;
    long r = x % y;
    *qp = q;
    *rp = r;
}
```

#### (b) Phân tích mã Assembly (Đối chiếu ATT và Intel/IDA Pro)

**Ánh xạ thanh ghi:** 
*   `x` in `%rdi`, `y` in `%rsi`, `qp` in `%rdx`, `rp` in `%rcx`.
*   *Lưu ý:* Vì lệnh `idivq` bắt buộc sử dụng `%rdx`, nên trình biên dịch phải "sơ tán" con trỏ `qp` đang nằm ở `%rdx` đi chỗ khác trước khi chia.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `remdiv:` | `remdiv:` | Nhãn hàm. |
| 2 | `movq %rdx, %r8` | `mov r8, rdx` | **Sơ tán `qp`**: Copy con trỏ `qp` sang `%r8`. |
| 3 | `movq %rdi, %rax` | `mov rax, rdi` | Chuẩn bị số bị chia: Đưa `x` vào `%rax`. |
| 4 | **`cqto`** | **`cqo`** | **Bù dấu**: Mở rộng `%rax` vào `%rdx` (tạo số bị chia 128-bit). |
| 5 | `idivq %rsi` | `idiv rsi` | **Chia cho `y`**: `%rax = x/y`, `%rdx = x%y`. |
| 6 | `movq %rax, (%r8)` | `mov [r8], rax` | Lưu thương số `q` vào địa chỉ `*qp`. |
| 7 | `movq %rdx, (%rcx)` | `mov [rcx], rdx` | Lưu số dư `r` vào địa chỉ `*rp`. |
| 8 | `ret` | `retn` | Trả về. |

---

### Phân tích kỹ thuật (Insights cho IDA Pro)

1.  **Tại sao phải lưu `%rdx`?**
    *   Theo quy ước gọi hàm (ABI), tham số thứ 3 (`qp`) được truyền vào qua `%rdx`. 
    *   Tuy nhiên, lệnh `idivq` ở dòng 5 đòi hỏi cặp `%rdx:%rax` phải làm số bị chia, và sau khi chia xong, nó lại dùng `%rdx` để lưu số dư. 
    *   Do đó, dòng 2 là bắt buộc để không làm mất địa chỉ của con trỏ `qp`.

2.  **Chuẩn bị số bị chia:** 
    *   Dòng 3 và 4 là bộ đôi luôn đi kèm nhau trước lệnh chia có dấu (`idiv`). Lệnh `cqto` (Intel gọi là `cqo`) đảm bảo số 64-bit trong `%rax` được biến thành số 128-bit hợp lệ để chia.

3.  **Hiệu quả kép:**
    *   Một lệnh `idivq` duy nhất thực hiện cả hai phép toán `/` (thương) và `%` (số dư). Đây là lý do tại sao trong C, nếu bạn thực hiện cả chia và lấy dư của cùng hai số cạnh nhau, trình biên dịch sẽ chỉ tạo ra một lệnh chia duy nhất trong mã máy để tối ưu tốc độ.

4.  **Phép chia không dấu:**
    *   Nếu là phép chia không dấu (`unsigned`), trình biên dịch sẽ dùng lệnh **`divq`**. Trước đó, thay vì `cqto`, nó sẽ dùng lệnh `xor %edx, %edx` (hoặc `mov $0, %rdx`) để xóa thanh ghi `%rdx` về 0.

---

### Practice Problem 3.12: Chuyển đổi mã Assembly sang phép chia không dấu

<img width="785" height="344" alt="image" src="https://github.com/user-attachments/assets/587ba30e-b233-488e-9014-23a928c53c82" />


**Đề bài:**
Xét hàm `uremdiv` tính thương và số dư của hai số 64-bit **không dấu**:

```c
void uremdiv(unsigned long x, unsigned long y,
             unsigned long *qp, unsigned long *rp) {
    unsigned long q = x / y;
    unsigned long r = x % y;
    *qp = q;
    *rp = r;
}
```

Hãy sửa đổi mã Assembly của hàm `remdiv` (phép chia có dấu đã học ở trang trước) để thực hiện hàm này.

---

### Lời giải: Mã Assembly cho phép chia không dấu

**Ánh xạ thanh ghi:** `x` (%rdi), `y` (%rsi), `qp` (%rdx), `rp` (%rcx).

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích sự thay đổi |
| :--- | :--- | :--- | :--- |
| 1 | `uremdiv:` | `uremdiv:` | Nhãn hàm. |
| 2 | `movq %rdx, %r8` | `mov r8, rdx` | Sơ tán con trỏ `qp` sang `%r8` (không đổi). |
| 3 | `movq %rdi, %rax` | `mov rax, rdi` | Đưa số bị chia `x` vào `%rax` (không đổi). |
| 4 | **`movq $0, %rdx`** | **`xor edx, edx`** | **Thay đổi:** Xóa sạch `%rdx` về 0 thay vì dùng `cqto`. |
| 5 | **`divq %rsi`** | **`div rsi`** | **Thay đổi:** Dùng lệnh chia không dấu `divq`. |
| 6 | `movq %rax, (%r8)` | `mov [r8], rax` | Lưu thương số (không đổi). |
| 7 | `movq %rdx, (%rcx)` | `mov [rcx], rdx` | Lưu số dư (không đổi). |
| 8 | `ret` | `retn` | Trả về. |

---

### Phân tích sự khác biệt (Insights cho IDA Pro)

1.  **Chuẩn bị số bị chia (Dividend Preparation):**
    *   **Có dấu (`signed`):** Phải dùng lệnh `cqto` (Intel: `cqo`) để mở rộng bit dấu từ `%rax` vào `%rdx`. Nếu không, phép chia sẽ sai nếu `x` là số âm.
    *   **Không dấu (`unsigned`):** Phải đảm bảo `%rdx` bằng **0**. Trình biên dịch thường dùng `xor edx, edx` vì nó ngắn (2 bytes) và nhanh hơn `movq $0, %rdx`.

2.  **Lệnh thực thi:**
    *   **`idivq`**: Dành cho số có dấu.
    *   **`divq`**: Dành cho số không dấu.

3.  **Cách nhận diện kiểu dữ liệu trong IDA:**
    *   Nếu bạn đang phân tích một hàm mà không có mã nguồn, việc thấy lệnh `xor edx, edx` ngay trước lệnh `div` là bằng chứng đanh thép rằng các biến liên quan là **`unsigned`**.
    *   Ngược lại, nếu thấy `cqo` (hoặc `cdq` cho 32-bit) đi kèm với `idiv`, chắc chắn đó là kiểu **có dấu**.

---

## 3.6 Control (Điều khiển luồng)

Cho đến nay, chúng ta mới chỉ xem xét hành vi của **mã đường thẳng (straight-line code)**, nơi các lệnh thực thi nối tiếp nhau theo trình tự. Tuy nhiên, các cấu trúc trong C như:
*   **Conditionals** (Câu lệnh điều kiện: `if`, `else`)
*   **Loops** (Vòng lặp: `while`, `for`, `do-while`)
*   **Switches** (Câu lệnh rẽ nhánh nhiều trường hợp)

Tất cả đều yêu cầu **thực thi có điều kiện (conditional execution)**, nghĩa là trình tự thực thi phụ thuộc vào kết quả của các bài kiểm tra dữ liệu (tests).

### Cơ chế mức máy của cấu trúc điều kiện

Mã máy cung cấp hai cơ chế cơ bản để triển khai hành vi có điều kiện:
1.  **Kiểm tra giá trị dữ liệu:** Thực hiện phép so sánh hoặc kiểm tra bit.
2.  **Thay đổi luồng:** Dựa trên kết quả kiểm tra, bộ xử lý sẽ:
    *   Thay đổi **Control Flow** (Luồng điều khiển): Nhảy đến một vị trí khác trong chương trình.
    *   Thay đổi **Data Flow** (Luồng dữ liệu): Chỉ chuyển dữ liệu khi điều kiện thỏa mãn (Conditional Moves).

---

### Cách luồng điều khiển hoạt động

Thông thường, cả câu lệnh trong C và lệnh mã máy đều được thực thi **tuần tự (sequentially)** theo thứ tự xuất hiện.
*   **Lệnh `jump`:** Là công cụ cốt lõi để thay đổi thứ tự thực thi. Nó chỉ thị cho bộ xử lý chuyển quyền điều khiển sang một phần khác của chương trình, thường là dựa trên kết quả của một phép thử nào đó.
*   **Vai trò của Trình biên dịch:** Phải tạo ra chuỗi các lệnh máy dựa trên các cơ chế cấp thấp này để hiện thực hóa các cấu trúc điều khiển bậc cao của C.

---

### Lộ trình nghiên cứu của Mục 3.6:
1.  Tìm hiểu hai cách triển khai các phép toán có điều kiện.
2.  Mô tả các phương pháp biểu diễn vòng lặp (Loops).
3.  Mô tả cách xử lý câu lệnh `switch`.

---

### IDA Pro Insights (Tầm quan trọng của Control Flow)

*   **Graph View:** Khi bạn nhấn `Space` trong IDA để chuyển sang Graph View, các "Basic Blocks" (khối lệnh đường thẳng) được kết nối với nhau bởi các mũi tên chính là biểu hiện của phần **Control** này.
*   **Mũi tên Xanh lá (Green):** Nhánh thực thi khi điều kiện là **Đúng (True)**.
*   **Mũi tên Đỏ (Red):** Nhánh thực thi khi điều kiện là **Sai (False)**.
*   **Mũi tên Xanh dương (Blue):** Nhánh nhảy không điều kiện (Unconditional jump).

---

## 3.6.1 Condition Codes (Mã điều kiện)

Ngoài các thanh ghi số nguyên, CPU còn duy trì một tập hợp các thanh ghi mã điều kiện 1-bit (thanh ghi Flag) mô tả các đặc tính của phép toán số học hoặc logic gần nhất. Các flag này có thể được kiểm tra để thực hiện các lệnh nhảy có điều kiện.

### Các mã điều kiện hữu dụng nhất:

| Flag | Tên gọi | Ý nghĩa (Khi flag = 1) |
| :--- | :--- | :--- |
| **CF** | **Carry flag** | Phép toán gần nhất gây ra tràn số đối với số **không dấu**. |
| **ZF** | **Zero flag** | Phép toán gần nhất cho kết quả bằng **0**. |
| **SF** | **Sign flag** | Phép toán gần nhất cho kết quả **âm** (Số có dấu). |
| **OF** | **Overflow flag** | Phép toán gần nhất gây ra tràn số bù hai (số **có dấu**). |

### Ví dụ: Phép toán `t = a + b`
Giả sử ta thực hiện lệnh `ADD`. Các flag sẽ được thiết lập tương ứng với các biểu thức logic trong C sau đây:
*   **CF:** `(unsigned) t < (unsigned) a` (Kiểm tra tràn số không dấu).
*   **ZF:** `(t == 0)` (Kết quả bằng 0).
*   **SF:** `(t < 0)` (Kết quả âm).
*   **OF:** `(a < 0 == b < 0) && (t < 0 != a < 0)` (Kiểm tra tràn số có dấu).

---

### Quy tắc thiết lập Flag của các lớp lệnh:

1.  **Lệnh `leaq`:** **KHÔNG** thay đổi bất kỳ mã điều kiện nào, vì nó được dùng để tính toán địa chỉ.
2.  **Các lệnh Logic (XOR, AND, OR):** Flag **CF** và **OF** luôn được đặt về **0**.
3.  **Các lệnh Dịch bit (Shift):** Flag **CF** sẽ nhận giá trị của bit cuối cùng bị dịch ra ngoài, còn **OF** được đặt về **0**.
4.  **Lệnh `INC` và `DEC`:** Thiết lập **ZF**, **SF**, **OF** nhưng giữ nguyên (không thay đổi) **CF**.

---

### Các lệnh So sánh và Kiểm tra (CMP & TEST)

Có hai lớp lệnh (Hình 3.13) chuyên dùng để thiết lập mã điều kiện mà không làm thay đổi bất kỳ thanh ghi đa năng nào khác.

#### Hình 3.13: Lệnh so sánh và kiểm tra

| Lệnh (ATT) | Lệnh (Intel/IDA) | Dựa trên phép toán | Mô tả |
| :--- | :--- | :--- | :--- |
| **CMP** $S_1, S_2$ | `cmp S2, S1` | $S_2 - S_1$ | **So sánh**: Giống lệnh `SUB` nhưng chỉ cập nhật flag, không lưu kết quả trừ. |
| `cmpb`, `cmpw` | `cmp` | | So sánh byte, word... |
| **TEST** $S_1, S_2$ | `test S2, S1` | $S_1  \\&  S_2$ | **Kiểm tra**: Giống lệnh `AND` nhưng chỉ cập nhật flag, không lưu kết quả. |
| `testb`, `testw` | `test` | | Kiểm tra byte, word... |

**Lưu ý về thứ tự toán hạng của CMP (ATT):**
Trong cú pháp ATT, lệnh `cmp $S_1, S_2$` thực hiện phép tính $S_2 - S_1$. Điều này thường gây nhầm lẫn khi đọc:
*   `cmp %rax, %rdx` $\rightarrow$ Đang so sánh `%rdx` với `%rax`.

---

### IDA Pro Insights (Kỹ năng đọc luồng điều khiển)

1.  **Mẫu hình `test rax, rax`:** 
    *   Bạn sẽ thấy mẫu này cực kỳ nhiều trong IDA ngay sau một lời gọi hàm.
    *   **Mục đích:** Kiểm tra xem giá trị trả về trong `rax` có bằng 0 hay không (ví dụ kiểm tra con trỏ NULL).
    *   Nếu `rax` bằng 0, **ZF** sẽ được đặt thành 1.
2.  **Phân biệt CMP và TEST:**
    *   Dùng **CMP** khi muốn so sánh lớn hơn, nhỏ hơn, bằng nhau giữa hai giá trị.
    *   Dùng **TEST** khi muốn kiểm tra một bit cụ thể (Bitwise mask) hoặc kiểm tra một giá trị với chính nó (để xem nó là 0, âm hay dương).
3.  **Flag và rẽ nhánh:** 
    *   IDA Pro sử dụng các flag này để vẽ mũi tên. Ví dụ: Nếu một lệnh `jz` (Jump if Zero) đi sau `test rax, rax`, mũi tên xanh lá sẽ dẫn đến khối lệnh thực thi khi `rax == 0`.

---

## 3.6.2 Accessing the Condition Codes (Truy cập các mã điều kiện)

Thay vì đọc trực tiếp các mã điều kiện (flag), có 3 cách phổ biến để sử dụng chúng:
1.  Thiết lập một byte đơn lẻ thành **0** hoặc **1** tùy thuộc vào sự kết hợp của các mã điều kiện.
2.  Nhảy có điều kiện (Jump) đến một phần khác của chương trình.
3.  Di chuyển dữ liệu có điều kiện (Conditional transfer of data).

### Lớp lệnh SET

Trường hợp đầu tiên sử dụng các lệnh thuộc lớp **`SET`**. Các lệnh này sẽ ghi giá trị **0** hoặc **1** vào một byte đích dựa trên sự kết hợp của các flag trạng thái.

**Lưu ý quan trọng về hậu tố (Suffixes):**
Đối với lớp lệnh `SET`, các hậu tố **KHÔNG** chỉ định kích thước toán hạng mà chỉ định **điều kiện so sánh**. 
*   **Ví dụ:** Lệnh `setl` (Set Less - nhỏ hơn) và `setb` (Set Below - nhỏ hơn/không dấu) hoàn toàn khác nhau về điều kiện kiểm tra, mặc dù cả hai đều chỉ ghi dữ liệu vào 1 byte.
*   **Cảnh báo:** Đừng nhầm lẫn hậu tố `l` trong `setl` là "long word" hay `b` trong `setb` là "byte".

---

### Cách thức hoạt động và Đích đến

*   **Toán hạng đích:** Một lệnh `SET` có thể ghi vào một trong các **thanh ghi đơn byte bậc thấp** (như `%al`, `%bl`,... - xem lại Hình 3.2) hoặc một **vị trí bộ nhớ 1-byte**.
*   **Cơ chế:** Lệnh này chỉ cập nhật 1 byte duy nhất đó thành 0 hoặc 1.
*   **Tạo kết quả 32/64-bit:** Vì lệnh `SET` chỉ tác động lên 1 byte, nên để có được kết quả 32-bit hoặc 64-bit hoàn chỉnh (ví dụ để trả về một giá trị `int` hoặc `long` trong C), ta phải thực hiện thêm bước **xóa các bit bậc cao** (clear high-order bits) của thanh ghi đó.

---

### IDA Pro Insights (Cách nhận diện Boolean trong C)

Khi bạn soi mã trong IDA Pro, các lệnh `SET` thường xuất hiện trong các đoạn mã tính toán logic (ví dụ: `return a < b;`).

| Lệnh (ATT/Sách) | Lệnh (Intel/IDA Pro) | Giải thích thực tế |
| :--- | :--- | :--- |
| `setl %al` | `setl al` | Nếu điều kiện "Nhỏ hơn" thỏa mãn, đặt `al = 1`, ngược lại `al = 0`. |
| `setne %al` | `setnz al` | (Set Not Zero) Đặt `al = 1` nếu kết quả so sánh trước đó là khác nhau. |

**Mẫu hình điển hình (Idiom):**
Trình biên dịch thường kết hợp `SET` với `MOVZX` để xóa các bit thừa:
1.  `cmp rdi, rsi` (So sánh a và b)
2.  `setl al` (Đặt al = 1 nếu a < b)
3.  **`movzx eax, al`** (Xóa 31 bit cao của `eax`, biến `al` thành một số `int` 32-bit hoàn chỉnh).

---

### Ví dụ: Tính toán biểu thức C `a < b`
Giả sử `a` và `b` đều có kiểu `long`. Một chuỗi lệnh điển hình để tính toán biểu thức này diễn ra như sau:

---

### Hình 3.14: Các lệnh SET

<img width="972" height="617" alt="image" src="https://github.com/user-attachments/assets/78799fbb-c9fb-4d7c-a2a0-8229c52ec733" />


Mỗi lệnh này thiết lập một byte đơn lẻ thành **0** hoặc **1** dựa trên sự kết hợp của các mã điều kiện (flag). Một số lệnh có tên đồng nghĩa (Synonyms).

| Lệnh (ATT) | Intel (IDA) | Điều kiện (Flag logic) | Mô tả (Dành cho `CMP S1, S2`) |
| :--- | :--- | :--- | :--- |
| **`sete D`** | `setz` | `ZF` | Bằng nhau / Bằng 0 (Equal / zero) |
| **`setne D`** | `setnz` | `~ZF` | Khác nhau / Khác 0 (Not equal / not zero) |
| **Số có dấu** | | | |
| **`sets D`** | `sets` | `SF` | Kết quả âm (Negative) |
| **`setns D`** | `setns` | `~SF` | Kết quả không âm (Nonnegative) |
| **`setg D`** | `setg` | `~(SF ^ OF) & ~ZF` | Lớn hơn (Greater) |
| **`setge D`** | `setge` | `~(SF ^ OF)` | Lớn hơn hoặc bằng (Greater or equal) |
| **`setl D`** | `setl` | `SF ^ OF` | Nhỏ hơn (Less) |
| **`setle D`** | `setle` | `(SF ^ OF) \| ZF` | Nhỏ hơn hoặc bằng (Less or equal) |
| **Số không dấu**| | | |
| **`seta D`** | `seta` | `~CF & ~ZF` | Lớn hơn (Above) |
| **`setae D`** | `setae` | `~CF` | Lớn hơn hoặc bằng (Above or equal) |
| **`setb D`** | `setb` | `CF` | Nhỏ hơn (Below) |
| **`setbe D`** | `setbe` | `CF \| ZF` | Nhỏ hơn hoặc bằng (Below or equal) |

---

### Ví dụ hàm so sánh: `comp`

Xét hàm C so sánh hai giá trị kiểu `long`:
```c
int comp(long a, long b) {
    return a < b;
}
```

**Mã Assembly tương ứng:**
*Quy ước: a trong %rdi, b trong %rsi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `cmpq %rsi, %rdi` | `cmp rdi, rsi` | So sánh `a` với `b` (Thực hiện `a - b`). |
| 2 | `setl %al` | `setl al` | Đặt `al = 1` nếu `a < b` (có dấu). |
| 3 | `movzbl %al, %eax` | `movzx eax, al` | Xóa các bit cao của `%rax`, kết quả là 0 hoặc 1. |
| 4 | `ret` | `retn` | Trả về kết quả trong `%eax`. |

---

### Phân tích kỹ thuật từ sách:

1.  **Thứ tự toán hạng của `cmpq`:** Mặc dù trong ATT toán hạng được liệt kê là `%rsi` (b) rồi đến `%rdi` (a), nhưng phép so sánh thực tế là **`a` so với `b`**. Đây là điểm gây bối rối nhất khi đọc cú pháp ATT.
2.  **Tên đồng nghĩa (Synonyms):** Trình biên dịch và trình nghịch đảo mã (Disassembler) có thể chọn các tên khác nhau cho cùng một lệnh máy. Ví dụ: `setg` (lớn hơn) và `setnle` (không nhỏ hơn hoặc bằng) là một. IDA Pro thường ưu tiên các tên phổ biến như `setz`, `setnz`.
3.  **Sự khác biệt Signed vs Unsigned:**
    *   Máy tính **không** gán kiểu dữ liệu (có dấu hay không dấu) cho thanh ghi. 
    *   Thay vào đó, nó sử dụng các lệnh khác nhau để thông báo cho CPU cách đọc Flag. 
    *   Ví dụ: Để thực hiện `a < b`, nếu `a, b` có dấu trình biên dịch dùng `setl` (dựa trên `SF` và `OF`). Nếu `a, b` không dấu, nó dùng `setb` (dựa trên `CF`).

---

### IDA Pro Insights (Góc nhìn thực tế)

*   **Tại sao dùng `movzx`?** Lệnh `setl` chỉ ghi vào `al` (8 bit thấp). Nếu không có lệnh `movzx eax, al`, các bit cũ còn sót lại trong `%rax` sẽ làm sai lệch kết quả trả về của hàm (vốn là kiểu `int` 32-bit).
*   **Zero-extension:** Lưu ý rằng lệnh `movzx eax, al` không chỉ xóa các bit cao của `eax` mà còn tự động xóa luôn 32 bit cao nhất của thanh ghi 64-bit `rax` về 0.
*   **Logic toán học của Flag:** IDA giúp bạn không cần nhớ `SF ^ OF`, nhưng bạn nên nhớ:
    *   **`l` (Less)** / **`g` (Greater)**: Dùng cho số **có dấu** (Signed).
    *   **`b` (Below)** / **`a` (Above)**: Dùng cho số **không dấu** (Unsigned).

---

### Practice Problem 3.13: Suy luận kiểu dữ liệu và toán tử so sánh

<img width="965" height="773" alt="image" src="https://github.com/user-attachments/assets/6fefbefb-6fe5-4cb8-8b1d-e8e6deb72848" />


<details>
<summary><b>Nhấn để xem phân tích chi tiết mã nguồn C từ Assembly</b></summary>

**Đề bài:**
Cho hàm C sau:
```c
int comp(data_t a, data_t b) {
    return a COMP b;
}
```
Giả sử `a` nằm trong một phần của `%rdi` và `b` nằm trong một phần của `%rsi`. Dựa trên các chuỗi lệnh assembly dưới đây, hãy xác định kiểu dữ liệu `data_t` và toán tử so sánh `COMP`.

| Câu | Mã Assembly (ATT) | Mã Intel (IDA Pro) | Kiểu dữ liệu (`data_t`) | Toán tử (`COMP`) | Giải thích |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **A** | `cmpl %esi, %edi`<br>`setl %al` | `cmp edi, esi`<br>`setl al` | **`int`** | **`<`** | `cmpl` là 4-byte. `setl` là so sánh nhỏ hơn (có dấu). |
| **B** | `cmpw %si, %di`<br>`setge %al` | `cmp di, si`<br>`setge al` | **`short`** | **`>=`** | `cmpw` là 2-byte. `setge` là lớn hơn hoặc bằng (có dấu). |
| **C** | `cmpb %sil, %dil`<br>`setbe %al` | `cmp dil, sil`<br>`setbe al` | **`unsigned char`** | **`<=`** | `cmpb` là 1-byte. `setbe` là nhỏ hơn hoặc bằng (không dấu - *below or equal*). |
| **D** | `cmpq %rsi, %rdi`<br>`setne %al` | `cmp rdi, rsi`<br>`setne al` | **`long`**, **`unsigned long`**, hoặc **`pointer`** | **`!=`** | `cmpq` là 8-byte. `setne` (khác nhau) dùng chung cho cả có dấu và không dấu. |

</details>

---

### Practice Problem 3.14: Suy luận phép thử với giá trị 0

<img width="832" height="639" alt="image" src="https://github.com/user-attachments/assets/d2c76ce8-4370-4df5-a21d-dd93020a4b01" />

<details>
<summary><b>Nhấn để xem phân tích chi tiết các phép thử logic</b></summary>

**Đề bài:**
Cho hàm C sau:
```c
int test(data_t a) {
    return a TEST 0;
}
```
Giả sử `a` nằm trong một phần của thanh ghi `%rdi`. Hãy xác định các kiểu dữ liệu `data_t` và toán tử so sánh `TEST` tương ứng với mỗi chuỗi lệnh sau.

| Câu | Mã Assembly (ATT) | Mã Intel (IDA Pro) | Kiểu dữ liệu (`data_t`) | Toán tử (`TEST`) | Giải thích |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **A** | `testq %rdi, %rdi`<br>`setge %al` | `test rdi, rdi`<br>`setge al` | **`long`** | **`>=`** | `testq` là 8-byte. `setge` là lớn hơn hoặc bằng (có dấu). |
| **B** | `testw %di, %di`<br>`sete %al` | `test di, di`<br>`sete al` | **`short`** hoặc **`unsigned short`** | **`==`** | `testw` là 2-byte. `sete` (bằng 0) dùng chung cho cả hai loại. |
| **C** | `testb %dil, %dil`<br>`seta %al` | `test dil, dil`<br>`seta al` | **`unsigned char`** | **`>`** | `testb` là 1-byte. `seta` là lớn hơn (không dấu - *above*). |
| **D** | `testl %edi, %edi`<br>`setle %al` | `test edi, edi`<br>`setle al` | **`int`** | **`<=`** | `testl` là 4-byte. `setle` là nhỏ hơn hoặc bằng (có dấu). |

</details>

---

### IDA Pro Insights (Mẹo nhận diện nhanh)

1.  **Hệ lệnh TEST:** Trong IDA, lệnh `test reg, reg` là cách tiêu chuẩn để kiểm tra một giá trị so với số 0.
    *   Nếu theo sau là `setz` (hoặc `jz`): Kiểm tra `if (reg == 0)`.
    *   Nếu theo sau là `setnz` (hoặc `jnz`): Kiểm tra `if (reg != 0)`.
    *   Nếu theo sau là `sets` (hoặc `js`): Kiểm tra `if (reg < 0)` (số có dấu).
2.  **Kích thước thanh ghi:** 
    *   `dil`: 1 byte (`char`).
    *   `di`: 2 bytes (`short`).
    *   `edi`: 4 bytes (`int`).
    *   `rdi`: 8 bytes (`long` hoặc con trỏ).
3.  **Hậu tố lệnh SET:** Hãy luôn nhớ:
    *   **`e` / `ne`**: Dành cho so sánh bằng/khác (không phân biệt dấu).
    *   **`g` / `l` / `ge` / `le`**: Dành cho so sánh **CÓ DẤU**.
    *   **`a` / `b` / `ae` / `be`**: Dành cho so sánh **KHÔNG DẤU**.

---

## 3.6.3 Jump Instructions (Các lệnh nhảy)

Thông thường, các lệnh thực thi nối tiếp nhau theo thứ tự xuất hiện. Lệnh **`jump`** có thể khiến việc thực thi chuyển sang một vị trí hoàn toàn mới trong chương trình. Các đích đến của lệnh nhảy thường được đánh dấu bằng các **nhãn (labels)** trong mã assembly.

### Ví dụ về trình tự thực thi:
```assembly
    movq $0, %rax       ; Thiết lập rax = 0
    jmp .L1             ; Nhảy đến nhãn .L1
    movq (%rax), %rdx   ; Lệnh này bị bỏ qua (nếu chạy sẽ gây lỗi null pointer)
.L1:
    popq %rdx           ; Đích đến của lệnh nhảy
```
Trong ví dụ này, lệnh `jmp .L1` khiến chương trình bỏ qua lệnh `movq` và tiếp tục thực thi tại lệnh `popq`.

---

### Hình 3.15: Các lệnh nhảy (Jump Instructions)

Lệnh nhảy có hai loại: **Không điều kiện** (luôn nhảy) và **Có điều kiện** (chỉ nhảy khi flag thỏa mãn).

| Lệnh (ATT) | Intel (IDA) | Điều kiện nhảy | Mô tả |
| :--- | :--- | :--- | :--- |
| **`jmp Label`** | `jmp Label` | 1 (Luôn nhảy) | Nhảy trực tiếp |
| **`jmp *Op`** | `jmp Op` | 1 (Luôn nhảy) | Nhảy gián tiếp |
| **`je Label`** | `jz` | `ZF` | Bằng nhau / Bằng 0 |
| **`jne Label`** | `jnz` | `~ZF` | Khác nhau / Khác 0 |
| **`js Label`** | `js` | `SF` | Số âm |
| **`jns Label`** | `jns` | `~SF` | Số không âm |
| **Số có dấu** | | | |
| **`jg Label`** | `jg` | `~(SF ^ OF) & ~ZF` | Lớn hơn |
| **`jge Label`** | `jge` | `~(SF ^ OF)` | Lớn hơn hoặc bằng |
| **`jl Label`** | `jl` | `SF ^ OF` | Nhỏ hơn |
| **`jle Label`** | `jle` | `(SF ^ OF) \| ZF` | Nhỏ hơn hoặc bằng |
| **Số không dấu**| | | |
| **`ja Label`** | `ja` | `~CF & ~ZF` | Lớn hơn (Above) |
| **`jae Label`** | `jae` | `~CF` | Lớn hơn hoặc bằng (Above or equal) |
| **`jb Label`** | `jb` | `CF` | Nhỏ hơn (Below) |
| **`jbe Label`** | `jbe` | `CF \| ZF` | Nhỏ hơn hoặc bằng (Below or equal) |

---

### Phân loại lệnh nhảy: Trực tiếp và Gián tiếp

1.  **Direct Jump (Nhảy trực tiếp):**
    *   Đích đến là một nhãn được viết trực tiếp trong mã nguồn (ví dụ: `jmp .L1`).
    *   Trong file thực thi nhị phân, địa chỉ đích thường được mã hóa dưới dạng độ lệch (offset) so với lệnh hiện tại.

2.  **Indirect Jump (Nhảy gián tiếp):**
    *   Đích đến được đọc từ một thanh ghi hoặc một vị trí bộ nhớ.
    *   **Cú pháp ATT:** Sử dụng dấu `*` trước toán hạng (ví dụ: `jmp *%rax` sử dụng giá trị trong `%rax` làm địa chỉ đích; `jmp *(%rax)` đọc địa chỉ đích từ bộ nhớ tại nơi `%rax` trỏ tới).
    *   **Trong IDA Pro:** Bạn sẽ thấy lệnh này phổ biến trong các bảng nhảy (jump tables) của lệnh `switch` hoặc khi gọi các hàm ảo (virtual functions) trong C++.

---

### IDA Pro Insights (Kỹ năng phân tích luồng)

*   **Tên lệnh (Mnemonics):** IDA thường dùng `jz`/`jnz` thay vì `je`/`jne`. Hãy nhớ chúng là một.
*   **Graph View:** Mỗi lệnh nhảy có điều kiện sẽ tạo ra hai nhánh thoát từ một khối lệnh (Basic Block). 
    *   Nhánh nhảy (đúng điều kiện) thường là mũi tên **xanh lá**.
    *   Nhánh không nhảy (chạy tiếp lệnh dưới) là mũi tên **đỏ**.
*   **Nhảy gián tiếp:** Nếu bạn thấy `jmp rax` hoặc `jmp qword ptr [rax+rcx*8]`, đây là dấu hiệu của một cấu trúc điều khiển phức tạp. IDA thường sẽ cố gắng phân tích "Jump Table" để vẽ ra các mũi tên đến từng `case` của lệnh `switch`.
*   **Điều kiện nhảy:** Các lệnh `jCC` (Jump if Condition Code) luôn dựa vào kết quả của lệnh `CMP` hoặc `TEST` ngay trước đó.

---

## 3.6.4 Jump Instruction Encodings (Mã hóa lệnh nhảy)

Trong mã máy, các đích đến của lệnh nhảy (jump targets) được mã hóa theo nhiều cách khác nhau. Phổ biến nhất là **PC-relative** (tương đối so với PC).

### 1. Cơ chế PC-relative
*   **Cách tính:** Mã hóa sự chênh lệch (offset) giữa địa chỉ của **lệnh đích** và địa chỉ của **lệnh ngay sau lệnh nhảy**.
*   **Kích thước offset:** Thường là 1, 2 hoặc 4 bytes.
*   **Lợi ích:** Cho phép mã lệnh có thể thực thi ở bất kỳ đâu trong bộ nhớ mà không cần thay đổi (Position-independent code).

### 2. Cơ chế Absolute Addressing (Địa chỉ tuyệt đối)
*   Sử dụng trực tiếp 4 bytes để chỉ định địa chỉ đích.
*   Trình liên kết (linker) và trình dịch (assembler) sẽ chọn cách mã hóa phù hợp nhất.

---

### Ví dụ phân tích: File `branch.c`

Hãy xem xét đoạn mã assembly và bản disassembly (nghịch đảo mã) tương ứng:

**Mã Assembly gốc (ATT):**
```assembly
1   movq    %rdi, %rax
2   jmp     .L2
3 .L3:
4   sarq    %rax
5 .L2:
6   testq   %rax, %rax
7   jg      .L3
8   rep; ret
```

**Bản Disassembly (File vật thể `.o`):**

| Offset | Bytes (Mã máy) | Lệnh (Intel/IDA) | Giải thích PC-relative |
| :--- | :--- | :--- | :--- |
| `0:` | `48 89 f8` | `mov rax, rdi` | |
| `3:` | **`eb 03`** | `jmp 8 <.L2>` | **Nhảy tiến:** Đích là `0x8`. Lệnh sau jmp là `0x5`. Offset = $8 - 5 = \mathbf{3}$. |
| `5:` | `48 d1 f8` | `sar rax, 1` | |
| `8:` | `48 85 c0` | `test rax, rax` | |
| `b:` | **`7f f8`** | `jg 5 <.L3>` | **Nhảy lùi:** Đích là `0x5`. Lệnh sau jg là `0xd`. Offset = $5 - 13 = \mathbf{-8}$ (`f8` trong Hex). |
| `d:` | `f3 c3` | `rep ret` | |

---

### Aside: Lệnh `rep; ret` có tác dụng gì?
Trong IDA Pro, đôi khi bạn thấy chuỗi `repz retq` hoặc `rep ret`.
*   **Nguồn gốc:** Đây là một "mẹo" dành cho các bộ xử lý AMD cũ. Việc nhảy thẳng đến một lệnh `ret` có thể gây chậm nhịp xử lý (branch prediction penalty). 
*   **Tác dụng:** Lệnh `rep` ở đây đóng vai trò như một lệnh **NOP** (No-operation - không làm gì cả) được chèn vào trước `ret` để tránh lỗi hiệu năng trên.
*   **Kết luận:** Bạn có thể hoàn toàn bỏ qua tiền tố `rep` này khi phân tích logic, nó không làm thay đổi hành vi của lệnh `ret`.

---

### Sự thay đổi sau khi Liên kết (Linking)

Sau khi các file được liên kết thành chương trình thực thi, các địa chỉ ảo được ấn định nhưng **giá trị mã hóa PC-relative vẫn giữ nguyên**.

**Bản Disassembly của chương trình đã liên kết:**
```assembly
4004d0: 48 89 f8        mov    rax, rdi
4004d3: eb 03           jmp    4004d8 <loop+0x8>
4004d5: 48 d1 f8        sar    rax, 1
4004d8: 48 85 c0        test   rax, rax
4004db: 7f f8           jg     4004d3 <loop+0x5>
4004dd: f3 c3           repz retq
```

*   **Quan sát:** Mặc dù địa chỉ bắt đầu là `4004d0`, nhưng các byte mã máy của lệnh nhảy (`eb 03` và `7f f8`) hoàn toàn không đổi so với file `.o`.
*   **Tính toán lại:** Tại địa chỉ `4004db`, lệnh `jg` nhảy về `4004d3`. Lệnh ngay sau `jg` nằm ở `4004dd`.
    *   $4004d3 - 4004dd = -10$ (thập phân) $\rightarrow$ À, trong ví dụ này sách tính toán offset dựa trên độ dài lệnh cụ thể. Kết quả cuối cùng vẫn là giá trị bù hai được mã hóa vào lệnh.

---

### IDA Pro Insights (Cách đọc Opcode)

1.  **Nhận diện Jump ngắn:** Nếu bạn thấy Opcode bắt đầu bằng `eb` (JMP), `74` (JZ), `75` (JNZ),... đó là các lệnh nhảy ngắn (Short Jumps) sử dụng offset 1-byte. Khoảng cách nhảy tối đa là khoảng 128 bytes.
2.  **Tính toán địa chỉ đích:** IDA tự động tính toán địa chỉ đích và hiển thị nhãn hoặc địa chỉ thực cho bạn. Tuy nhiên, khi bạn patch code (sửa mã máy), bạn phải tự tính toán giá trị PC-relative này.
3.  **Quy tắc ngón tay cái:** 
    *   Offset dương: Nhảy tiến (xuống dưới).
    *   Offset âm (bắt đầu bằng `F`, `E`,...): Nhảy lùi (lên trên) - thường thấy trong các vòng lặp.

---

### Practice Problem 3.15: Tính toán địa chỉ đích của lệnh nhảy (PC-relative)

<img width="857" height="670" alt="image" src="https://github.com/user-attachments/assets/218bf985-ea39-4362-aac4-0cebe655607f" />


**Quy tắc cốt lõi:**
Trong cơ chế PC-relative, địa chỉ đích (Target Address) được tính theo công thức:
$$\text{Target} = \text{Địa chỉ của lệnh NGAY SAU lệnh nhảy} + \text{Giá trị Offset}$$

---

#### Câu A: Nhảy tiến (Positive Offset)
*   **Mã máy:** `4003fa: 74 02` (`je XXXXXX`)
*   **Lệnh tiếp theo:** `4003fc: ff d0` (`callq *%rax`)
*   **Offset:** `02` (hệ 16)
*   **Tính toán:** $\text{Target} = 0x4003fc + 0x02 = \mathbf{0x4003fe}$

#### Câu B: Nhảy lùi (Negative Offset - 1 byte)
*   **Mã máy:** `40042f: 74 f4` (`je XXXXXX`)
*   **Lệnh tiếp theo:** `400431: 5d` (`pop %rbp`)
*   **Offset:** `f4` (8-bit signed hex).
    *   *Cách đổi nhanh:* `f4` tương đương với $-12$ trong hệ thập phân (vì $0x100 - 0xf4 = 0x0c = 12$).
*   **Tính toán:** $\text{Target} = 0x400431 - 0x0c = \mathbf{0x400425}$

#### Câu C: Tìm địa chỉ của lệnh (Reversing the calculation)
*   **Lệnh nhảy:** `XXXXXX: 77 02` (`ja 400547`)
*   **Lệnh tiếp theo:** `XXXXXX: 5d` (`pop %rbp`)
*   **Offset:** `02`. **Đích:** `0x400547`.
*   **Tính toán:**
    1.  Địa chỉ lệnh tiếp theo (pop) = $\text{Target} - \text{Offset} = 0x400547 - 0x02 = \mathbf{0x400545}$
    2.  Địa chỉ lệnh nhảy (ja) = $\text{Địa chỉ pop} - \text{Kích thước lệnh ja} (2 \text{ bytes}) = 0x400545 - 2 = \mathbf{0x400543}$

#### Câu D: Nhảy lùi với Offset 4-byte (Little-endian)
*   **Mã máy:** `4005e8: e9 73 ff ff ff` (`jmpq XXXXXX`)
*   **Lệnh tiếp theo:** `4005ed: 90` (`nop`)
*   **Offset (4 bytes):** `73 ff ff ff`
    *   *Đọc theo Little-endian:* `0xffffff73` (giá trị bù hai 32-bit).
    *   *Giá trị thập phân:* $0xffffff73$ tương đương $-141$ ($0x100000000 - 0xffffff73 = 0x8d = 141$).
*   **Tính toán:** $\text{Target} = 0x4005ed - 141$ (đổi 141 sang hex là $0x8d$).
    *   $0x4005ed - 0x8d = \mathbf{0x400560}$

---

### IDA Pro Insights (Kỹ năng Patching & Debugging)

1.  **Hex View vs. Disassembly:** IDA Pro luôn tự tính toán các địa chỉ này và hiển thị tên nhãn hoặc địa chỉ thực cho bạn. Tuy nhiên, nếu bạn dùng tính năng **Patch Program** để đổi một lệnh `jz` thành `jnz`, IDA sẽ chỉ cho bạn sửa các byte (Opcode). Việc hiểu cách tính offset giúp bạn biết mình đang nhảy đi đâu.
2.  **Short Jumps vs. Near Jumps:**
    *   Câu A, B, C sử dụng lệnh nhảy 2-byte (1 byte Opcode `74`/`77` + 1 byte Offset). Đây là **Short Jump**.
    *   Câu D sử dụng lệnh 5-byte (Opcode `e9` + 4 bytes Offset). Đây là **Near Jump**, cho phép nhảy đến bất kỳ đâu trong phạm vi $\pm 2GB$.
3.  **Mẹo tính nhanh offset âm:**
    *   Trong các file thực thi, offset âm luôn bắt đầu bằng nhiều chữ `f` (ví dụ `f4`, `ffffff73`).
    *   Khi bạn thấy mũi tên trong IDA nhảy ngược lên phía trên, hãy nhìn vào Hex View, bạn sẽ thấy các byte bắt đầu bằng `f`.

---

## 3.6.5 Implementing Conditional Branches with Conditional Control

Cách phổ biến nhất để chuyển đổi các biểu thức và câu lệnh điều kiện từ C sang mã máy là sử dụng sự kết hợp giữa **nhảy có điều kiện (conditional jumps)** và **nhảy không điều kiện (unconditional jumps)**.

### Ví dụ minh họa: Hàm `absdiff_se`
Hàm này tính giá trị tuyệt đối của hiệu hai số $x$ và $y$, đồng thời tăng giá trị của một trong hai biến toàn cục `lt_cnt` hoặc `ge_cnt`.

#### 1. Các phiên bản mã nguồn:

<table style="width:100%; border-collapse:collapse; table-layout:fixed; font-family:Arial, sans-serif;">
  <thead>
    <tr>
      <th style="background:#2c3e50; color:#fff; padding:10px; text-align:left;">(a) Original C code</th>
      <th style="background:#2c3e50; color:#fff; padding:10px; text-align:left;">(b) Equivalent Goto version</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="vertical-align:top; border:1px solid #ddd;">
        <pre style="margin:0; padding:12px; background:#f6f8fa; overflow-x:auto; white-space: pre;">            
            <code>
        void absdiff_se(long x, long y) {
             if (x &lt; y) {
                 lt_cnt++;
                 result = y - x;
             } else {
                 ge_cnt++;
                 result = x - y;
             }
             return result;
         }
            </code>
           </pre>
      </td>
      <td style="vertical-align:top; border:1px solid #ddd;">
        <pre style="margin:0; padding:12px; background:#f6f8fa; overflow-x:auto; white-space: pre;">
           <code>
              <br>
        void absdiff_se_goto(long x, long y) {
                if (x &gt;= y)
                    goto x_ge_y;
                lt_cnt++;
                result = y - x;
                return result;
            x_ge_y:
                ge_cnt++;
                result = x - y;
                return result;
            }
      </code>
   </pre>
      </td>
    </tr>
  </tbody>
</table>

**Giải thích:** Phiên bản (b) sử dụng lệnh `goto` để mô phỏng chính xác cách mà luồng điều khiển của mã máy vận hành. Việc sử dụng `goto` thường bị coi là phong cách lập trình tồi, nhưng ở đây nó giúp ta hiểu cách trình biên dịch "phẳng hóa" cấu trúc `if-else`.

#### 2. Phân tích mã Assembly (Generated Assembly code):
*Quy ước: x trong %rdi, y trong %rsi*

<img width="619" height="374" alt="image" src="https://github.com/user-attachments/assets/3bea973a-bdc6-4eb9-b539-0f576548fe67" />


| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `cmpq %rsi, %rdi` | `cmp rdi, rsi` | So sánh `x` và `y`. |
| 2 | `jge .L2` | `jge short loc_L2` | **Nếu x >= y**: Nhảy đến nhãn `.L2` (nhánh Else). |
| 3 | `addq $1, lt_cnt(%rip)` | `add lt_cnt[rip], 1` | `lt_cnt++` (nhánh Then). |
| 4 | `movq %rsi, %rax` | `mov rax, rsi` | `result = y` |
| 5 | `subq %rdi, %rax` | `sub rax, rdi` | `result = y - x` |
| 6 | `ret` | `retn` | Trả về. |
| 7 | `.L2:` | `loc_L2:` | Nhãn nhánh Else. |
| 8 | `addq $1, ge_cnt(%rip)` | `add ge_cnt[rip], 1` | `ge_cnt++` |
| 9 | `movq %rdi, %rax` | `mov rax, rdi` | `result = x` |
| 10 | `subq %rsi, %rax` | `sub rax, rsi` | `result = x - y` |
| 11 | `ret` | `retn` | Trả về. |

---

### Aside: Mô tả mã máy bằng ngôn ngữ C
Trình biên dịch tạo ra các khối mã riêng biệt cho nhánh "then" và nhánh "else". Nó chèn các lệnh nhảy có điều kiện và không điều kiện để đảm bảo chỉ có khối mã đúng được thực thi. Việc tác giả sử dụng phiên bản `goto` là để tạo ra một cầu nối logic giúp người học dễ dàng ánh xạ (mapping) giữa mã nguồn bậc cao và mã máy cấp thấp.

---

### Khuôn mẫu chung (Template) cho cấu trúc If-Else

Dạng tổng quát của một câu lệnh `if-else` trong C là:
```c
if (test-expr)
    then-statement
else
    else-statement
```

Trình biên dịch thường chuyển đổi cấu trúc này sang dạng mã máy tương đương với logic `goto` sau:

```c
    t = test-expr;
    if (!t) 
        goto false;
    then-statement
    goto done;
false:
    else-statement
done:
```

**Phân tích luồng thực thi:**
1.  **`test-expr`**: Được tính toán và thiết lập các mã điều kiện (flag).
2.  **`if (!t) goto false`**: Nếu điều kiện sai, bộ xử lý nhảy qua nhánh `then` để thực hiện nhánh `else`.
3.  **`then-statement`**: Được thực thi nếu điều kiện đúng.
4.  **`goto done`**: Sau khi xong nhánh `then`, phải có một lệnh nhảy không điều kiện để bỏ qua nhánh `else`.

---

### IDA Pro Insights (Góc nhìn sơ đồ khối)

*   **Cấu trúc hình thoi:** Trong Graph View của IDA, cấu trúc `if-else` thường tạo thành một hình thoi hoặc hình chữ Y.
*   **Mũi tên đỏ (False/Jump):** Tương ứng với lệnh `jge .L2` ở dòng 2. Nếu điều kiện so sánh `x < y` bị sai (tức là $x \geq y$), IDA sẽ vẽ mũi tên màu đỏ dẫn bạn đến khối lệnh `loc_L2`.
*   **Mũi tên xanh lá (True/Fall-through):** Nếu điều kiện đúng, chương trình chạy tiếp xuống dòng lệnh ngay dưới (`lt_cnt++`).
*   **Điểm hội tụ (Done):** Cả hai nhánh cuối cùng thường sẽ hội tụ về một lệnh `ret` hoặc một khối lệnh chung phía sau (nhãn `done`). Tuy nhiên, trong ví dụ `absdiff_se`, trình biên dịch tối ưu bằng cách đặt lệnh `ret` ở cuối mỗi nhánh để thoát hàm nhanh nhất có thể.

---

### Practice Problem 3.16: Phân tích cơ chế Ngắt mạch (Short-circuit)

<img width="941" height="741" alt="image" src="https://github.com/user-attachments/assets/3337786a-ffcd-4417-9809-9e2aa79cca6b" />


**Đề bài:**
Cho hàm C sử dụng toán tử logic `&&`:
```c
void cond(short a, short *p) {
    if (a && *p < a)
        *p = a;
}
```

**Mã Assembly (Đối chiếu ATT và Intel/IDA Pro):**
*Quy ước: a trong %rdi, p trong %rsi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `testq %rdi, %rdi` | `test rdi, rdi` | Kiểm tra `a` có bằng 0 không. |
| 2 | `je .L1` | `jz short loc_L1` | **Nếu a == 0**: Nhảy đến kết thúc (Ngắt mạch). |
| 3 | `cmpw %di, (%rsi)` | `cmp [rsi], di` | So sánh `*p` với `a`. |
| 4 | `jge .L1` | `jge short loc_L1` | **Nếu *p >= a**: Nhảy đến kết thúc. |
| 5 | `movw %di, (%rsi)` | `mov [rsi], di` | Thực hiện `*p = a`. |
| 6 | `.L1: rep; ret` | `loc_L1: retn` | Kết thúc hàm. |

**A. Viết phiên bản `goto` mô phỏng luồng điều khiển trên:**
```c
void cond_goto(short a, short *p) {
    if (a == 0) 
        goto done;
    if (*p >= a) 
        goto done;
    *p = a;
done:
    return;
}
```

**B. Tại sao mã Assembly có 2 nhánh điều kiện dù mã C chỉ có 1 lệnh `if`?**
*   **Lý do:** Do cơ chế **Short-circuit evaluation (đánh giá ngắt mạch)** của toán tử `&&`. 
*   Trong biểu thức `(a && *p < a)`, nếu vế đầu tiên (`a`) là sai (bằng 0), toàn bộ biểu thức chắc chắn sai, nên vế sau (`*p < a`) sẽ **không bao giờ được thực thi**. 
*   Trình biên dịch hiện thực hóa điều này bằng cách tạo ra hai lần kiểm tra và hai lệnh nhảy khác nhau.

---

### Practice Problem 3.17: Quy tắc thay thế cho cấu trúc If-Else

<img width="965" height="467" alt="image" src="https://github.com/user-attachments/assets/c0ff845a-3a0e-4706-9096-84c44d67e3aa" />


**Đề bài:**
Xét một quy tắc thay thế (Alternate rule) để dịch `if-else` sang `goto` như sau:
```c
    t = test-expr;
    if (t)
        goto true;
    else-statement
    goto done;
true:
    then-statement
done:
```

**A. Viết lại phiên bản `goto` của hàm `absdiff_se` dựa trên quy tắc này:**
```c
long absdiff_se_alt(long x, long y) {
    long result;
    if (x < y) 
        goto true;
    ge_cnt++;
    result = x - y;
    goto done;
true:
    lt_cnt++;
    result = y - x;
done:
    return result;
}
```

**B. Tại sao trình biên dịch lại chọn quy tắc này thay vì quy tắc khác?**
*   **Tính hiệu quả của Cache và Nhánh (Branch Prediction):** Việc chọn quy tắc nào thường phụ thuộc vào việc trình biên dịch dự đoán nhánh nào (`then` hay `else`) có khả năng xảy ra thường xuyên hơn.
*   Quy tắc gốc (dịch ngược điều kiện) giúp code chạy thẳng (fall-through) vào nhánh `then`, trong khi quy tắc thay thế này chạy thẳng vào nhánh `else`. 
*   Mục tiêu là để các lệnh thực thi thường xuyên nhất nằm liên tục trong bộ nhớ, giúp tối ưu hóa việc nạp lệnh của CPU.

---

### IDA Pro Insights (Kỹ năng thực tế)

1.  **Nhận diện `&&` và `||`:** 
    *   Trong IDA, nếu bạn thấy 2 hoặc nhiều lệnh `cmp/test` và `jCC` liên tiếp dẫn đến cùng một địa chỉ "thoát", đó chính là biểu hiện của **Short-circuit logic**. 
    *   Với `&&`, các lệnh nhảy sẽ dẫn đến nhánh **False**.
    *   Với `||`, các lệnh nhảy sẽ dẫn đến nhánh **True**.
2.  **Lệnh `rep; ret`:** Như đã đề cập ở trang trước, IDA thường hiển thị là `retn`. Nếu bạn thấy `f3 c3` trong Hex View, hãy hiểu đơn giản đó là lệnh `return`.
3.  **Kích thước dữ liệu (Problem 3.16):** Lưu ý lệnh `cmpw` và `movw`. Chữ `w` (word) cho thấy chúng ta đang làm việc với kiểu `short` (16-bit). Trong IDA, bạn sẽ thấy `di` (16-bit) thay vì `rdi` (64-bit).

---

### Practice Problem 3.18: Dịch ngược cấu trúc If-Else lồng nhau

<img width="555" height="739" alt="image" src="https://github.com/user-attachments/assets/6d8baddb-ad73-47f3-a991-c5c9114f37b1" />


**Đề bài:** Hoàn thiện các biểu thức còn thiếu trong mã nguồn C dựa trên mã Assembly được cung cấp.

#### 1. Phân tích luồng khởi tạo và tham số:
*   `x` in `%rdi`, `y` in `%rsi`, `z` in `%rdx`.
*   Lệnh 1-2:
    *   `leaq (%rdx,%rsi), %rax`: `%rax = z + y`
    *   `subq %rdi, %rax`: `%rax = (z + y) - x`
*   **Suy luận:** Biến `val` được khởi tạo bằng `z + y - x`.

#### 2. Phân tích các nhánh rẽ (Control Flow):

| Nhánh Assembly | Lệnh kiểm tra & Nhảy | Logic C tương ứng | Kết quả xử lý |
| :--- | :--- | :--- | :--- |
| **Gốc** | `cmpq $5, %rdx`<br>`jle .L2` | `if (z > 5)` | Nếu `z <= 5` nhảy đến `.L2` (nhánh `else if`). |
| **Trong If (1)** | `cmpq $2, %rsi`<br>`jle .L3` | `if (y > 2)` | Nếu `y > 2`, chạy tiếp. Nếu `y <= 2` nhảy đến `.L3`. |
| **Dưới If (1)** | `movq %rdi, %rax`<br>`idivq %rdx` | `val = x / z` | Thực hiện phép chia `x / z` rồi `ret`. |
| **Nhãn .L3** | `movq %rdi, %rax`<br>`idivq %rsi` | `else { val = x / y }` | (Nhánh `else` của `y > 2`). Thực hiện `x / y` rồi `ret`. |
| **Nhãn .L2** | `cmpq $3, %rdx`<br>`jge .L4` | `else if (z < 3)` | Nếu `z >= 3` nhảy đến `.L4` (kết thúc). |
| **Dưới .L2** | `movq %rdx, %rax`<br>`idivq %rsi` | `val = z / y` | (Nhánh `z < 3`). Thực hiện `z / y`. |
| **Nhãn .L4** | `rep; ret` | `return val` | Giữ nguyên giá trị `val` ban đầu (`z + y - x`). |

---

### Lời giải mã C hoàn chỉnh:

```c
short test(short x, short y, short z) {
    short val = z + y - x; // Lệnh leaq và subq đầu hàm
    if (z > 5) {           // Lệnh cmpq $5, %rdx và jle .L2
        if (y > 2)         // Lệnh cmpq $2, %rsi và jle .L3
            val = x / z;   // Lệnh idivq %rdx
        else
            val = x / y;   // Nhãn .L3 và idivq %rsi
    } else if (z < 3) {    // Nhãn .L2, cmpq $3, %rdx và jge .L4
        val = z / y;       // Lệnh idivq %rsi dưới nhãn .L2
    }
    return val;            // Nhãn .L4 trả về %rax
}
```

---

### IDA Pro Insights (Kỹ năng dịch ngược cấu trúc phức tạp)

1.  **Nhận diện Biến cục bộ vs Tham số:**
    *   Trong IDA, bạn sẽ thấy `%rdi`, `%rsi`, `%rdx` được ký hiệu là các tham số hàm.
    *   Việc `%rax` được tính toán ngay từ đầu (`leaq`, `subq`) rồi sau đó bị ghi đè trong các nhánh `if` là dấu hiệu của một biến được khởi tạo mặc định, sau đó cập nhật theo điều kiện.

2.  **Cấu trúc lồng nhau trong Graph View:**
    *   Khối lệnh đầu tiên sẽ có 2 mũi tên: Mũi tên đỏ dẫn đến `.L2`, mũi tên xanh dẫn đến khối kiểm tra `y`.
    *   Tại khối kiểm tra `y`, lại có 2 mũi tên dẫn đến 2 phép chia khác nhau.
    *   Sự hội tụ: Lưu ý nhãn `.L4`. Nó là điểm hội tụ cuối cùng của nhánh `else if (z < 3)` nhưng lại không chứa lệnh tính toán nào. Điều này có nghĩa là nó giữ lại kết quả đã có sẵn trong `%rax` từ trước.

3.  **Phép chia (`idivq`):**
    *   Như đã học ở chương trước, trước `idivq` thường phải có bước chuẩn bị số bị chia. Ở đây mã assembly được viết rút gọn (lược bỏ `cqto`), nhưng trong IDA thực tế, bạn sẽ thấy `cqo` xuất hiện liên tục trước mỗi lệnh `idiv`.

---

## 3.6.6 Implementing Conditional Branches with Conditional Moves

Cách truyền thống để thực hiện các phép toán có điều kiện là thông qua việc **chuyển giao điều khiển (conditional transfer of control)**, nơi chương trình chọn một luồng thực thi dựa trên điều kiện. Cơ chế này đơn giản nhưng có thể rất kém hiệu quả trên các bộ xử lý hiện đại.

### Chiến lược thay đổi dữ liệu (Conditional transfer of data)

Một chiến lược thay thế là **chuyển giao dữ liệu có điều kiện**. Cách tiếp cận này tính toán trước kết quả của cả hai nhánh điều kiện, sau đó mới chọn một kết quả dựa trên việc điều kiện có thỏa mãn hay không. 
*   **Lợi ích:** Chiến lược này có thể được triển khai bằng một lệnh **conditional move** đơn giản, phù hợp hơn với đặc tính hiệu năng của các bộ xử lý hiện đại (vốn dựa trên cơ chế pipeline và dự đoán nhánh).

---

### Ví dụ minh họa: Hàm `absdiff`

Hàm này tính giá trị tuyệt đối của hiệu hai số $x$ và $y$ ($|x - y|$). 

#### (a) Mã nguồn C gốc
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

#### (b) Mô phỏng logic bằng phép gán có điều kiện (Cmov version)
Trình biên dịch sẽ biến đổi logic trên thành dạng tính toán song song:
```c
long cmovdiff(long x, long y) {
    long rval = y - x;  // Tính sẵn kết quả nhánh then
    long eval = x - y;  // Tính sẵn kết quả nhánh else
    int ntest = x >= y; // Kiểm tra điều kiện ngược (x < y)
    if (ntest) rval = eval; // Nếu ntest đúng (x >= y), chọn kết quả eval
    return rval;
}
```

---

#### (c) Mã Assembly (Ánh xạ ATT và Intel/IDA Pro)
*Quy ước: x trong %rdi, y trong %rsi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `movq %rsi, %rax` | `mov rax, rsi` | `rax = y` |
| 2 | `subq %rdi, %rax` | `sub rax, rdi` | `rval = y - x` |
| 3 | `movq %rdi, %rdx` | `mov rdx, rdi` | `rdx = x` |
| 4 | `subq %rsi, %rdx` | `sub rdx, rsi` | `eval = x - y` |
| 5 | `cmpq %rsi, %rdi` | `cmp rdi, rsi` | So sánh `x` và `y` (x : y) |
| 6 | **`cmovge %rdx, %rax`** | **`cmovge rax, rdx`** | **Nếu x >= y**: Gán `rax = rdx`. |
| 7 | `ret` | `retn` | Trả về kết quả trong `%rax`. |

---

### Phân tích kỹ thuật: Tại sao Conditional Move lại nhanh hơn?

1.  **Loại bỏ lệnh nhảy:** Trong mã assembly trên, hoàn toàn không có lệnh `jmp` hay `jCC`. Chương trình chạy một lèo từ dòng 1 đến dòng 7.
2.  **Pipeline:** Các bộ xử lý hiện đại thực hiện nhiều lệnh cùng lúc bằng cách nạp trước các lệnh tiếp theo vào "đường ống" (pipeline). 
    *   Với lệnh nhảy (`jCC`), nếu bộ xử lý dự đoán sai nhánh sẽ đi, nó phải xóa sạch pipeline và nạp lại từ đầu, gây lãng phí thời gian (khoảng 15-30 chu kỳ máy).
    *   Với `cmov`, bộ xử lý cứ thế thực hiện tất cả các lệnh tính toán. Lệnh `cmov` cuối cùng chỉ là một phép chọn dữ liệu đơn giản, không làm gián đoạn luồng nạp lệnh.

---

### IDA Pro Insights (Nhận diện tối ưu hóa luồng)

*   **Graph View:** Thay vì thấy các khối lệnh rẽ nhánh với mũi tên xanh/đỏ phức tạp, bạn sẽ thấy một khối lệnh hình chữ nhật duy nhất (đường thẳng). Đây là dấu hiệu hàm đã được tối ưu hóa bằng `cmov`.
*   **Lớp lệnh `cmovCC`:** Tương tự như `setCC` và `jCC`, lớp lệnh này có nhiều biến thể:
    *   `cmove` (bằng), `cmovne` (khác).
    *   `cmovl` (nhỏ hơn), `cmovge` (lớn hơn hoặc bằng)...
*   **Chi phí:** Mặc dù `cmov` tránh được lỗi dự đoán nhánh, nhưng nó bắt buộc bộ xử lý phải thực hiện **tất cả** các phép tính của cả hai nhánh. Trình biên dịch sẽ chỉ dùng `cmov` khi các phép tính này đơn giản (như phép trừ trong ví dụ trên). Nếu hai nhánh chứa các phép toán phức tạp hoặc tốn thời gian, lệnh nhảy (`if-else` truyền thống) vẫn sẽ được ưu tiên.

---

### Phân tích hiệu năng: Tại sao chiến lược thay đổi dữ liệu lại nhanh hơn?

Để hiểu tại sao mã dựa trên việc di chuyển dữ liệu có điều kiện lại có thể vượt qua mã dựa trên việc chuyển giao điều khiển, chúng ta phải hiểu cách thức hoạt động của các bộ xử lý hiện đại.

#### 1. Cơ chế Pipelining (Đường ống)
Các bộ xử lý đạt được hiệu năng cao thông qua **pipelining**, nơi việc thực thi một lệnh được chia thành một chuỗi các giai đoạn (như nạp lệnh từ bộ nhớ, xác định loại lệnh, đọc bộ nhớ, thực hiện phép toán số học, ghi vào bộ nhớ, cập nhật PC). 
*   **Cách thức:** Bộ xử lý đạt hiệu suất cao bằng cách chồng chéo các giai đoạn của các lệnh liên tiếp (ví dụ: nạp lệnh thứ 3 trong khi đang tính toán cho lệnh thứ 2 và đang ghi kết quả cho lệnh thứ 1).
*   **Điều kiện lý tưởng:** Đường ống phải luôn chứa đầy các lệnh sắp được thực thi.

#### 2. Thách thức từ lệnh Nhảy (Conditional Jumps)
Khi bộ xử lý gặp một lệnh nhảy có điều kiện (thường gọi là **"branch"**), nó không thể xác định nhánh nào sẽ được đi tiếp cho đến khi lệnh so sánh được thực hiện xong.
*   **Branch Prediction (Dự đoán nhánh):** Bộ xử lý hiện đại sử dụng logic dự đoán tinh vi để "đoán" xem lệnh nhảy có được thực thi hay không. Tỷ lệ đoán đúng của các bộ vi xử lý hiện đại thường đạt khoảng **90%**.
*   **Hậu quả khi đoán sai (Misprediction):** Nếu dự đoán sai, bộ xử lý phải hủy bỏ tất cả các công việc mà nó đã thực hiện trước đó trong đường ống, quay lại vị trí đúng và nạp lại đường ống từ đầu. Việc này gây ra một hình phạt nặng nề về hiệu năng, thường mất từ **15 đến 30 chu kỳ máy (clock cycles)**.

---

### Ví dụ so sánh thực tế: Intel Haswell
Sách thực hiện đo thời gian chạy của hàm `absdiff` trên bộ xử lý Intel Haswell với các điều kiện thử nghiệm khác nhau:

1.  **Dùng lệnh Nhảy (Conditional Jumps):**
    *   Nếu dữ liệu dễ dự đoán (luôn đúng hoặc luôn sai): Tốn khoảng **8 chu kỳ**.
    *   Nếu dữ liệu ngẫu nhiên (Dự đoán đúng 50%): Tốn khoảng **17.5 chu kỳ**.
    *   **Suy luận:** Hình phạt cho việc dự đoán sai nhánh (Misprediction penalty) được tính toán là khoảng **19 chu kỳ**. Thời gian thực thi biến động từ 8 đến 27 chu kỳ tùy vào độ chính xác của dự đoán.

2.  **Dùng Conditional Moves (`cmov`):**
    *   Tốn khoảng **8 chu kỳ** bất kể dữ liệu đầu vào là gì.
    *   **Lý do:** Luồng điều khiển không phụ thuộc vào dữ liệu. Bộ xử lý chỉ việc chạy thẳng một mạch, giúp giữ cho đường ống luôn đầy mà không cần phải dự đoán.

---

### Aside: Cách tính toán hình phạt (Penalty)
<details>
<summary><b>Nhấn để xem logic toán học</b></summary>

Giả sử xác suất dự đoán sai là $p$. Thời gian thực thi khi không có lỗi dự đoán là $T_{OK}$, và hình phạt khi dự đoán sai là $T_{MP}$.
Thời gian trung bình thực thi là: $T_{avg}(p) = (1 - p)T_{OK} + p(T_{OK} + T_{MP}) = T_{OK} + pT_{MP}$.

Với dữ liệu ngẫu nhiên ($p = 0.5$), ta có phương thức tính $T_{MP}$:
$T_{avg}(0.5) = T_{OK} + 0.5 T_{MP}$
$\Rightarrow T_{MP} = 2(T_{avg} - T_{OK})$

Áp dụng số liệu: $T_{OK} = 8$ và $T_{avg} = 17.5$:
$T_{MP} = 2(17.5 - 8) = \mathbf{19}$ chu kỳ.

</details>

---

### IDA Pro Insights (Khi nào nên tìm kiếm CMOV)

1.  **Tối ưu hóa "Flat code":** Trong IDA, nếu bạn thấy một đoạn code thực hiện các phép tính logic phức tạp mà hoàn toàn không có mũi tên rẽ nhánh (Graph View phẳng), hãy nhìn kỹ vào các lệnh `cmovCC`. Đây là dấu hiệu của việc trình biên dịch đang cố gắng "giải cứu" đường ống của CPU khỏi các lỗi dự đoán nhánh.
2.  **Logic "Tính cả hai":** Một đặc điểm nhận dạng `cmov` là bạn sẽ thấy các lệnh tính toán của **cả hai nhánh** (ví dụ `y-x` và `x-y`) xuất hiện liên tiếp nhau ngay trước lệnh `cmov`. 
3.  **Hệ quả khi dịch ngược:** Khi viết lại mã C từ `cmov`, bạn có thể viết dưới dạng toán tử ba ngôi `result = (x < y) ? (y - x) : (x - y);` để phản ánh đúng tư duy "chọn dữ liệu" của mã máy.

---

### Practice Problem 3.19: Tính toán hình phạt dự đoán sai nhánh (Branch Misprediction Penalty)

<img width="1049" height="323" alt="image" src="https://github.com/user-attachments/assets/ff6bbea6-e31f-46c2-8310-e4e65f5857ce" />

**Đề bài:**
Chạy trên một mẫu bộ xử lý mới, mã nguồn của chúng ta tốn khoảng **45 chu kỳ (cycles)** khi mẫu hình rẽ nhánh là ngẫu nhiên, và khoảng **25 chu kỳ** khi mẫu hình rẽ nhánh cực kỳ dễ dự đoán.

**Câu hỏi:**
A. Hình phạt khi dự đoán sai (miss penalty) xấp xỉ là bao nhiêu?
B. Hàm này sẽ tốn bao nhiêu chu kỳ khi nhánh bị dự đoán sai?

---
<details>

<summary>Phân tích và Giải bài tập</summary>

Dựa trên các công thức lý thuyết đã học ở trang trước (Mục 3.6.6):
*   **$T_{OK}$**: Thời gian thực thi khi dự đoán đúng (xác suất sai $p \approx 0$). Theo đề bài, $T_{OK} = 25$ chu kỳ.
*   **$T_{avg}(0.5)$**: Thời gian thực thi trung bình khi rẽ nhánh ngẫu nhiên (xác suất sai $p = 0.5$). Theo đề bài, $T_{avg} = 45$ chu kỳ.
*   **$T_{MP}$**: Hình phạt (penalty) khi dự đoán sai.

#### A. Tính toán hình phạt dự đoán sai ($T_{MP}$):
Sử dụng công thức tính thời gian trung bình: $T_{avg}(0.5) = T_{OK} + 0.5 \times T_{MP}$
1.  Thay số vào phương trình: $45 = 25 + 0.5 \times T_{MP}$
2.  Chuyển vế: $0.5 \times T_{MP} = 45 - 25 = 20$
3.  Kết quả: $T_{MP} = 20 / 0.5 = \mathbf{40}$ **chu kỳ**.

#### B. Tính toán tổng thời gian khi dự đoán sai:
Khi bộ xử lý dự đoán sai nhánh, nó phải thực hiện các công việc tính toán thông thường cộng thêm việc dọn dẹp và nạp lại đường ống (pipeline) do hình phạt dự đoán sai gây ra.
*   Công thức: $T_{mispredicted} = T_{OK} + T_{MP}$
*   Thay số: $25 + 40 = \mathbf{65}$ **chu kỳ**.

</details>

---

### IDA Pro Insights (Liên hệ thực tế)

Mặc dù đây là một bài toán định lượng về hiệu suất phần cứng, nhưng nó giải thích lý do tại sao trình biên dịch lại thực hiện những biến đổi mã mà bạn thấy trong IDA Pro:

*   **Cái giá của lệnh nhảy:** Bài tập này cho thấy một lệnh nhảy sai có thể khiến hàm chạy chậm đi hơn **2.5 lần** (từ 25 lên 65 chu kỳ).
*   **Giải thích cho lệnh `CMOV`:** Khi bạn soi mã trong IDA Pro và thấy trình biên dịch sử dụng các lệnh di chuyển có điều kiện (`cmovCC`) thay vì các cấu trúc rẽ nhánh (`jCC`) thông thường, đó là vì trình biên dịch muốn đảm bảo thời gian thực thi luôn là hằng số ($T_{OK}$), loại bỏ hoàn toàn rủi ro phải chịu hình phạt $T_{MP}$ rất lớn này.

---

### Hình 3.18: Các lệnh di chuyển dữ liệu có điều kiện (The conditional move instructions)

Các lệnh này sao chép giá trị từ nguồn $S$ sang đích $R$ nếu điều kiện thỏa mãn. 
*   **Toán hạng:** Nguồn có thể là thanh ghi hoặc bộ nhớ, nhưng đích **bắt buộc** phải là thanh ghi.
*   **Kích thước:** Hỗ trợ các đại lượng 16, 32, và 64 bits. Không hỗ trợ đơn byte (8-bit).
*   **Cú pháp:** Tên lệnh chứa hậu tố điều kiện (tương tự `SET` và `JUMP`).

| Lệnh (ATT/Intel) | Synonym (Đồng nghĩa) | Điều kiện (Flag) | Mô tả |
| :--- | :--- | :--- | :--- |
| **`cmove S, R`** | `cmovz` | `ZF` | Bằng nhau / Bằng 0 |
| **`cmovne S, R`** | `cmovnz` | `~ZF` | Khác nhau / Khác 0 |
| **Số có dấu** | | | |
| **`cmovs S, R`** | | `SF` | Số âm |
| **`cmovns S, R`** | | `~SF` | Số không âm |
| **`cmovg S, R`** | `cmovnle` | `~(SF ^ OF) & ~ZF` | Lớn hơn |
| **`cmovge S, R`** | `cmovnl` | `~(SF ^ OF)` | Lớn hơn hoặc bằng |
| **`cmovl S, R`** | `cmovnge` | `SF ^ OF` | Nhỏ hơn |
| **`cmovle S, R`** | `cmovng` | `(SF ^ OF) \| ZF` | Nhỏ hơn hoặc bằng |
| **Số không dấu** | | | |
| **`cmova S, R`** | `cmovnbe` | `~CF & ~ZF` | Lớn hơn (Above) |
| **`cmovae S, R`** | `cmovnb` | `~CF` | Lớn hơn hoặc bằng |
| **`cmovb S, R`** | `cmovnae` | `CF` | Nhỏ hơn (Below) |
| **`cmovbe S, R`** | `cmovna` | `CF \| ZF` | Nhỏ hơn hoặc bằng |

---

### Chuyển đổi biểu thức điều kiện sang mã máy

Xét biểu thức gán có điều kiện trong C:
`v = test-expr ? then-expr : else-expr;`

#### 1. Cách tiếp cận truyền thống (Control Transfer)
Sử dụng lệnh nhảy để bỏ qua một trong hai nhánh. Logic này có hai chuỗi lệnh riêng biệt:
```c
    if (!test-expr)
        goto false;
    v = then-expr;
    goto done;
false:
    v = else-expr;
done:
```
*Nhược điểm:* Hiệu năng kém nếu CPU dự đoán sai nhánh.

#### 2. Cách tiếp cận hiện đại (Conditional Move)
Cả `then-expr` và `else-expr` đều được tính toán trước. Lệnh `cmov` sau đó được dùng để chọn giá trị cuối cùng. Logic này có thể mô tả qua mã trừu tượng sau:
```c
    v = then-expr;
    ve = else-expr;
    t = test-expr;
    if (!t) v = ve; // Lệnh cmov thực hiện dòng này
```
Dòng cuối cùng trong chuỗi này được thực hiện bằng một lệnh `cmov` duy nhất — giá trị của `ve` chỉ được sao chép vào `v` nếu điều kiện `t` không thỏa mãn.

---

### Hạn chế của Conditional Move

Không phải mọi biểu thức điều kiện đều có thể được biên dịch bằng lệnh `cmov`. 
*   **Đặc điểm quan trọng:** Mã trừu tượng ở trên cho thấy bộ xử lý luôn phải tính toán cả `then-expr` và `else-expr` bất kể kết quả của `test-expr` là gì. 
*   Nếu một trong hai phép toán này có thể gây ra lỗi (ví dụ: chia cho 0 hoặc truy cập con trỏ NULL), hoặc nếu chúng có các tác dụng phụ (side effects) không mong muốn, trình biên dịch sẽ không được phép sử dụng `cmov`.

---

### IDA Pro Insights (Nhận diện logic 3 ngôi)

*   **Toán tử 3 ngôi (`? :`):** Khi bạn thấy cấu trúc gán giá trị bằng `cmov` trong IDA, khả năng rất cao mã nguồn gốc sử dụng toán tử 3 ngôi hoặc một lệnh `if-else` cực kỳ đơn giản mà trình biên dịch đã tự tối ưu hóa.
*   **Kích thước toán hạng:** Vì `cmov` không hỗ trợ 8-bit, nếu bạn so sánh các biến kiểu `char`, trình biên dịch sẽ nạp chúng vào thanh ghi 32-bit (như `eax`, `ebx`) rồi mới dùng `cmov`.
*   **Truy cập bộ nhớ:** Lệnh `cmov` có thể đọc từ bộ nhớ (`cmovge rax, [rdi]`). Bộ xử lý sẽ luôn thực hiện việc đọc bộ nhớ này trước khi kiểm tra flag. Nếu địa chỉ trong `[rdi]` không hợp lệ, chương trình sẽ crash ngay lập tức dù điều kiện `ge` có đúng hay không. Đây là lý do tại sao IDA thường hiển thị `if-else` truyền thống cho các thao tác kiểm tra con trỏ.

---

## 3.6.6 Implementing Conditional Branches with Conditional Moves (Tiếp theo)

Như đã đề cập, không phải mọi biểu thức điều kiện đều có thể được biên dịch bằng lệnh `cmov`. Lý do chính là vì mã máy phải tính toán kết quả của cả hai nhánh `then-expr` và `else-expr` trước khi chọn lựa. Nếu một trong hai nhánh gây ra lỗi hoặc có tác dụng phụ, chương trình sẽ hoạt động sai.

### Trường hợp rủi ro: Giải mã con trỏ (Pointer Dereferencing)

Xét hàm C sau đây, dùng để đọc giá trị từ một con trỏ nhưng trả về 0 nếu con trỏ là NULL:

```c
long cread(long *xp) {
    return (xp ? *xp : 0);
}
```

#### Một cách triển khai SAI (Invalid implementation):
Nếu trình biên dịch cố gắng dùng `cmov`, mã assembly có thể trông như sau:

<img width="562" height="243" alt="image" src="https://github.com/user-attachments/assets/163182d7-d28d-4aa1-ba8a-b3931a1813f0" />


| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Logic tính toán |
| :--- | :--- | :--- | :--- |
| 1 | `movq (%rdi), %rax` | `mov rax, [rdi]` | `v = *xp` (Giải mã con trỏ) |
| 2 | `testq %rdi, %rdi` | `test rdi, rdi` | Kiểm tra `xp` có phải NULL không. |
| 3 | `movl $0, %edx` | `mov edx, 0` | `ve = 0` |
| 4 | **`cmove %rdx, %rax`** | **`cmovz rax, rdx`** | Nếu `xp == NULL`, đặt kết quả = 0. |
| 5 | `ret` | `retn` | Trả về. |

**Tại sao mã này lại SAI?**
*   **Lỗi bộ nhớ:** Lệnh ở dòng 1 (`movq (%rdi), %rax`) thực hiện việc đọc bộ nhớ ngay lập tức. 
*   Nếu `xp` là con trỏ NULL (`0`), lệnh này sẽ gây ra lỗi truy cập bộ nhớ (segmentation fault) và làm chương trình crash **ngay lập tức**, trước khi lệnh `cmove` ở dòng 4 kịp sửa sai.
*   **Kết luận:** Trong trường hợp này, trình biên dịch **bắt buộc** phải sử dụng cấu trúc rẽ nhánh (lệnh nhảy) để đảm bảo lệnh giải mã con trỏ chỉ được thực hiện khi điều kiện con trỏ khác NULL là đúng.

---

### Cân nhắc về Hiệu năng (Efficiency Trade-offs)

Ngay cả khi việc tính toán cả hai nhánh không gây ra lỗi, lệnh `cmov` cũng không phải lúc nào cũng tốt hơn lệnh nhảy:
1.  **Phí phạm tính toán:** Nếu các biểu thức ở hai nhánh đòi hỏi nhiều phép tính phức tạp, bộ xử lý sẽ lãng phí rất nhiều chu kỳ máy để tính toán một kết quả mà cuối cùng sẽ bị vứt bỏ.
2.  **Lựa chọn của Trình biên dịch:** 
    *   Trình biên dịch phải cân nhắc giữa việc tính toán dư thừa (`cmov`) và rủi ro bị phạt do dự đoán sai nhánh (`jump`).
    *   Thực tế, các trình biên dịch như GCC chỉ sử dụng `conditional move` khi các biểu thức cực kỳ đơn giản (ví dụ: chỉ là một lệnh cộng hoặc trừ đơn lẻ).
    *   Ngay cả khi hình phạt do dự đoán sai nhánh là rất lớn, trình biên dịch vẫn sẽ ưu tiên lệnh nhảy nếu biểu thức tính toán quá phức tạp.

---

### IDA Pro Insights (Kỹ năng nhận diện logic an toàn)

*   **Bảo vệ con trỏ:** Khi bạn thấy một cấu trúc `if (ptr != NULL)` trong C, trong IDA bạn sẽ **luôn luôn** thấy nó dưới dạng một lệnh nhảy (`jnz` hoặc `jz`) chứ không bao giờ là `cmov`. Đây là dấu hiệu nhận diện các bước kiểm tra an toàn bộ nhớ.
*   **Dấu hiệu của toán tử 3 ngôi:** Ngược lại, nếu bạn thấy các lệnh tính toán số học đơn giản (như cộng, trừ, dịch bit) của hai nhánh xuất hiện đồng thời rồi kết thúc bằng `cmov`, bạn có thể tự tin dịch ngược nó về toán tử `? :` trong C.
*   **Sự thông minh của CPU modern:** Các bộ xử lý hiện đại rất giỏi trong việc dự đoán các nhánh lặp đi lặp lại. Vì vậy, các lệnh nhảy truyền thống đôi khi vẫn đạt hiệu suất cực cao mà không cần đến `cmov`.

---

### Practice Problem 3.20: Nhận diện phép Chia số nguyên có dấu

<img width="996" height="665" alt="image" src="https://github.com/user-attachments/assets/d0fe7ded-589f-4253-bdde-b84ae3fa42c5" />


**Đề bài:** Xác định toán tử `OP` trong đoạn mã C sau:

```c
#define OP ________ /* Toán tử bí ẩn */

short arith(short x) {
    return x OP 16;
}
```

**Mã Assembly phân tích:**

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `leaq 15(%rdi), %rbx` | `lea rbx, [rdi + 15]` | `rbx = x + 15` (Tính toán Bias). |
| 2 | `testq %rdi, %rdi` | `test rdi, rdi` | Kiểm tra dấu của `x`. |
| 3 | `cmovns %rdi, %rbx` | `cmovns rbx, rdi` | **Nếu x >= 0**: `rbx = x`. (Bỏ qua Bias). |
| 4 | `sarq $4, %rbx` | `sar rbx, 4` | Dịch phải số học 4 bit ($2^4 = 16$). |

**Phân tích mẫu hình (Idiom):**
Đây là cách tối ưu để thực hiện phép **Chia số nguyên có dấu cho lũy thừa của 2** ($x / 2^k$). 
*   Trong số học máy tính, phép dịch phải (`sar`) của số âm luôn làm tròn xuống ($-\infty$). Tuy nhiên, ngôn ngữ C yêu cầu phép chia phải làm tròn về số **0**.
*   Để sửa lỗi này, trình biên dịch thêm một giá trị **Bias** ($2^k - 1$) vào số bị chia trước khi dịch nếu số đó là số âm.
*   Ở đây $k=4$ ($2^4=16$), nên Bias là $16-1 = 15$. 

**Kết luận:** Toán tử `OP` là phép chia **`/`**.

---

### Practice Problem 3.21: Dịch ngược cấu trúc rẽ nhánh phức tạp

<img width="732" height="812" alt="image" src="https://github.com/user-attachments/assets/b971a8c6-d4a3-49c1-8f75-d077eb0bdfd3" />


**Đề bài:** Điền vào các biểu thức còn thiếu trong mã C dựa trên luồng Assembly.

#### Phân tích luồng Assembly (Ánh xạ thanh ghi: x in %rdi, y in %rsi):

1.  **Khởi tạo:** `leaq 12(%rsi), %rbx` $\rightarrow$ `val = y + 12`.
2.  **Kiểm tra điều kiện 1:** `testq %rdi, %rdi` + `jge .L2`
    *   Nếu `x >= 0` nhảy tới `.L2`.
    *   $\Rightarrow$ Nhánh chạy thẳng (fall-through) là **`if (x < 0)`**.
3.  **Bên trong nhánh `if (x < 0)`:**
    *   Tính `t1 = x * y` (lệnh `imulq`).
    *   Tính `t2 = x | y` (lệnh `orq`).
    *   So sánh `x` và `y` (`cmpq %rsi, %rdi`).
    *   `cmovge %rdx, %rbx`: Nếu $x \ge y$, chọn `t2`, ngược lại giữ `t1`.
    *   $\Rightarrow$ `if (x < y) val = x * y; else val = x | y;`
4.  **Tại nhãn `.L2` (Nhánh `else if` khi $x \ge 0$):**
    *   Thực hiện phép chia `x / y` (lệnh `idivq`).
    *   So sánh `y` với `10` (`cmpq $10, %rsi`).
    *   `cmovge %rdi, %rbx`: Nếu $y \ge 10$, cập nhật `val = x / y`.

#### Lời giải mã C hoàn chỉnh:

```c
short test(short x, short y) {
    short val = y + 12;    // Lệnh leaq đầu hàm
    if (x < 0) {           // jge .L2 (nhảy nếu x >= 0, nên fall-through là x < 0)
        if (x < y)         // cmovge sử dụng rdx (x|y) khi x >= y, nên mặc định là x * y
            val = x * y;
        else
            val = x | y;
    } else if (y >= 10)    // Nhãn .L2 và cmovge khi y >= 10
        val = x / y;
    return val;
}
```

---

### IDA Pro Insights (Mẹo cho người Reverse)

1.  **Nhận diện Phép chia tối ưu (3.20):** Khi bạn thấy tổ hợp `lea (bias)` + `test` + `cmovns` + `sar`, hãy đọc ngay lập tức là một phép chia hằng số lũy thừa 2. Đừng mất công tính toán từng bước.
2.  **Hỗn hợp Jump và Cmov (3.21):** 
    *   Trình biên dịch thường kết hợp cả hai. Lệnh nhảy (`jge`) được dùng cho các khối mã lớn hoặc phức tạp (`if (x < 0)`). 
    *   Lệnh di chuyển có điều kiện (`cmovge`) được dùng cho các phép gán đơn giản bên trong đó để tránh làm hỏng đường ống (pipeline) của CPU.
3.  **Thứ tự trong Cmov:** Luôn nhớ lệnh `cmovCC` hoạt động như sau:
    *   `mov temp, val1`
    *   `op val2`
    *   `cmovCC temp, val2` (Nếu thỏa mãn điều kiện, `temp` sẽ bị ghi đè bởi `val2`).
    *   Điều này giúp bạn xác định đâu là nhánh `if` và đâu là nhánh `else` trong mã nguồn C gốc.

---

## 3.6.7 Loops (Vòng lặp)

Ngôn ngữ C cung cấp các cấu trúc vòng lặp như `do-while`, `while`, và `for`. Tuy nhiên, **không có lệnh vòng lặp tương ứng trong mã máy**. Thay vào đó, trình biên dịch sử dụng kết hợp giữa các phép thử điều kiện và lệnh nhảy (jumps) để tạo ra hiệu ứng lặp.

### Do-While Loops (Vòng lặp Do-While)

Cấu trúc tổng quát của `do-while` trong C:
```c
do
    body-statement
while (test-expr);
```

Hiệu ứng của vòng lặp là thực thi `body-statement` liên tục, đánh giá `test-expr`, và tiếp tục lặp nếu kết quả đánh giá là khác 0. **Lưu ý:** `body-statement` luôn được thực thi ít nhất một lần.

**Dạng chuyển đổi sang `goto` (mô phỏng mã máy):**
```c
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
```
Trong mỗi lần lặp, chương trình thực thi nội dung vòng lặp rồi mới kiểm tra điều kiện. Nếu thỏa mãn, nó nhảy ngược lại (jump back) về phía trên.

---

### Ví dụ: Hàm tính Giai thừa (`fact_do`)

Hình 3.19 minh họa cách triển khai hàm tính $n!$ bằng vòng lặp `do-while`. Lưu ý hàm này chỉ hoạt động đúng với $n > 0$.

#### 1. Các phiên bản mã nguồn

| (a) Original C code | (b) Equivalent goto version |
| :--- | :--- |
| ```c {4-8} long fact_do(long n) { long result = 1; do { result *= n; n = n - 1; } while (n > 1); return result; } ``` | ```c {4-10} long fact_do_goto(long n) { long result = 1; loop: result *= n; n = n - 1; if (n > 1) goto loop; return result; } ``` |

#### 2. Phân tích mã Assembly
*Quy ước: n nằm trong %rdi, result nằm trong %rax*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `fact_do:` | `fact_do:` | Nhãn hàm. |
| 2 | `movl $1, %eax` | `mov eax, 1` | Khởi tạo `result = 1` (Lưu ý: xóa 32-bit cao của rax). |
| 3 | `.L2:` | `loc_L2:` | **Nhãn vòng lặp (Loop label).** |
| 4 | `imulq %rdi, %rax` | `imul rax, rdi` | `result *= n` |
| 5 | `subq $1, %rdi` | `sub rdi, 1` | `n = n - 1` |
| 6 | `cmpq $1, %rdi` | `cmp rdi, 1` | So sánh `n` với 1. |
| 7 | **`jg .L2`** | **`jg short loc_L2`** | **Nếu n > 1, nhảy ngược về .L2.** |
| 8 | `rep; ret` | `retn` | Trả về `result`. |

---

### IDA Pro Insights (Kỹ năng nhận diện vòng lặp)

1.  **Mũi tên ngược (Back-edges):** 
    *   Trong Graph View của IDA, điểm đặc trưng nhất của vòng lặp là một mũi tên xuất phát từ khối lệnh phía dưới quay ngược lên một khối lệnh phía trên.
    *   Với `do-while`, bạn sẽ thấy một khối lệnh thực hiện tính toán, sau đó kết thúc bằng một lệnh so sánh và một mũi tên (thường là màu xanh lá - True) quay lại chính đầu khối đó hoặc khối phía trên.
2.  **Cấu trúc tối giản:** 
    *   Vòng lặp `do-while` là cấu trúc hiệu quả nhất về mặt mã máy vì nó chỉ tốn **duy nhất một lệnh nhảy có điều kiện** để duy trì vòng lặp. Các loại vòng lặp khác (`while`, `for`) thường phức tạp hơn.
3.  **Thanh ghi `%rax`:** 
    *   Trình biên dịch dùng `movl $1, %eax` để tối ưu kích thước mã máy thay vì `movq $1, %rax`. Quy ước x86-64 sẽ tự động xóa các bit cao, đảm bảo `%rax` là 1.

---

### Practice Problem 3.22: Tràn số trong tính toán Giai thừa

Bài tập này yêu cầu xem xét giới hạn của các kiểu dữ liệu khi tính toán giai thừa ($n!$), dựa trên ví dụ hàm `fact_do` ở trang trước.

#### A. Tính $14!$ với kiểu `int` 32-bit
*   **Giới hạn của `int` 32-bit (có dấu):** Giá trị lớn nhất là $2^{31} - 1 = \mathbf{2,147,483,647}$ (khoảng $2.1 \times 10^9$).
*   **Giá trị giai thừa:**
    *   $12! = 479,001,600$ (Vẫn nằm trong giới hạn).
    *   $13! = 6,227,020,800$ (Đã lớn hơn $2.1 \times 10^9 \rightarrow$ **Bắt đầu tràn số**).
    *   $14! = 87,178,291,200$.
*   **Kết luận:** Việc tính $14!$ bằng kiểu `int` 32-bit chắc chắn sẽ gây ra **tràn số**. Kết quả nhận được trong thanh ghi sẽ bị sai lệch hoàn toàn so với giá trị toán học thực tế.

#### B. Tính $14!$ với kiểu `long int` 64-bit
*   **Giới hạn của `long` 64-bit (có dấu):** Giá trị lớn nhất là $2^{63} - 1 \approx \mathbf{9.22 \times 10^{18}}$.
*   **Giá trị giai thừa:**
    *   $14! = 87,178,291,200$ (khoảng $8.7 \times 10^{10}$).
    *   Kiểu 64-bit có thể chứa được giá trị lên đến **$20!$** ($20! \approx 2.4 \times 10^{18}$).
*   **Kết luận:** Với kiểu `long` 64-bit, việc tính $14!$ diễn ra **an toàn**, không bị tràn số.

---

### IDA Pro Insights (Nhận diện lỗi tràn số)

Khi bạn dịch ngược một hàm tính toán như giai thừa trong IDA Pro, hãy chú ý các dấu hiệu sau để biết liệu chương trình có đang xử lý tràn số hay không:

1.  **Kích thước lệnh và thanh ghi:**
    *   Nếu bạn thấy các lệnh sử dụng thanh ghi 32-bit (`eax`, `ebx`, `imull`), chương trình đang dùng kiểu `int`. Hãy cẩn thận với các số lớn hơn 12!.
    *   Nếu bạn thấy thanh ghi 64-bit (`rax`, `rbx`, `imulq`), chương trình đang dùng kiểu `long`.

2.  **Kiểm tra Flags sau phép nhân:**
    *   Lệnh nhân `imul` sẽ thiết lập **Overflow Flag (OF)** và **Carry Flag (CF)** nếu kết quả vượt quá kích thước của thanh ghi đích.
    *   Trong IDA, nếu bạn thấy sau lệnh `imul` có một lệnh nhảy như **`jo`** (Jump if Overflow) hoặc **`into`** (Interrupt on Overflow), đó là dấu hiệu mã nguồn C có các cơ chế kiểm tra lỗi tràn số. Nếu không có, chương trình sẽ lẳng lặng chạy tiếp với kết quả sai.

3.  **Hậu quả trong Debugger:**
    *   Khi bạn trace hàm này trong IDA Debugger, nếu xảy ra tràn số ở kiểu 32-bit, giá trị trong thanh ghi `eax` sẽ đột ngột thay đổi từ một số dương lớn sang một số âm (do bit dấu bị ghi đè) hoặc một số dương nhỏ không logic. Đây là dấu hiệu nhận biết kiểu dữ liệu bị chọn sai kích thước trong mã nguồn gốc.

---

### Phân tích logic vòng lặp Do-While (Tiếp theo)

Mã `goto` ở Hình 3.19(b) cho thấy cách vòng lặp được chuyển đổi thành sự kết hợp cấp thấp của các phép thử và lệnh nhảy có điều kiện. Sau khi khởi tạo `result`, chương trình bắt đầu vòng lặp:
1.  Thực thi thân vòng lặp (cập nhật các biến `result` và `n`).
2.  Kiểm tra xem $n > 1$ hay không.
3.  Nếu đúng, nhảy ngược lại điểm bắt đầu vòng lặp.

Lệnh nhảy có điều kiện **`jg` (dòng 7)** là chỉ thị then chốt để triển khai vòng lặp. Nó quyết định việc tiếp tục lặp hay thoát ra ngoài.

---

### Aside: Reverse Engineering Loops (Nghịch đảo mã vòng lặp)

Chìa khóa để hiểu mối liên hệ giữa mã assembly và mã nguồn gốc là tìm ra sự **ánh xạ (mapping)** giữa các giá trị trong chương trình và các thanh ghi. 

**Thách thức thực tế:**
*   Với các chương trình phức tạp, trình biên dịch thường sắp xếp lại các phép tính khiến một số biến trong mã C không có thành phần tương ứng trực tiếp trong mã máy.
*   Nhiều giá trị mới được đưa vào mã máy mà không hề tồn tại trong mã nguồn gốc.
*   Trình biên dịch cố gắng giảm thiểu việc sử dụng thanh ghi bằng cách ánh xạ nhiều giá trị khác nhau của chương trình vào cùng một thanh ghi duy nhất.

**Chiến lược giải đố (Clues):**
Để giải mã vòng lặp, hãy quan sát các manh mối sau:
1.  **Khởi tạo:** Thanh ghi được nạp giá trị gì trước khi vào vòng lặp?
2.  **Cập nhật & Kiểm tra:** Thanh ghi đó được thay đổi và so sánh như thế nào bên trong vòng lặp?
3.  **Sử dụng sau vòng lặp:** Giá trị của thanh ghi đó được dùng làm gì sau khi thoát vòng lặp?

---

### Thực hành ánh xạ thanh ghi cho hàm `fact_do`

Để nghịch đảo mã Hình 3.19(c), ta cần xác định thanh ghi nào giữ giá trị nào:

| Vị trí | Lệnh (ATT) | Lệnh (Intel/IDA) | Phân tích logic |
| :--- | :--- | :--- | :--- |
| **Tham số** | (Ngầm định) | (Ngầm định) | Ta biết `n` được truyền vào qua thanh ghi `%rdi`. |
| **Dòng 2** | `movl $1, %eax` | `mov eax, 1` | Khởi tạo giá trị 1. (Lưu ý: ghi vào `eax` cũng xóa luôn 32 bit cao của `rax`). |
| **Dòng 4** | `imulq %rdi, %rax` | `imul rax, rdi` | `%rax` được cập nhật liên tục bằng phép nhân với `%rdi` (`n`). |
| **Kết thúc** | `rep; ret` | `retn` | `%rax` được dùng để trả về giá trị của hàm. |

**Kết luận:** Vì `%rax` được khởi tạo bằng 1, được nhân dồn trong vòng lặp và dùng để trả về, ta khẳng định `%rax` tương ứng với biến **`result`** trong mã C.

---

### IDA Pro Insights (Tư duy Reverse Engineering)

1.  **Xác định biến điều khiển:** Trong IDA, hãy nhìn vào lệnh so sánh ngay trước mũi tên nhảy ngược (back-edge). Trong ví dụ này là `cmp rdi, 1`. Thanh ghi nằm trong lệnh này (`rdi`) chính là biến điều khiển vòng lặp (biến `n`).
2.  **Theo dõi thanh ghi tích lũy:** Mọi hàm tính toán thường trả về kết quả qua `rax`. Nếu bạn thấy `rax` được khởi tạo trước vòng lặp và được cập nhật bên trong, đó chính là biến "đích" (như `result` hoặc `sum`).
3.  **Graph View Mapping:** 
    *   Khối lệnh nằm trên nhãn `.L2` là phần **khởi tạo**.
    *   Khối lệnh chứa mũi tên quay lại chính nó là **thân vòng lặp**.
    *   Đường mũi tên đi xuống dưới (thoát vòng lặp) dẫn đến phần **kết thúc/trả về**.
4.  **Tối ưu hóa thanh ghi:** IDA đôi khi sẽ đổi tên thanh ghi thành tên biến (như `n` hoặc `result`) nếu có thông tin debug. Nếu không, bạn nên dùng tính năng **Rename Register (phím N)** trong IDA để đổi tên `%rdi` thành `arg_n` và `%rax` thành `var_result` để dễ đọc hơn.

---

*(Nội dung bức ảnh kết thúc ở phần kết luận về việc ánh xạ %rax với biến result)*.

<tạm dừng>
