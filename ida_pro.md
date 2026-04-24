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

*(Nội dung bức ảnh dừng lại ở phần giải thích về phép chia không dấu sử dụng divq)*.

---

<tạm dừng>
