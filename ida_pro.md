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

<tiếp tục>

Dưới đây là nội dung từ trang 213, tập trung vào lớp lệnh di chuyển dữ liệu có bù dấu (**Sign-extension**) và lệnh đặc biệt `cltq`. Đây là những lệnh tối quan trọng khi làm việc với các kiểu dữ liệu có dấu (signed integers) như `int` sang `long` trong C.

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
| **`cltq`** | **`cdqe`** | `%rax \leftarrow \text{SignExtend}(\%eax)` | Mở rộng dấu `%eax` vào `%rax`. |

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

Bài tập này yêu cầu chọn cặp lệnh di chuyển dữ liệu phù hợp để thực hiện phép toán:
`*dp = (dest_t) *sp;`
Với `sp` là con trỏ kiểu `src_t`, `dp` là con trỏ kiểu `dest_t`.

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

<tiếp tục>

Dưới đây là nội dung từ trang 216, bao gồm phần giải thích về con trỏ dành cho người mới và lời giải chi tiết cho bảng chuyển đổi kiểu dữ liệu trong bài tập 3.4.

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

Bạn được cung cấp thông tin sau về một hàm có prototype:

```c
void decode1(long *xp, long *yp, long *zp);
```

*(Nội dung bức ảnh dừng lại ở phần giới thiệu Practice Problem 3.5)*.

---

<tạm dừng>
