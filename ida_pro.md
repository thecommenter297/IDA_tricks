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

*(Nội dung bức ảnh dừng lại ở đoạn dẫn nhập vào Mục 3.7)*.

---

<tạm dừng>
