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
| 1 | `orq %rsi, %rdx` | `or rdx, rsi` | `rdx = z \| y` (Đây là `p1`) |
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

<img width="821" height="407" alt="image" src="https://github.com/user-attachments/assets/0c824e20-3472-4cab-9407-1034d29c6951" />


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

### Practice Problem 3.23: Nghịch đảo mã vòng lặp có sử dụng con trỏ

**Mã nguồn C:**
```c
short dw_loop(short x) {
    short y = x / 9;
    short *p = &x;
    short n = 4 * x;
    do {
        x += y;
        (*p) += 5;
        n -= 2;
    } while (n > 0);
    return x;
}
```

**Mã Assembly phân tích (ATT Syntax):**

| Dòng | Lệnh Assembly | Phân tích logic | Ánh xạ biến |
| :--- | :--- | :--- | :--- |
| 1 | `movq %rdi, %rbx` | Sao chép `x` vào `%rbx` | `%rbx` $\leftarrow$ `x` |
| 2 | `movq %rdi, %rcx` | Chuẩn bị tính `y` | |
| 3 | `idivq ...` (blur) | Thực hiện `x / 9` | `%rcx` $\leftarrow$ `y` |
| 4 | `leaq (,%rdi,4), %rdx`| Tính `4 * x` | `%rdx` $\leftarrow$ `n` |
| 5 | `.L2:` | **Nhãn vòng lặp** | |
| 6 | `leaq 5(%rbx,%rcx), %rbx`| **`rbx = rbx + rcx + 5`** | **`x = x + y + 5`** |
| 7 | `subq $2, %rdx` | `n = n - 2` | `n` cập nhật |
| 8 | `testq %rdx, %rdx` | Kiểm tra `n` | |
| 9 | `jg .L2` | Nếu `n > 0`, lặp lại | |
| 10 | `rep; ret` | Trả về `%rax` (giả định là `%rbx`) | |

---

### Trả lời câu hỏi:

#### A. Thanh ghi nào được dùng cho các biến x, y và n?
*   **`x`**: Nằm trong thanh ghi **`%rbx`**. (Ban đầu là `%rdi`, sau đó được đưa vào `%rbx` để bảo toàn qua các vòng lặp).
*   **`y`**: Nằm trong thanh ghi **`%rcx`**.
*   **`n`**: Nằm trong thanh ghi **`%rdx`**.

#### B. Trình biên dịch đã loại bỏ biến con trỏ `p` và việc giải mã con trỏ (`*p += 5`) như thế nào?
*   **Cơ chế:** Trình biên dịch nhận thấy `p` luôn trỏ đến địa chỉ của `x` (`short *p = &x`). 
*   Thay vì thực hiện hai bước riêng biệt: `x += y` (thanh ghi) và `*p += 5` (bộ nhớ), trình biên dịch gộp chúng lại thành một phép toán duy nhất trên thanh ghi: **`x = x + y + 5`**. 
*   Điều này được thể hiện qua lệnh **`leaq 5(%rbx,%rcx), %rbx`** ở dòng 6. Việc loại bỏ truy cập bộ nhớ giúp vòng lặp chạy nhanh hơn đáng kể.

#### C. Chú thích mã Assembly (Annotated Code):
```assembly
dw_loop:
    movq    %rdi, %rbx          ; rbx = x
    movq    %rdi, %rax          ; Chuẩn bị số bị chia cho phép /9
    ... (phép chia) ...         ; rcx = x / 9 (biến y)
    leaq    (,%rdi,4), %rdx     ; rdx = 4 * x (biến n)
.L2:                            ; Vòng lặp:
    leaq    5(%rbx,%rcx), %rbx  ; x = x + y + 5 (gộp x += y và *p += 5)
    subq    $2, %rdx            ; n -= 2
    testq   %rdx, %rdx          ; Kiểm tra n
    jg      .L2                 ; Nếu n > 0, tiếp tục lặp
    movq    %rbx, %rax          ; Trả về x
    ret
```

---

### IDA Pro Insights (Kỹ năng đọc code tối ưu)

1.  **Sự biến mất của con trỏ:** Trong IDA, nếu bạn thấy một con trỏ được khai báo nhưng mã máy hoàn toàn không có lệnh truy cập bộ nhớ dạng `[reg]`, hãy nghi ngờ ngay việc trình biên dịch đã "nội hàm hóa" (inlining) biến đó vào thanh ghi.
2.  **Lệnh LEA làm toán:** Hãy chú ý lệnh `leaq 5(%rbx,%rcx), %rbx`. Đây không phải là tính địa chỉ, mà là trình biên dịch dùng đơn vị tính địa chỉ để làm phép toán `a = a + b + hằng_số` trong đúng 1 chu kỳ máy.
3.  **Hội hợp biến (Variable Merging):** Đây là kỹ thuật tối ưu hóa phổ biến. Khi hai tên biến cùng trỏ vào một thực thể dữ liệu, trình biên dịch sẽ gộp các thao tác trên chúng lại để tiết kiệm lệnh.

---

### While Loops (Vòng lặp While)

Cấu trúc tổng quát của lệnh `while` trong C:
```c
while (test-expr)
    body-statement
```

**Sự khác biệt so với `do-while`:** 
Vòng lặp `while` đánh giá biểu thức điều kiện (`test-expr`) **trước khi** thực hiện thân vòng lặp (`body-statement`). Do đó, thân vòng lặp có khả năng không được thực thi lần nào nếu điều kiện sai ngay từ đầu.

---

### Phương pháp 1: Jump-to-Middle (Nhảy vào giữa)

Đây là phương pháp mặc định của GCC khi sử dụng cờ `-Og`. Nó thực hiện bước kiểm tra ban đầu bằng cách sử dụng một lệnh nhảy không điều kiện dẫn tới cuối vòng lặp, nơi đặt biểu thức kiểm tra.

**Khuôn mẫu (Template) chuyển đổi sang `goto`:**
```c
    goto test;
loop:
    body-statement
test:
    t = test-expr;
    if (t)
        goto loop;
```

---

### Ví dụ: Hàm tính Giai thừa (`fact_while`)

Hình 3.20 minh họa cách triển khai hàm giai thừa bằng vòng lặp `while` theo phương pháp Jump-to-middle. Khác với phiên bản `do-while`, phiên bản này tính toán chính xác cho trường hợp $0! = 1$.

#### 1. Các phiên bản mã nguồn

| (a) Original C code | (b) Equivalent goto version |
| :--- | :--- |
| ```c {4-8} long fact_while(long n) { long result = 1; while (n > 1) { result *= n; n = n - 1; } return result; } ``` | ```c {4-12} long fact_while_jm_goto(long n) { long result = 1; goto test; loop: result *= n; n = n - 1; test: if (n > 1) goto loop; return result; } ``` |

#### 2. Phân tích mã Assembly (Generated Assembly code)
*Quy ước: n nằm trong %rdi, result nằm trong %rax*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `fact_while:` | `fact_while:` | Nhãn hàm. |
| 2 | `movl $1, %eax` | `mov eax, 1` | `result = 1`. |
| 3 | **`jmp .L5`** | **`jmp short loc_test`** | **Nhảy thẳng đến phần kiểm tra (test).** |
| 4 | `.L6:` | `loc_loop:` | **Nhãn thân vòng lặp (Body).** |
| 5 | `imulq %rdi, %rax` | `imul rax, rdi` | `result *= n`. |
| 6 | `subq $1, %rdi` | `sub rdi, 1` | `n = n - 1`. |
| 7 | `.L5:` | `loc_test:` | **Nhãn kiểm tra (Test).** |
| 8 | `cmpq $1, %rdi` | `cmp rdi, 1` | So sánh `n` với 1. |
| 9 | **`jg .L6`** | **`jg short loc_loop`** | **Nếu n > 1, nhảy ngược lên Body.** |
| 10 | `rep; ret` | `retn` | Trả về `result`. |

---

### IDA Pro Insights (Nhận diện Jump-to-Middle)

Khi bạn nhìn vào Graph View của IDA Pro, cấu trúc Jump-to-middle có các đặc điểm nhận dạng rất rõ ràng:

1.  **Cấu trúc 3 khối lệnh:**
    *   **Khối 1 (Entry/Init):** Chứa các lệnh khởi tạo và kết thúc bằng một mũi tên **xanh dương** (Jump không điều kiện) nhảy bỏ qua một khối mã.
    *   **Khối 2 (Body):** Nằm ngay dưới Khối 1 nhưng bị nhảy qua. Nó thực hiện logic chính của vòng lặp và sau đó chạy thẳng xuống Khối 3.
    *   **Khối 3 (Condition):** Chứa lệnh `cmp` và `jCC`. 
        *   Mũi tên **xanh lá** (True) quay ngược lại Khối 2.
        *   Mũi tên **đỏ** (False) đi xuống lệnh `ret`.

2.  **Tại sao lại dùng Jump-to-middle?**
    *   Cách này cho phép trình biên dịch sử dụng lại logic của vòng lặp `do-while` (vốn rất hiệu quả vì chỉ có một lệnh nhảy điều kiện ở cuối) cho vòng lặp `while` mà vẫn đảm bảo được việc kiểm tra điều kiện trước lần lặp đầu tiên.

3.  **Điểm khác biệt trong IDA:**
    *   Trong IDA, bạn sẽ thấy mũi tên xanh dương từ khối khởi tạo "đâm" vào giữa cấu trúc. Đó chính là lý do phương pháp này có tên là "Jump to middle".

---

### Practice Problem 3.24: Dịch ngược vòng lặp While (Jump-to-middle)

**Đề bài:** Hoàn thiện các biểu thức còn thiếu trong mã C dựa trên mã Assembly được cung cấp (sử dụng phương pháp Jump-to-middle).

**Mã Assembly phân tích:**
*Quy ước: a trong %rdi, b trong %rsi, result trong %rax*

| Dòng | AT&T Syntax | Intel Syntax (IDA) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `loop_while:` | `loop_while:` | Nhãn hàm. |
| 2 | `movl $0, %eax` | `mov eax, 0` | **`result = 0`**. |
| 3 | `jmp .L2` | `jmp short loc_test` | Nhảy đến phần kiểm tra. |
| 4 | `.L3:` | `loc_loop:` | **Nhãn thân vòng lặp**. |
| 5 | `leaq (%rsi,%rdi), %rdx`| `lea rdx, [rsi + rdi]` | `temp = b + a`. |
| 6 | `addq %rdx, %rax` | `add rax, rdx` | **`result += temp`**. |
| 7 | `subq $1, %rdi` | `sub rdi, 1` | **`a = a - 1`**. |
| 8 | `.L2:` | `loc_test:` | **Nhãn kiểm tra**. |
| 9 | `cmpq %rsi, %rdi` | `cmp rdi, rsi` | So sánh `a` với `b`. |
| 10| `jg .L3` | `jg short loc_loop` | **Nếu a > b, tiếp tục lặp**. |
| 11| `rep; ret` | `retn` | Trả về `result`. |

<details>
<summary><b>Nhấn để xem mã C hoàn chỉnh</b></summary>

```c
short loop_while(short a, short b) {
    short result = 0;
    while (a > b) {
        result = result + (a + b);
        a = a - 1;
    }
    return result;
}
```
</details>

---

### Phương pháp 2: Guarded-do (Do-while có bảo vệ)

Đây là chiến lược dịch vòng lặp `while` khi sử dụng mức tối ưu hóa cao hơn (ví dụ cờ `-O1`). Thay vì nhảy vào giữa, trình biên dịch biến đổi vòng lặp `while` thành một cấu trúc **`if` bao quanh một vòng lặp `do-while`**.

**Khuôn mẫu (Template) chuyển đổi:**
```c
/* Cách 1: Chuyển sang do-while */
t = test-expr;
if (!t)
    goto done;
do {
    body-statement
} while (test-expr);
done:

/* Cách 2: Chuyển hoàn toàn sang logic goto */
t = test-expr;
if (!t)
    goto done;
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
done:
```

### Tại sao Guarded-do lại tốt hơn?
*   **Tối ưu hóa khởi đầu:** Trình biên dịch thường có thể chứng minh được rằng điều kiện kiểm tra ban đầu luôn đúng, từ đó loại bỏ hoàn toàn lệnh `if` bảo vệ bên ngoài.
*   **Hiệu năng:** Thân vòng lặp lúc này chỉ đơn thuần là một khối `do-while` liên tục, rất thân thiện với bộ dự đoán nhánh của CPU.

---

### Ví dụ: Hàm giai thừa tối ưu hóa (Figure 3.21)

Khi biên dịch hàm giai thừa với `-O1`, trình biên dịch nhận ra rằng:
1.  Vòng lặp sẽ bị bỏ qua hoàn toàn nếu ban đầu $n \leq 1$.
2.  **Sự thay đổi về điều kiện:** Trong mã máy, thay vì kiểm tra `n > 1`, trình biên dịch có thể đổi thành **`n != 1`**. 
    *   *Lý do:* Vì nếu vòng lặp đã được thực thi (tức là $n > 1$) và mỗi bước `n` đều giảm đi 1, thì giá trị đầu tiên làm điều kiện `n > 1` bị sai chắc chắn phải là `n == 1`. 
    *   Phép thử `n != 1` đôi khi được CPU thực hiện hiệu quả hơn hoặc mã hóa ngắn gọn hơn.

---

### IDA Pro Insights (Nhận diện Guarded-do)

1.  **Cấu trúc Graph View đặc trưng:**
    *   Bạn sẽ thấy một khối lệnh kiểm tra ở đầu hàm với hai mũi tên: 
        *   Một mũi tên (False) nhảy vọt qua toàn bộ vòng lặp để tới lệnh `ret` (đây là cái "Guard" - lớp bảo vệ).
        *   Một mũi tên (True) dẫn vào một cấu trúc vòng lặp `do-while` khép kín.
2.  **Mũi tên lặp:** Trong IDA, cấu trúc này trông "sạch" hơn Jump-to-middle vì không có mũi tên xanh dương nhảy vào giữa khối. Nó chỉ có một lối vào duy nhất và một mũi tên nhảy ngược (back-edge) duy nhất.
3.  **Biến đổi điều kiện:** Đừng ngạc nhiên khi thấy IDA hiển thị lệnh `jnz` (nhảy nếu không bằng 0) thay vì `jg` (nhảy nếu lớn hơn) trong các vòng lặp đếm ngược. Đó là kết quả của việc trình biên dịch đã tối ưu hóa phép thử như giải thích ở phần "n != 1" phía trên.

---

### Practice Problem 3.25: Dịch ngược vòng lặp While (Guarded-do)

**Đề bài:** Hoàn thiện mã nguồn C dựa trên mã Assembly được biên dịch với cờ `-O1`.

**Phân tích luồng Assembly:**
*Quy ước: a nằm trong %rdi, b nằm trong %rsi, result nằm trong %rax*

| Dòng | AT&T Syntax | Intel Syntax (IDA) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `loop_while2:` | `loop_while2:` | Nhãn hàm. |
| 2 | `testq %rsi, %rsi` | `test rsi, rsi` | Kiểm tra giá trị `b`. |
| 3 | **`jle .L8`** | **`jle short loc_exit`** | **Lớp bảo vệ (Guard):** Nếu `b <= 0`, nhảy tới nhãn thoát. |
| 4 | `movq %rsi, %rax` | `mov rax, rsi` | Khởi tạo **`result = b`**. |
| 5 | `.L7:` | `loc_loop:` | **Nhãn bắt đầu do-while**. |
| 6 | `imulq %rdi, %rax` | `imul rax, rdi` | **`result *= a`**. |
| 7 | `subq %rdi, %rsi` | `sub rsi, rdi` | **`b -= a`**. |
| 8 | `testq %rsi, %rsi` | `test rsi, rsi` | Kiểm tra `b` mới. |
| 9 | **`jg .L7`** | **`jg short loc_loop`** | **Nếu b > 0, tiếp tục lặp**. |
| 10 | `rep; ret` | `retn` | Trả về từ nhánh lặp. |
| 11 | `.L8:` | `loc_exit:` | Nhãn khi không vào vòng lặp. |
| 12 | `movq %rsi, %rax` | `mov rax, rsi` | Gán `result = b` (khi b <= 0). |
| 13 | `ret` | `retn` | Trả về. |

<details>
<summary><b>Nhấn để xem mã C hoàn chỉnh</b></summary>

```c
long loop_while2(long a, long b) {
    long result = b;        // Khởi tạo dựa trên dòng 4 và 12
    while (b > 0) {         // Guard (dòng 3) và Condition (dòng 9)
        result = result * a; // Lệnh imulq
        b = b - a;          // Lệnh subq
    }
    return result;
}
```
</details>

---

### Figure 3.21: Giai thừa sử dụng Guarded-do Translation

Đây là phiên bản "xịn" hơn của vòng lặp `while`. Trình biên dịch không chỉ thêm lớp bảo vệ (`if`) mà còn tối ưu hóa cả phép thử điều kiện.

#### Phân tích điểm đặc biệt trong Assembly:
1.  **Hai lệnh `ret`:** Bạn sẽ thấy lệnh `ret` xuất hiện ở hai nơi (dòng 10 và 13). 
    *   Trong IDA Pro, đây là dấu hiệu của việc tối ưu hóa đường dẫn (Exit paths). Một cái dành cho trường hợp vòng lặp chạy xong, một cái dành cho trường hợp điều kiện `while` sai ngay từ đầu.
2.  **Tối ưu hóa phép thử (`n != 1`):**
    *   Nguyên bản C: `while (n > 1)`.
    *   Assembly dòng 9: `jne .L6` (Nhảy nếu không bằng 1).
    *   **Tại sao?** Vì trình biên dịch biết chắc chắn rằng nếu đã vượt qua lớp bảo vệ `n > 1` (dòng 2-3), và mỗi bước `n` giảm đi 1, thì giá trị đầu tiên khiến `n > 1` sai chỉ có thể là `n == 1`. Lệnh `jne` (không bằng) thường được thực thi nhanh hơn hoặc gọn hơn lệnh `jg` (lớn hơn).

---

### IDA Pro Insights (Tư duy dịch ngược nâng cao)

1.  **Nhận diện "Đường thoát hiểm" (Early Exit):**
    *   Trong Graph View, nếu khối lệnh đầu tiên có một mũi tên nhảy vọt xuống tận lệnh `ret` cuối cùng, đó chính là cấu trúc **Guard**. Nó bảo vệ chương trình không thực thi vòng lặp khi dữ liệu không hợp lệ.
2.  **Mẫu hình hai lần khởi tạo:**
    *   Để ý lệnh `movl $1, %eax` (khởi tạo `result`) xuất hiện ở cả dòng 4 và dòng 12. 
    *   Trình biên dịch làm vậy để đảm bảo dù chương trình đi theo nhánh nào (vào vòng lặp hay bỏ qua vòng lặp), thanh ghi trả về `%rax` luôn có giá trị đúng. Trong IDA, bạn sẽ thấy hai khối lệnh khác nhau cùng chứa một lệnh nạp hằng số giống nhau.
3.  **Tối ưu hóa `jne` thay cho `jg`:**
    *   Khi thấy một vòng lặp đếm ngược kết thúc bằng `jnz` hoặc `jne`, đừng vội kết luận mã nguồn C dùng `!=`. Hãy xem xét logic giảm dần, thường thì mã nguồn gốc vẫn dùng `> 0` hoặc `> 1`.

---

### Practice Problem 3.26: Nghịch đảo mã vòng lặp While và logic Bit

**Đề bài:** Hoàn thiện mã nguồn C và giải mã chức năng của hàm `test_one` dựa trên mã Assembly.

#### 1. Phân tích mã Assembly (Ánh xạ ATT và Intel/IDA Pro)
*Quy ước: x nằm trong %rdi, val nằm trong %rax*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `test_one:` | `test_one:` | Nhãn hàm. |
| 2 | `movl $1, %eax` | `mov eax, 1` | Khởi tạo **`val = 1`**. |
| 3 | **`jmp .L5`** | **`jmp short loc_test`** | **Jump-to-middle**: Nhảy đến phần kiểm tra. |
| 4 | `.L6:` | `loc_loop:` | **Nhãn thân vòng lặp (Body)**. |
| 5 | `xorq %rdi, %rax` | `xor rax, rdi` | **`val = val ^ x`** (XOR giá trị `x` vào `val`). |
| 6 | `shrq %rdi` | `shr rdi, 1` | **`x >>= 1`** (Dịch phải logic 1 bit). |
| 7 | `.L5:` | `loc_test:` | **Nhãn kiểm tra (Test)**. |
| 8 | `testq %rdi, %rdi` | `test rdi, rdi` | Kiểm tra `x` có bằng 0 hay không. |
| 9 | `jne .L6` | `jnz short loc_loop` | **Nếu x != 0, tiếp tục lặp**. |
| 10 | `andl $1, %eax` | `and eax, 1` | **`val = val & 1`** (Chỉ lấy bit cuối cùng). |
| 11 | `ret` | `retn` | Trả về kết quả. |

---

### Trả lời câu hỏi:

#### A. Phương pháp dịch vòng lặp nào đã được sử dụng?
*   **Trả lời:** Phương pháp **Jump-to-middle**. 
*   **Dấu hiệu:** Có lệnh nhảy không điều kiện (`jmp .L5`) ngay đầu hàm để nhảy qua thân vòng lặp và đi vào phần kiểm tra điều kiện trước.

#### B. Điền vào các phần còn thiếu của mã C:
```c
short test_one(unsigned short x) {
    short val = 1;
    while (x != 0) {
        val = val ^ x;
        x >>= 1;
    }
    return val & 1;
}
```

#### C. Mô tả chức năng của hàm này (bằng tiếng Anh/Việt):
*   **Chức năng:** Hàm này tính toán **tính chẵn lẻ (parity)** của giá trị `x`. 
*   **Giải thích:** Vòng lặp XOR tất cả các bit của `x` lại với nhau (thông qua việc dịch bit dần dần). Kết quả cuối cùng là `val & 1`.
    *   Nếu số lượng bit 1 trong `x` là **số lẻ**: Kết quả trả về là **0**.
    *   Nếu số lượng bit 1 trong `x` là **số chẵn**: Kết quả trả về là **1**. (Lưu ý: `val` khởi tạo bằng 1 nên kết quả bị đảo ngược so với parity thông thường).

---

### IDA Pro Insights (Kỹ năng nhận diện thuật toán Bit)

1.  **Mẫu hình XOR tích lũy:** Trong IDA, khi bạn thấy một thanh ghi (như `rax`) thực hiện XOR với một thanh ghi khác (`rdi`) liên tục bên trong một vòng lặp dịch bit (`shr`), đó là dấu hiệu kinh điển của các thuật toán:
    *   Tính Parity.
    *   Tính Checksum đơn giản.
    *   Các bước trong thuật toán mã hóa/hashing.
2.  **Lệnh `and eax, 1` ở cuối:** Đây là cách cực kỳ phổ biến để kiểm tra một số là chẵn hay lẻ, hoặc đơn giản là trích xuất giá trị của một flag Boolean từ một thanh ghi tích lũy dữ liệu.
3.  **Graph View nhận diện cấu trúc:** Nhìn vào Graph View của bài này, mũi tên xanh dương từ khối khởi tạo "đâm" vào khối kiểm tra ở dưới cùng trước khi mũi tên xanh lá quật ngược lên khối xử lý ở giữa. Đây chính là hình ảnh "vân tay" của Jump-to-middle mà bạn nên ghi nhớ để nhận diện nhanh cấu trúc `while` trong IDA.

---

### For Loops (Vòng lặp For)

Cấu trúc tổng quát của một vòng lặp `for` trong C như sau:

```c
for (init-expr; test-expr; update-expr)
    body-statement
```

Theo chuẩn ngôn ngữ C (với một ngoại lệ duy nhất sẽ được nêu ở Bài tập 3.29), hành vi của vòng lặp này hoàn toàn giống hệt với việc sử dụng vòng lặp `while` như sau:

```c
init-expr;
while (test-expr) {
    body-statement
    update-expr;
}
```

Chương trình trước tiên đánh giá biểu thức khởi tạo `init-expr`. Sau đó, nó đi vào một vòng lặp nơi đầu tiên đánh giá điều kiện kiểm tra `test-expr`. Nếu kiểm tra thất bại, vòng lặp kết thúc; nếu thành công, chương trình thực thi thân vòng lặp `body-statement` và cuối cùng đánh giá biểu thức cập nhật `update-expr`.

---

### Các chiến lược dịch mã máy cho vòng lặp For

Dựa trên việc chuyển đổi sang `while`, trình biên dịch GCC sẽ áp dụng một trong hai chiến lược tùy thuộc vào mức độ tối ưu hóa:

#### 1. Chiến lược Jump-to-Middle (Nhảy vào giữa)
GCC thường sử dụng cách này ở mức tối ưu hóa thấp (`-Og`). Mã logic `goto` sẽ trông như sau:

```c
    init-expr;
    goto test;
loop:
    body-statement
    update-expr;
test:
    t = test-expr;
    if (t)
        goto loop;
```

#### 2. Chiến lược Guarded-do (Do-while có bảo vệ)
GCC sử dụng cách này ở các mức tối ưu hóa cao hơn (ví dụ `-O1`). Cấu trúc được biến đổi để kiểm tra điều kiện trước khi vào vòng lặp `do-while`:

```c
    init-expr;
    t = test-expr;
    if (!t)
        goto done;
loop:
    body-statement
    update-expr;
    t = test-expr;
    if (t)
        goto loop;
done:
```

---

### Ví dụ thực tế: Hàm Giai thừa dùng vòng lặp For (`fact_for`)

Xét hàm tính giai thừa được viết bằng vòng lặp `for`:

```c
long fact_for(long n)
{
    long i;
    long result = 1;
    for (i = 2; i <= n; i++)
        result *= i;
    return result;
}
```

Trong ví dụ này:
*   `init-expr`: `i = 2`
*   `test-expr`: `i <= n`
*   `update-expr`: `i++`
*   `body-statement`: `result *= i`

---

### IDA Pro Insights (Cách nhận diện vòng lặp For)

Trong IDA Pro, vòng lặp `for` sẽ không hiển thị từ khóa "for", bạn phải tự nhận diện các thành phần của nó thông qua cấu trúc Graph View:

1.  **Phần Khởi tạo (`init-expr`):** Nằm ở khối lệnh ngay phía trên vòng lặp (ví dụ: `mov ecx, 2`).
2.  **Phần Cập nhật (`update-expr`):** Trong IDA, lệnh này thường nằm ở **cuối khối lệnh thân vòng lặp**, ngay trước lệnh nhảy ngược lên đầu. Hãy tìm các lệnh như `add eax, 1` hoặc `inc rbx` nằm lẻ loi ở cuối block.
3.  **Phần Kiểm tra (`test-expr`):**
    *   Nếu là **Jump-to-middle**: Khối khởi tạo sẽ có mũi tên xanh dương nhảy xuống một khối kiểm tra ở dưới cùng.
    *   Nếu là **Guarded-do**: Sẽ có một khối kiểm tra "lớp bảo vệ" ngay sau khối khởi tạo, nếu sai sẽ nhảy vọt ra khỏi hàm (mũi tên đỏ).
4.  **Cấu trúc 3 thành phần:** Khi dịch ngược, nếu bạn thấy một thanh ghi được khởi tạo, sau đó được so sánh ở điều kiện lặp, và được tăng/giảm ở cuối thân vòng lặp, hãy mạnh dạn viết lại nó thành cấu trúc `for (i = ...; i < ...; i++)` trong C để code trông gọn gàng hơn.

---

### Phân tích các thành phần của vòng lặp For

Như đã trình bày, cách tự nhiên để viết hàm giai thừa với vòng lặp `for` là nhân các thừa số từ 2 đến $n$. Các thành phần của vòng lặp trong đoạn mã `fact_for` được xác định như sau:

*   **`init-expr`** (Khởi tạo): `i = 2`
*   **`test-expr`** (Kiểm tra): `i <= n`
*   **`update-expr`** (Cập nhật): `i++`
*   **`body-statement`** (Thân vòng lặp): `result *= i;`

---

### Chuyển đổi sang vòng lặp While tương đương

Thay thế các thành phần trên vào khuôn mẫu (template) chuyển đổi từ `for` sang `while`, ta có hàm `fact_for_while`:

```c
long fact_for_while(long n)
{
    long i = 2;
    long result = 1;
    while (i <= n) {
        result *= i;
        i++;
    }
    return result;
}
```

---

### Chuyển đổi sang dạng Goto (Jump-to-Middle)

Tiếp tục áp dụng chiến lược **Jump-to-middle** (thường thấy ở mức tối ưu hóa `-Og`) cho vòng lặp `while` ở trên, chúng ta thu được phiên bản logic `goto` mô phỏng sát mã máy:

```c
long fact_for_jm_goto(long n)
{
    long i = 2;
    long result = 1;
    goto test;  // Nhảy vào giữa (điểm kiểm tra)
loop:
    result *= i;
    i++;
test:
    if (i <= n)
        goto loop; // Nếu đúng thì quay lại thân vòng lặp
    return result;
}
```

---

### IDA Pro Insights (Cách nhận diện logic For)

Khi bạn gặp một vòng lặp trong IDA Pro được biên dịch từ `for`, hãy chú ý các đặc điểm logic sau:

1.  **Thanh ghi chỉ số (Index Register):** Trong ví dụ này, `i` thường sẽ là một thanh ghi như `%rcx` hoặc `%rdx`. Bạn sẽ thấy nó được nạp giá trị `2` ngay trước vòng lặp.
2.  **Lệnh cập nhật nằm ở cuối:** Trong Graph View, khối lệnh "thân vòng lặp" (nhãn `loop`) sẽ kết thúc bằng lệnh tăng chỉ số (`inc` hoặc `add ..., 1`) trước khi chạy thẳng vào khối kiểm tra. 
3.  **Vị trí của `update-expr`:** Một sai lầm phổ biến khi mới đọc Assembly là nghĩ rằng mọi thứ viết trong `for(...)` của C sẽ nằm cạnh nhau trong mã máy. Thực tế:
    *   `init-expr` nằm **phía trên** vòng lặp.
    *   `test-expr` nằm ở **khối kiểm tra** (thường là khối cuối cùng của vòng lặp).
    *   `update-expr` nằm ở **cuối khối thân vòng lặp**.
4.  **Tư duy Mapping:** Khi thấy cấu trúc:
    *   `Khởi tạo biến X`
    *   `Nhảy tới Kiểm tra`
    *   `Thân vòng lặp (Làm việc -> Thay đổi X)`
    *   `Kiểm tra X`
    $\rightarrow$ Bạn hãy mạnh dạn gom nó lại thành `for (X = ...; X < ...; X++)` khi viết lại mã giả (pseudocode).

---

### Phân tích mã Assembly thực tế của hàm `fact_for`

Khi biên dịch hàm `fact_for` bằng lệnh `gcc -Og`, mã máy được tạo ra tuân thủ chính xác theo khuôn mẫu **Jump-to-middle**:

**Ánh xạ thanh ghi:**
*   `n` nằm trong `%rdi` (Tham số đầu tiên).
*   `result` nằm trong `%rax` (Thanh ghi kết quả).
*   `i` nằm trong `%rdx` (Biến đếm).

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `fact_for:` | `fact_for:` | Nhãn hàm. |
| 2 | `movl $1, %eax` | `mov eax, 1` | `result = 1` |
| 3 | `movl $2, %edx` | `mov edx, 2` | `i = 2` (Khởi tạo `init-expr`) |
| 4 | **`jmp .L8`** | **`jmp short loc_test`** | **Nhảy vào giữa** (tới phần kiểm tra). |
| 5 | `.L9:` | `loc_loop:` | **Thân vòng lặp (Body)** |
| 6 | `imulq %rdx, %rax` | `imul rax, rdx` | `result *= i` |
| 7 | `addq $1, %rdx` | `add rdx, 1` | `i++` (Cập nhật `update-expr`) |
| 8 | `.L8:` | `loc_test:` | **Phần kiểm tra (Test)** |
| 9 | `cmpq %rdi, %rdx` | `cmp rdx, rdi` | So sánh `i` với `n`. |
| 10 | **`jle .L9`** | **`jle short loc_loop`** | **Nếu i <= n, nhảy ngược lên Body.** |
| 11 | `rep; ret` | `retn` | Trả về `result`. |

---

### Practice Problem 3.27: Chuyển đổi Fibonacci sang Guarded-do

<details>
<summary><b>Nhấn để xem lời giải viết mã Goto (Guarded-do)</b></summary>

**Đề bài:** Viết mã `goto` cho hàm `fibonacci` sử dụng vòng lặp `while` và áp dụng phép biến đổi **Guarded-do**.

**Mã logic chuyển đổi:**
```c
long fibonacci(long n) {
    long i = 2;
    long a = 0;
    long b = 1;
    long next;

    // Lớp bảo vệ (Guard) của vòng lặp while
    if (!(i <= n))
        goto done;

loop:
    // Thân vòng lặp
    next = a + b;
    a = b;
    b = next;
    i++;

    // Kiểm tra để lặp (Giống do-while)
    if (i <= n)
        goto loop;

done:
    return b;
}
```
*Ghi chú:* Phương pháp Guarded-do biến `while` thành một cấu trúc `if` bao ngoài một vòng lặp `do-while`.

</details>

---

### Tổng kết về Vòng lặp trong Mã máy

Từ các ví dụ đã học, chúng ta thấy rằng cả ba dạng vòng lặp trong C (`do-while`, `while`, và `for`) đều có thể được dịch sang mã máy bằng các chiến lược đơn giản. 
*   Bản chất của chúng là tạo ra các đoạn mã chứa một hoặc nhiều **lệnh nhảy có điều kiện (conditional branches)**. 
*   **Chuyển giao điều khiển có điều kiện (Conditional transfer of control)** là cơ chế cơ bản nhất và duy nhất để hiện thực hóa vòng lặp ở mức máy.

---

### IDA Pro Insights (Nhận diện "vân tay" của For)

1.  **Thứ tự thực thi thực tế:** Khi nhìn vào Graph View của IDA cho một vòng lặp `for`, thứ tự bạn sẽ thấy là:
    *   `Khối nạp hằng số` (Init: `i=2`, `result=1`).
    *   `Mũi tên nhảy vọt` (JMP tới Test).
    *   `Khối tính toán` (Body: `imul`, `add`).
    *   `Khối điều kiện` (Test: `cmp`, `jle`).
2.  **Sự biến mất của biến `i`:** Trong các mức tối ưu hóa cao, nếu biến đếm `i` chỉ được dùng để đếm số lần lặp mà không tham gia vào tính toán trong thân vòng lặp, IDA Pro có thể cho bạn thấy trình biên dịch đã đổi nó thành lệnh **`dec`** (giảm dần về 0) để tận dụng flag **ZF**, giúp mã gọn hơn.
3.  **Mẹo dịch ngược:** Nếu thấy một cấu trúc có "Lớp bảo vệ" (mũi tên đỏ nhảy thoát ngay từ đầu) và một mũi tên xanh quay ngược (lặp), hãy ưu tiên viết lại thành `for` hoặc `while` trong C để logic dễ hiểu nhất.

---

### Practice Problem 3.28: Nghịch đảo mã vòng lặp For phức tạp

**Đề bài:** Hoàn thiện mã nguồn C và giải mã chức năng của hàm `test_two` dựa trên mã Assembly.

#### 1. Phân tích mã Assembly (Ánh xạ ATT và Intel/IDA Pro)
*Quy ước: x nằm trong %rdi, val nằm trong %rax, i nằm trong %rdx*

| Dòng | AT&T Syntax | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 2 | `movl $0, %edx` | `mov edx, 0` | Khởi tạo biến đếm **`i = 0`**. |
| 3 | `movl $0, %eax` | `mov eax, 0` | Khởi tạo **`val = 0`**. |
| 4 | `.L10:` | `loc_loop:` | **Nhãn thân vòng lặp**. |
| 5 | `movq %rdi, %rcx` | `mov rcx, rdi` | `temp = x`. |
| 6 | `andl $1, %ecx` | `and ecx, 1` | `temp = x & 1` (Lấy bit cuối cùng). |
| 7 | `addq %rax, %rax` | `add rax, rax` | **`val <<= 1`** (Dịch trái val 1 bit). |
| 8 | `orq %rcx, %rax` | `or rax, rcx` | **`val |= temp`** (Nạp bit của x vào val). |
| 9 | `shrq %rdi` | `shr rdi, 1` | **`x >>= 1`** (Dịch phải logic x 1 bit). |
| 10 | `addq $1, %rdx` | `add rdx, 1` | **`i++`** (Tăng biến đếm). |
| 11 | `cmpq $64, %rdx` | `cmp rdx, 40h` | So sánh `i` với 64. |
| 12 | `jne .L10` | `jnz short loc_loop`| **Nếu i != 64, tiếp tục lặp**. |

---

<details>
<summary><b>Nhấn để xem lời giải chi tiết cho Problem 3.28</b></summary>

**A. Điền vào các phần còn thiếu của mã C:**
```c
short test_two(unsigned short x) {
    short val = 0;
    short i;
    for (i = 0; i < 64; i++) {
        val = (val << 1) | (x & 1);
        x >>= 1;
    }
    return val;
}
```

**B. Tại sao không có lệnh kiểm tra ban đầu (initial test)?**
*   **Lý do:** Trình biên dịch nhận thấy số lần lặp là cố định (64 lần), và 64 chắc chắn lớn hơn 0. Do đó, vòng lặp chắc chắn sẽ được thực thi ít nhất một lần. Trình biên dịch đã tối ưu bằng cách sử dụng cấu trúc **do-while** (kiểm tra ở cuối) để loại bỏ một lệnh nhảy không cần thiết ở đầu hàm.

**C. Chức năng của hàm này:**
*   Hàm này thực hiện việc **đảo ngược thứ tự các bit (Bit Reversal)** của một số nguyên. Nó duyệt qua từng bit của `x` từ phải sang trái và đẩy chúng vào `val` từ trái sang phải.

</details>

---

### Practice Problem 3.29: Lệnh Continue trong vòng lặp For

**Đề bài:** Phân tích sự khác biệt giữa `for` và `while` khi có lệnh `continue`.

```c
/* Tính tổng các số lẻ từ 0 đến 9 */
long sum = 0;
long i;
for (i = 0; i < 10; i++) {
    if (i & 1) // Nếu là số lẻ
        continue;
    sum += i;
}
```

---

<details>
<summary><b>Nhấn để xem phân tích lỗi logic khi chuyển đổi</b></summary>

**A. Chuyện gì xảy ra nếu áp dụng quy tắc chuyển đổi "ngây thơ" sang `while`?**
Nếu ta dịch một cách máy móc:
```c
long sum = 0;
long i = 0;
while (i < 10) {
    if (i & 1)
        continue; // Lệnh này sẽ nhảy thẳng lên kiểm tra (i < 10)
    sum += i;
    i++; // Lệnh cập nhật bị bỏ qua!
}
```
*   **Lỗi:** Khi gặp số lẻ đầu tiên (`i=1`), lệnh `continue` sẽ nhảy vọt lên kiểm tra điều kiện `while`, bỏ qua lệnh `i++`. Biến `i` sẽ mãi mãi bằng 1, gây ra **vòng lặp vô tận**.

**B. Cách thay thế `continue` bằng `goto` để đúng logic của vòng lặp `for`:**
Trong vòng lặp `for`, lệnh `continue` phải nhảy đến phần **cập nhật biến đếm** (`update-expr`) chứ không phải phần kiểm tra điều kiện.

```c
    long sum = 0;
    long i = 0;
    while (i < 10) {
        if (i & 1)
            goto update;
        sum += i;
    update:
        i++;
    }
```

</details>

---

### IDA Pro Insights (Cách nhận diện lệnh Continue)

1.  **Mũi tên trong Graph View:** 
    *   Trong một vòng lặp bình thường, hầu hết các mũi tên sẽ dẫn đến cuối thân vòng lặp. 
    *   Nếu có lệnh `continue`, bạn sẽ thấy một mũi tên xuất phát từ giữa thân vòng lặp, **nhảy qua** các lệnh tính toán phía dưới, nhưng lại **đâm vào** ngay trước lệnh tăng biến đếm (`inc` hoặc `add`).
2.  **Phân biệt với `break`:**
    *   Lệnh `break` sẽ có mũi tên nhảy thoát hoàn toàn ra khỏi cấu trúc vòng lặp.
    *   Lệnh `continue` vẫn nằm trong phạm vi vòng lặp nhưng nhảy đến phần "đuôi" (phần cập nhật).
3.  **Sự tối ưu của Compiler:** Trình biên dịch rất hiếm khi để xảy ra lỗi như bài 3.29. Khi bạn thấy một lệnh nhảy dẫn tới phần `update` của vòng lặp, hãy chắc chắn đó là một lệnh `continue` trong mã nguồn C.

---

## 3.6.8 Switch Statements (Câu lệnh Switch)

Câu lệnh `switch` cung cấp khả năng rẽ nhánh nhiều hướng dựa trên giá trị của một chỉ số số nguyên. Chúng đặc biệt hữu ích khi xử lý các phép thử có số lượng kết quả đầu ra rất lớn.

### Ưu điểm vượt trội của Jump Table (Bảng nhảy)
*   **Bản chất:** Jump Table là một mảng (array) chứa địa chỉ của các đoạn mã máy. Phần tử thứ $i$ trong mảng là địa chỉ của đoạn mã thực thi khi chỉ số `index` bằng $i$.
*   **Hiệu năng $O(1)$:** Thay vì kiểm tra tuần tự bằng chuỗi lệnh `if-else` ($O(n)$), chương trình truy cập trực tiếp vào bảng nhảy bằng chỉ số để tìm địa chỉ đích và thực hiện lệnh nhảy gián tiếp. Thời gian thực thi là **hằng số** ($O(1)$), hoàn toàn độc lập với số lượng các `case`.
*   **Quyết định của GCC:** Trình biên dịch sẽ chọn phương pháp Jump Table khi số lượng `case` đủ lớn (thường là từ **4 cases trở lên**) và dải giá trị giữa các case **không quá thưa thớt** (small range).

---

### Phân tích Ví dụ: Hàm `switch_eg` (Hình 3.22)

Hàm `switch_eg` có các đặc điểm thực tế thú vị:
*   Các giá trị `case` không liên tục: Có giá trị `100`, `102`-`104`, và `106` (thiếu `101` và `105`).
*   Nhiều nhãn trỏ về cùng một khối xử lý: `case 104` và `case 106`.
*   Hành vi trôi tuột (**Fall-through**): `case 102` không có lệnh `break` nên sẽ trôi tuột xuống thực hiện tiếp `case 103`.

#### 1. Các phiên bản mã nguồn C

| (a) Original C code | (b) Translation into extended C (GCC Extension) |
| :--- | :--- |
| ```c void switch_eg(long x, long n, long *dest) { long val = x; switch (n) { case 100: val *= 13; break; case 102: val += 10; /* Fall through */ case 103: val += 11; break; case 104: case 106: val *= val; break; default: val = 0; } *dest = val; } ``` | ```c void switch_eg_impl(long x, long n, long *dest) { // Bảng con trỏ địa chỉ nhãn (GCC && operator) static void *jt[7] = { &&loc_A, &&loc_def, &&loc_B, &&loc_C, &&loc_D, &&loc_def, &&loc_D }; // Dịch dải giá trị: n - 100 unsigned long index = n - 100; long val; // Kiểm tra an toàn: nếu ngoài dải 0-6 if (index > 6) goto loc_def; // Nhảy gián tiếp qua bảng nhảy goto *jt[index]; loc_A: val = x * 13; goto done; loc_B: x = x + 10; /* Fall through */ loc_C: val = x + 11; goto done; loc_D: val = x * x; goto done; loc_def: val = 0; done: *dest = val; } ``` |

---

### 2. Phân tích mã Assembly thực tế (Hình 3.23)

**Ánh xạ thanh ghi:** `x` trong `%rdi`, `n` trong `%rsi`, `dest` trong `%rdx`.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `switch_eg:` | `switch_eg:` | Nhãn hàm. |
| 2 | `subq $100, %rsi` | `sub rsi, 100` | `index = n - 100` (Dịch dải về 0-6). |
| 3 | `cmpq $6, %rsi` | `cmp rsi, 6` | So sánh chỉ số với hằng số 6. |
| 4 | **`ja .L8`** | **`ja short loc_def`** | **Mẹo Unsigned:** Nếu `index > 6` (bao gồm cả số âm sau khi dịch dải), nhảy đến `default`. |
| 5 | **`jmp *.L4(,%rsi,8)`**| **`jmp qword ptr [L4 + rsi*8]`** | **Indirect Jump**: Nhảy tới địa chỉ đọc từ bảng `.L4` tại vị trí `rsi * 8`. |
| 6 | `.L3:` | `loc_A:` | **Case 100:** |
| 7 | `leaq (%rdi,%rdi,2), %rax`| `lea rax, [rdi + rdi*2]` | `rax = 3 * x` |
| 8 | `leaq (%rdi,%rax,4), %rdi`| `lea rdi, [rdi + rax*4]` | `val = x + 4 * (3 * x) = 13 * x` (Mẹo LEA). |
| 9 | `jmp .L2` | `jmp short loc_done` | `break` $\rightarrow$ Nhảy tới kết thúc. |
| 10| `.L5:` | `loc_B:` | **Case 102:** |
| 11| `addq $10, %rdi` | `add rdi, 10` | `x = x + 10` (Không có break, trôi xuống `.L6`). |
| 12| `.L6:` | `loc_C:` | **Case 103:** |
| 13| `addq $11, %rdi` | `add rdi, 11` | `val = x + 11` |
| 14| `jmp .L2` | `jmp short loc_done` | `break;` |
| 15| `.L7:` | `loc_D:` | **Case 104, 106:** |
| 16| `imulq %rdi, %rdi` | `imul rdi, rdi` | `val = x * x` |
| 17| `jmp .L2` | `jmp short loc_done` | `break;` |
| 18| `.L8:` | `loc_def:` | **Default Case:** |
| 19| `movl $0, %edi` | `mov edi, 0` | `val = 0` (Dùng lệnh `movl` 32-bit tối ưu). |
| 20| `.L2:` | `loc_done:` | **Kết thúc:** |
| 21| `movq %rdi, (%rdx)` | `mov [rdx], rdi` | `*dest = val` |
| 22| `ret` | `retn` | Trả về. |

---

### Phân tích chi tiết các kỹ thuật tối ưu hóa trong `switch`:

1.  **Dịch dải giá trị (Range Shifting):**
    *   Các case gốc là 100-106. Trình biên dịch thực hiện `subq $100, %rsi` để chuyển dải giá trị về từ $0$ đến $6$. Việc này giúp bảng nhảy chỉ cần 7 phần tử thay vì tốn 107 phần tử (tiết kiệm bộ nhớ).
2.  **Mẹo kiểm tra Unsigned (`ja`):**
    *   Để kiểm tra xem `index` có nằm ngoài dải $0-6$ hay không, trình biên dịch sử dụng lệnh nhảy **`ja` (Jump if Above - Không dấu)**. 
    *   Nếu `n < 100`, thì `index = n - 100` sẽ là một số âm. Trong biểu diễn số không dấu, số âm bù hai sẽ biến thành một số dương cực kỳ lớn (chắc chắn $> 6$). Do đó, chỉ với đúng một lệnh `ja` ở dòng 4, trình biên dịch đã loại bỏ được cả hai trường hợp: `index < 0` và `index > 6` để nhảy thẳng tới `default`.
3.  **Xử lý Case trống và trùng lặp:**
    *   Trong bảng nhảy `jt[7]`, các chỉ số trống (1 và 5, tương ứng `101` và `105`) đều chứa địa chỉ của `loc_def` (nhánh default).
    *   Các chỉ số trùng nhau (4 và 6, tương ứng `104` và `106`) đều chứa địa chỉ của `loc_D`.
4.  **Mẹo nhân 13 bằng LEA (Dòng 7-8):**
    *   Thay vì dùng lệnh nhân `imulq $13, ...`, trình biên dịch dùng hai lệnh `leaq` liên tiếp: Tính $3x$, sau đó tính $x + 4 \times (3x) = 13x$. Cách này nhanh hơn rất nhiều.

---

### IDA Pro Insights (Cách đọc Jump Table thực tế)

Khi phân tích một cấu trúc `switch` sử dụng Jump Table trong IDA Pro, bạn sẽ thấy nó hỗ trợ cực kỳ mạnh mẽ:

1.  **Lệnh nhảy gián tiếp:** Tìm kiếm lệnh `jmp ds:off_XXXX[rax*8]` (Intel Syntax). Đây chính là hiện thân của lệnh `jmp *.L4(,%rsi,8)`.
2.  **IDA tự tạo Sơ đồ rẽ nhánh:** IDA Pro sẽ tự động nhận diện bảng nhảy trong phân vùng dữ liệu `.rodata`, phân tích địa chỉ của từng nhánh và vẽ ra các mũi tên dẫn thẳng từ khối `jmp` gián tiếp đến từng khối `case` cụ thể trong Graph View. Bạn sẽ thấy một khối lệnh tỏa ra rất nhiều mũi tên (như hình rẻ quạt).
3.  **Bảng Switch Chú thích:** IDA sẽ hiển thị một bảng tóm tắt ngay phía dưới lệnh nhảy gián tiếp, liệt kê rõ:
    *   Địa chỉ của bảng nhảy (Jump Table base).
    *   Các giá trị của case tương ứng với từng địa chỉ (ví dụ: `case 100 -> loc_4005A0`).
    *   Địa chỉ của nhánh Default.
4.  **Fall-through:** Nếu trong Graph View của IDA, bạn thấy khối lệnh của `Case A` có một mũi tên chạy thẳng tuột xuống khối lệnh của `Case B` mà không qua lệnh so sánh hay rẽ nhánh nào, đó chính là hành vi **Fall-through** (thiếu `break` trong mã nguồn C).

---

### Khai báo Jump Table trong Assembly

Trong mã assembly, bảng nhảy được định nghĩa bằng các chỉ thị dữ liệu. Dưới đây là cách bảng nhảy `.L4` của ví dụ trước được tổ chức trong bộ nhớ:

```assembly
1   .section    .rodata         ; Chuyển sang phân vùng dữ liệu chỉ đọc
2   .align 8                    ; Căn chỉnh địa chỉ là bội số của 8
3   .L4:                        ; Nhãn bắt đầu bảng nhảy
4   .quad   .L3                 ; Chỉ số 0 (Case 100): loc_A
5   .quad   .L8                 ; Chỉ số 1 (Case 101): loc_def (nhánh mặc định)
6   .quad   .L5                 ; Chỉ số 2 (Case 102): loc_B
7   .quad   .L6                 ; Chỉ số 3 (Case 103): loc_C
8   .quad   .L7                 ; Chỉ số 4 (Case 104): loc_D
9   .quad   .L8                 ; Chỉ số 5 (Case 105): loc_def (nhánh mặc định)
10  .quad   .L7                 ; Chỉ số 6 (Case 106): loc_D
```

**Phân tích kỹ thuật:**
*   **`.rodata` (Read-Only Data):** Bảng nhảy được đặt trong phân vùng dữ liệu chỉ đọc của file thực thi. Đây là nơi chứa các hằng số không thay đổi khi chương trình chạy.
*   **`.quad`:** Chỉ thị khai báo một đại lượng 8-byte (64-bit). Mỗi mục trong bảng nhảy thực chất là một **địa chỉ bộ nhớ** (con trỏ mã lệnh) dẫn tới khối lệnh xử lý `case` tương ứng.
*   **Xử lý Case thiếu:** Các chỉ số 1 (`101`) và 5 (`105`) không có trong mã C, nên trình biên dịch điền địa chỉ của nhãn `.L8` (nhánh `default`) vào đó.
*   **Xử lý Case trùng:** Các chỉ số 4 (`104`) và 6 (`106`) đều trỏ về cùng một nhãn `.L7`, vì chúng dùng chung logic xử lý trong mã C.

---

### Cách thực hiện các nhánh và Fall-through

Các khối mã khác nhau (từ `loc_A` đến `loc_D` và `loc_def`) hiện thực hóa các nhánh của câu lệnh `switch`. 
*   **Cấu trúc chung:** Hầu hết các khối sẽ tính toán giá trị cho biến `val` và sau đó dùng lệnh nhảy `jmp .L2` để kết thúc (tương ứng lệnh `break` trong C).
*   **Xử lý Fall-through (Trôi tuột):** Riêng **Case 102** (nhãn `.L5`) không kết thúc bằng lệnh nhảy. Sau khi thực hiện xong logic của mình, bộ xử lý sẽ tự động thực thi tiếp lệnh của khối ngay bên dưới nó (`loc_C`). 
    *   Đây chính là cách mã máy thực hiện hành vi "fall-through" khi lập trình viên quên (hoặc cố ý không dùng) lệnh `break`.

---

### Hiệu năng của Bảng nhảy

Điểm mấu chốt cần thấy là việc sử dụng bảng nhảy cho phép thực hiện rẽ nhánh đa hướng một cách cực kỳ hiệu quả. 
*   Chương trình có thể nhảy tới bất kỳ vị trí nào trong 5 địa chỉ khác nhau chỉ bằng **duy nhất một lần truy xuất** vào bảng nhảy. 
*   Ngay cả khi câu lệnh `switch` có hàng trăm trường hợp, thời gian để tìm đến đúng `case` vẫn chỉ tốn một thao tác đọc bộ nhớ và một lệnh nhảy gián tiếp.

---

### IDA Pro Insights (Cách nhìn bảng nhảy trong Segment dữ liệu)

Trong IDA Pro, việc nhận diện bảng nhảy là chìa khóa để hiểu cấu trúc hàm phức tạp:

1.  **Phân vùng dữ liệu:** Bạn có thể tìm thấy bảng nhảy này bằng cách nhấn vào địa chỉ trong lệnh `jmp ds:off_XXXX`. IDA sẽ dẫn bạn đến phân vùng **CONST** hoặc **.rdata**. Tại đó, bảng sẽ hiển thị dưới dạng:
    ```assembly
    off_401234  dq offset loc_4005A0    ; Case 100
                dq offset loc_4005F0    ; Case 101 (Default)
                dq offset loc_4005B0    ; Case 102
                ...
    ```
2.  **dq (Define Quadword):** Tương đương với `.quad` trong sách, chỉ định địa chỉ 64-bit.
3.  **Nhận diện Fall-through:** Trong chế độ Graph View của IDA, nếu bạn thấy một khối lệnh có **mũi tên xanh dương** chạy thẳng xuống khối ngay dưới mà không có bất kỳ lệnh nhảy (`jmp`, `jz`...) nào ở cuối, đó chắc chắn là một lỗi thiếu `break` hoặc logic fall-through có chủ đích.
4.  **Sự thông minh của IDA:** IDA thường tạo ra một bảng chú thích (comment) màu xanh lá cây ngay phía sau lệnh nhảy gián tiếp, liệt kê toàn bộ các `case` mà bảng này hỗ trợ, giúp bạn không cần phải tự dò từng địa chỉ trong phân vùng dữ liệu.

---

### Practice Problem 3.30: Giải mã Bảng nhảy và Giá trị Case

**Mã nguồn C (Khung xương):**
```c
void switch2(short x, short *dest) {
    short val = 0;
    switch (x) {
        /* Thân của câu lệnh switch đã bị lược bỏ */
    }
    *dest = val;
}
```

#### 1. Phân tích mã Assembly khởi đầu:
*Quy ước: x ban đầu nằm trong %rdi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `addq $2, %rdi` | `add rdi, 2` | **Dịch dải:** `index = x + 2`. |
| 2 | `cmpq $8, %rdi` | `cmp rdi, 8` | So sánh chỉ số với hằng số 8. |
| 3 | `ja .L2` | `ja short loc_default` | Nếu `index > 8` (không dấu), nhảy tới `default`. |
| 4 | `jmp *.L4(,%rdi,8)` | `jmp qword ptr [L4 + rdi*8]` | Nhảy gián tiếp qua bảng nhảy `.L4`. |

---

#### 2. Phân tích Bảng nhảy (Jump Table `.L4`):

Dựa vào lệnh `addq $2, %rdi`, ta biết công thức tính chỉ số là: $index = x + 2 \Rightarrow x = index - 2$.
Bảng có 9 mục (từ chỉ số 0 đến 8):

| Index | Thanh ghi `%rdi` | Giá trị `x` gốc ($index - 2$) | Địa chỉ nhảy tới | Phân loại |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 0 | **-2** | `.L9` | Case -2 |
| 1 | 1 | **-1** | `.L5` | Case -1 |
| 2 | 2 | **0** | `.L6` | Case 0 |
| 3 | 3 | **1** | `.L7` | Case 1 |
| 4 | 4 | **2** | `.L2` | Mất (Default) |
| 5 | 5 | **3** | `.L7` | Case 3 |
| 6 | 6 | **4** | `.L8` | Case 4 |
| 7 | 7 | **5** | `.L2` | Mất (Default) |
| 8 | 8 | **6** | `.L5` | Case 6 |

---

### Trả lời câu hỏi:

**A. Các giá trị của nhãn case trong câu lệnh switch là gì?**
Dựa vào bảng trên, các giá trị `x` có nhãn xử lý riêng (không phải nhảy tới `.L2` - default) là:
**-2, -1, 0, 1, 3, 4, 6**.

**B. Những case nào có nhiều nhãn (multiple labels) trong mã C?**
Những giá trị `x` khác nhau nhưng nhảy tới cùng một nhãn assembly:
*   Nhãn **`.L5`**: Ứng với các case **-1** và **6**.
*   Nhãn **`.L7`**: Ứng với các case **1** và **3**.

---

### IDA Pro Insights (Cách đọc logic Switch-case)

1.  **Xác định dải giá trị (Range):** 
    *   Lệnh `add` ngay đầu cho bạn biết giá trị case nhỏ nhất (x + 2 = 0 $\rightarrow$ x_min = -2).
    *   Lệnh `cmp` cho bạn biết độ rộng của dải (9 giá trị: từ 0 đến 8).
    *   Trong IDA, bảng chú thích switch sẽ ghi rõ: `Cases -2 to 6`.
2.  **Nhận diện Case bị khuyết:** 
    *   Trong bảng nhảy trên, tại index 4 và 7 (tương ứng case 2 và 5), địa chỉ nhảy là `.L2`. Vì `.L2` cũng là địa chỉ của nhánh `default` (theo lệnh `ja .L2`), ta kết luận trong mã C gốc **không có** `case 2` và `case 5`.
3.  **Nhãn trùng lặp:** 
    *   Khi bạn thấy IDA vẽ hai mũi tên từ cùng một bảng nhảy dẫn đến cùng một khối lệnh (Basic Block), đó là biểu hiện của mã C dạng:
      ```c
      case 1:
      case 3:
          /* chung code xử lý */
          break;
      ```
4.  **Kiểu dữ liệu Index:** 
    *   Lưu ý lệnh `addq` và `cmpq` (8-byte) được dùng dù `x` là `short` (2-byte). Trình biên dịch thường mở rộng kích thước biến index lên 64-bit để thực hiện phép tính địa chỉ `[base + index * 8]` một cách an toàn.

---

### Practice Problem 3.31: Hoàn thiện hàm `switcher` từ mã Assembly

**Đề bài:** Dựa vào mã Assembly và bảng nhảy (Figure 3.24), hãy điền các giá trị và biểu thức còn thiếu vào mã nguồn C.

#### 1. Ánh xạ tham số (Register Mapping)
Theo chuẩn System V ABI cho x86-64, các tham số được ánh xạ như sau:
*   `a` (1st param): **`%rdi`**
*   `b` (2nd param): **`%rsi`**
*   `c` (3rd param): **`%rdx`**
*   `dest` (4th param): **`%rcx`**

*Lưu ý:* Mã máy sử dụng `%rdi` để lưu `val` sau khi đã thực hiện xong lệnh `switch(a)`.

---

#### 2. Giải mã Bảng nhảy (Jump Table `.L4`)
Lệnh `jmp *.L4(,%rdi,8)` nhảy dựa trên giá trị của `a` (0 đến 7):

| Chỉ số (a) | Nhãn đích | Khối mã xử lý | Phân tích logic |
| :--- | :--- | :--- | :--- |
| **0** | `.L3` | **Case B** | `val = c + 112` |
| **1** | `.L2` | **Default** | `val = b` |
| **2** | `.L5` | **Case C/D** | `val = (c + b) << 2` |
| **3** | `.L2` | **Default** | `val = b` |
| **4** | `.L6` | **Case E** | `val = a` (nhảy thẳng tới điểm lưu kết quả) |
| **5** | `.L7` | **Case A** | `c = b ^ 15`, trôi xuống **Case B** |
| **6** | `.L2` | **Default** | `val = b` |
| **7** | `.L5` | **Case C/D** | `val = (c + b) << 2` |

---

#### 3. Phân tích các khối mã (Code Blocks)

*   **Khối `.L7` (Case A):**
    *   `xorq $15, %rsi`: `b = b ^ 15`
    *   `movq %rsi, %rdx`: `c = b`
    *   *Fall through* (trôi xuống `.L3`).
*   **Khối `.L3` (Case B):**
    *   `leaq 112(%rdx), %rdi`: `val = c + 112`
    *   `jmp .L6`: `break;`
*   **Khối `.L5` (Case C & D):**
    *   `leaq (%rdx,%rsi), %rdi`: `temp = c + b`
    *   `salq $2, %rdi`: `val = temp << 2` (tương đương `(c + b) * 4`)
    *   `jmp .L6`: `break;`
*   **Khối `.L2` (Default):**
    *   `movq %rsi, %rdi`: `val = b`
    *   *Trôi xuống `.L6`*
*   **Khối `.L6` (Kết thúc):**
    *   `movq %rdi, (%rcx)`: `*dest = val`

---

### Lời giải mã C hoàn chỉnh

```c
void switcher(long a, long b, long c, long *dest)
{
    long val;
    switch(a) {
        case 5:              /* Case A */
            c = b ^ 15;
            /* Fall through */
        case 0:              /* Case B */
            val = c + 112;
            break;
        case 2:              /* Case C */
        case 7:              /* Case D */
            val = (c + b) * 4;
            break;
        case 4:              /* Case E */
            val = a;
            break;
        default:
            val = b;
    }
    *dest = val;
}
```

---

### IDA Pro Insights (Kỹ năng Reverse Switch-case)

1.  **Xác định biến Switch:** IDA sẽ hiển thị `jmp ds:off_XXXX[rdi*8]`. Nhìn vào thanh ghi nhân với 8 (`rdi`), bạn biết ngay đó là biến nằm trong `switch()`.
2.  **Nhận diện Fall-through:** 
    *   Trong mã Assembly trên, sau nhãn `.L7` **không có** lệnh `jmp`. 
    *   Trong IDA Graph View, bạn sẽ thấy một mũi tên chạy thẳng từ khối `.L7` xuống khối `.L3`. Đây là bằng chứng cho việc Case 5 trôi vào Case 0.
3.  **Tối ưu hóa Case E:** 
    *   Case 4 nhảy thẳng đến `.L6`. Tại `.L6`, lệnh `movq %rdi, (%rcx)` được thực thi. Vì lúc này `%rdi` vẫn đang giữ giá trị gốc của `a` (là 4), nên thực tế `val = 4`. 
    *   Trong IDA, điều này thường làm code rẽ nhánh trông rất gọn vì một `case` không cần khối xử lý riêng.
4.  **Mẹo kiểm tra nhanh:** Khi `a > 7` (lệnh `ja .L2` ở dòng 3), chương trình nhảy tới `.L2`. Vì vậy các chỉ số 1, 3, 6 (trỏ tới `.L2` trong bảng) và các giá trị nằm ngoài dải 0-7 đều thuộc về `default`.

---

## 3.7 Procedures (Thủ tục)

Thủ tục (Procedures) là một sự trừu tượng hóa then chốt trong phần mềm. Chúng cung cấp một cách để đóng gói mã thực thi một chức năng cụ thể với một tập hợp các đối số (arguments) và một giá trị trả về (return value) tùy chọn. Hàm này sau đó có thể được gọi từ nhiều điểm khác nhau trong chương trình. 

Phần mềm được thiết kế tốt sử dụng các thủ tục như một cơ chế che giấu thông tin (abstraction mechanism), ẩn đi chi tiết cài đặt của một hành động cụ thể trong khi cung cấp một định nghĩa giao diện rõ ràng và ngắn gọn về những giá trị nào sẽ được tính toán và những tác động nào mà thủ tục đó sẽ gây ra đối với trạng thái chương trình. 

Thủ tục xuất hiện dưới nhiều tên gọi khác nhau trong các ngôn ngữ lập trình: **functions** (hàm), **methods** (phương thức), **subroutines** (hàm con), **handlers** (trình xử lý)... nhưng chúng đều chia sẻ một tập hợp các đặc điểm chung.

---

### Các cơ chế hỗ trợ thủ tục mức máy

Giả sử thủ tục **P** gọi thủ tục **Q**, và **Q** thực thi rồi trả về lại cho **P**. Các hành động này liên quan đến một hoặc nhiều cơ chế sau:

1.  **Passing control (Chuyển giao điều khiển):** Thanh ghi con trỏ lệnh (Program Counter - `%rip`) phải được thiết lập thành địa chỉ bắt đầu của mã lệnh thủ tục **Q** khi bắt đầu gọi, và sau đó phải được thiết lập lại thành địa chỉ của lệnh ngay sau lệnh gọi trong **P** khi quay trở về.
2.  **Passing data (Truyền dữ liệu):** **P** phải có khả năng cung cấp một hoặc nhiều tham số cho **Q**, và **Q** phải có khả năng trả về một giá trị cho **P**.
3.  **Allocating and deallocating memory (Cấp phát và giải phóng bộ nhớ):** **Q** có thể cần cấp phát không gian cho các biến cục bộ khi bắt đầu thực thi và giải phóng không gian đó trước khi trả về.

---

### Chiến lược thực hiện trong x86-64

Việc triển khai các thủ tục trong x86-64 bao gồm sự kết hợp của các **chỉ thị đặc biệt** và một tập hợp các **quy ước (conventions)** về cách sử dụng tài nguyên máy (như thanh ghi và bộ nhớ chương trình). 

*   **Sự tối ưu:** Các nhà thiết kế đã nỗ lực rất lớn để giảm thiểu chi phí (overhead) phát sinh khi gọi một thủ tục. 
*   **Chiến lược tối giản:** x86-64 áp dụng chiến lược tối giản, chỉ triển khai các cơ chế thực sự cần thiết cho từng thủ tục cụ thể. 

Trong các phần tiếp theo, chúng ta sẽ xây dựng các cơ chế khác nhau từng bước một: đầu tiên mô tả việc điều khiển luồng (control), sau đó là truyền dữ liệu (data passing), và cuối cùng là quản lý bộ nhớ (memory management).

---

### IDA Pro Insights (Tư duy về Procedures)

Khi bạn mở một file thực thi trong IDA Pro, hầu hết thời gian bạn sẽ làm việc với các "Procedures" này:

*   **Nhận diện hàm:** IDA tự động phân tích các nhãn như `sub_401234` hoặc sử dụng tên thực (như `main`, `printf`) nếu có thông tin ký hiệu (symbols).
*   **Ba cơ chế trong IDA:**
    1.  **Control:** Bạn sẽ thấy lệnh `call` để gọi và `ret` để trả về.
    2.  **Data:** Bạn sẽ thấy dữ liệu được nạp vào `%rdi`, `%rsi`... trước lệnh `call` (tham số) và kết quả nằm trong `%rax` sau lệnh `call`.
    3.  **Memory:** Bạn sẽ thấy các lệnh như `sub rsp, 20h` ở đầu hàm để tạo "Stack Frame" cho biến cục bộ.

---

## 3.7.1 The Run-Time Stack (Ngăn xếp thực thi)

Một đặc điểm then chốt của cơ chế gọi hàm trong C (và hầu hết các ngôn ngữ khác) là việc sử dụng cấu trúc dữ liệu Ngăn xếp (Stack) để quản lý bộ nhớ theo nguyên tắc **Vào sau, Ra trước (LIFO)**.

### Cơ chế hoạt động cơ bản
Giả sử thủ tục **P** gọi thủ tục **Q**:
*   Trong khi **Q** đang chạy, **P** (và các hàm đã gọi P trước đó) tạm thời bị đình chỉ.
*   **Q** chỉ cần khả năng cấp phát không gian mới cho các biến cục bộ của nó hoặc để thiết lập lời gọi đến một thủ tục khác.
*   Khi **Q** trả về, toàn bộ không gian bộ nhớ mà nó đã cấp phát có thể được giải phóng.

### Quản lý Ngăn xếp trong x86-64
*   **Hướng phát triển:** Ngăn xếp phát triển về phía **địa chỉ thấp hơn**.
*   **Thanh ghi `%rsp`:** Luôn trỏ vào phần tử ở đỉnh ngăn xếp (địa chỉ thấp nhất).
*   **Thao tác dữ liệu:** Sử dụng lệnh `pushq` (đẩy vào) và `popq` (lấy ra).
*   **Cấp phát/Giải phóng nhanh:** 
    *   Để cấp phát không gian cho dữ liệu (không cần khởi tạo giá trị): Chỉ cần **giảm** `%rsp` một lượng tương ứng.
    *   Để giải phóng: Chỉ cần **tăng** `%rsp`.

---

### Stack Frame (Khung ngăn xếp)

Khi một thủ tục x86-64 yêu cầu không gian lưu trữ vượt quá những gì các thanh ghi có thể chứa, nó sẽ cấp phát không gian trên ngăn xếp. Vùng không gian này được gọi là **Stack Frame**.

#### Hình 3.25: Cấu trúc tổng quát của một Stack Frame

Hình vẽ mô tả cách phân chia ngăn xếp thành các "khung" cho từng hàm:

<img width="924" height="851" alt="image" src="https://github.com/user-attachments/assets/062056b7-1053-49b6-9807-c403cc897089" />

---

### Các quy tắc đặc thù trong x86-64

1.  **Truyền tham số:** x86-64 có thể truyền tối đa 6 giá trị số nguyên (bao gồm cả con trỏ) qua thanh ghi. Nếu hàm **Q** yêu cầu nhiều hơn 6 tham số, các tham số từ thứ 7 trở đi sẽ được **P** lưu vào khung ngăn xếp của chính nó trước khi gọi **Q**.
2.  **Tính hiệu quả:** Thủ tục chỉ cấp phát những phần của khung ngăn xếp mà chúng thực sự cần. 
3.  **Leaf Procedures (Thủ tục lá):** 
    *   Đây là các hàm không gọi thêm bất kỳ hàm nào khác. 
    *   Nếu tất cả biến cục bộ của hàm này có thể nằm gọn trong các thanh ghi, nó **không cần cấp phát stack frame**.
    *   Tất cả các ví dụ hàm chúng ta đã xem xét từ đầu chương 3 đến giờ đều là các thủ tục lá và không yêu cầu stack frame.

---

### IDA Pro Insights (Kỹ năng đọc Stack Frame)

Khi bạn nhìn vào một hàm trong IDA Pro, bạn sẽ thấy cách nó hiện thực hóa Hình 3.25:

1.  **Prologue (Đoạn đầu hàm):**
    *   `push rbp` và `mov rbp, rsp`: Thiết lập Frame Pointer (nếu không được tối ưu hóa).
    *   `sub rsp, 40h`: Lệnh này chính là thao tác "mở rộng ranh giới ngăn xếp" để tạo không gian cho **Local variables** và **Argument build area**.
2.  **Định danh biến:** 
    *   IDA sẽ ký hiệu các biến có offset dương từ `%rbp` (hoặc phía trên `%rsp` sau khi `sub`) là **`arg_X`**. Đây chính là các tham số từ thứ 7 trở đi nằm trong khung của hàm gọi.
    *   Các biến có offset âm từ `%rbp` (hoặc nằm ngay tại `%rsp`) là **`var_X`**. Đây chính là vùng **Local variables**.
3.  **Return Address:** IDA luôn hiểu ngầm có địa chỉ trả về nằm ngay phía trên vùng biến cục bộ. Nếu bạn thấy một hàm truy cập vào `[rsp + <kích_thước_frame>]`, nó đang chạm vào địa chỉ trả về hoặc tham số trên stack.

---

### Hình 3.26: Minh họa hoạt động của lệnh Call và Ret

Sơ đồ này mô tả quá trình chuyển giao điều khiển giữa hàm `main` và hàm `multstore`.

1.  **Trạng thái (a) - Đang thực thi `call`:**
    *   Thanh ghi `%rip` đang ở địa chỉ `0x400563` (lệnh `call`).
    *   Thanh ghi `%rsp` đang ở `0x7fffffffe840`.
2.  **Trạng thái (b) - Sau khi `call` (vừa vào hàm mới):**
    *   Lệnh `call` đã đẩy **địa chỉ trả về** `0x400568` vào ngăn xếp.
    *   Thanh ghi `%rsp` giảm xuống `0x7fffffffe838` (bớt 8 bytes).
    *   Thanh ghi `%rip` được thiết lập thành `0x400540` (điểm bắt đầu của `multstore`).
3.  **Trạng thái (c) - Sau khi `ret` (quay về hàm gọi):**
    *   Lệnh `ret` đã lấy (pop) giá trị `0x400568` từ đỉnh ngăn xếp và nạp vào `%rip`.
    *   Thanh ghi `%rsp` tăng lại lên `0x7fffffffe840`.
    *   Chương trình tiếp tục thực thi lệnh ngay sau `call`.

---

### Phân tích mã Assembly trích dẫn

Dưới đây là đoạn mã nghịch đảo từ hai hàm liên quan:

| Địa chỉ | Mã máy (Bytes) | ATT Syntax (Sách) | Intel Syntax (IDA Pro) | Ghi chú logic |
| :--- | :--- | :--- | :--- | :--- |
| **Hàm multstore** | | | | |
| `400540:` | `53` | `push %rbx` | `push rbx` | Bắt đầu hàm. |
| ... | ... | ... | ... | ... |
| `40054d:` | `c3` | **`retq`** | **`retn`** | Trả về hàm gọi. |
| **Hàm main** | | | | |
| `400563:` | `e8 d8 ff ff ff` | **`callq 400540`** | **`call sub_400540`** | Gọi hàm `multstore`. |
| `400568:` | `48 8b 54 24 08` | `mov 0x8(%rsp), %rdx` | `mov rdx, [rsp+8]` | **Địa chỉ trả về**. |

---

### Cơ chế kỹ thuật chi tiết

1.  **Lệnh `callq 400540`:**
    *   Địa chỉ của chính lệnh này là `0x400563`.
    *   Địa chỉ của lệnh tiếp theo ngay sau nó là **`0x400568`**. 
    *   Khi thực thi, bộ xử lý đẩy `0x400568` vào Stack và nhảy tới nhãn `<multstore>`.
2.  **Quá trình thực thi hàm được gọi:**
    *   Hàm `multstore` chạy cho đến khi gặp lệnh `retq` tại địa chỉ `0x40054d`.
3.  **Lệnh `retq`:**
    *   Nó đọc giá trị ở đỉnh Stack (lúc này là `0x400568`) và nạp vào thanh ghi `%rip`.
    *   Việc thực thi của hàm `main` được tiếp tục chính xác tại vị trí sau lệnh gọi (dòng 6).

---

### IDA Pro Insights (Cách quan sát lời gọi hàm)

1.  **Nhận diện địa chỉ trả về:** Trong IDA Debugger, khi bạn bước vào (Step Into - F7) một lệnh `call`, hãy nhìn ngay vào thanh ghi `RSP`. Giá trị tại địa chỉ mà `RSP` trỏ tới chính là nơi chương trình sẽ quay về sau khi hàm kết thúc.
2.  **Tên hàm (Labels):** 
    *   Sách ghi: `callq 400540 <multstore>`.
    *   IDA Pro thường hiển thị: `call multstore` (nếu có symbols) hoặc `call sub_400540` (nếu không có). 
    *   Bạn có thể nhấn đúp chuột vào tên hàm trong IDA để nhảy tới định nghĩa của hàm đó.
3.  **Opcode của Call:** Lưu ý mã máy `e8 d8 ff ff ff`. 
    *   `e8` là opcode của lệnh `call` tương đối (PC-relative).
    *   `d8 ff ff ff` là giá trị offset bù hai (số âm), chỉ ra khoảng cách từ lệnh hiện tại tới hàm đích. IDA tự động tính toán khoảng cách này để hiển thị địa chỉ thực `400540`.

---

### Hình 3.27: Phân tích luồng thực thi chi tiết (Execution Trace)

Ví dụ này gồm hai hàm: `top` và `leaf`. Hàm `main` gọi `top`, sau đó `top` gọi `leaf`.

#### (a) Mã Assembly của các hàm (ATT vs Intel/IDA Pro)

| Nhãn | Địa chỉ | ATT Syntax (Sách) | Intel Syntax (IDA Pro) | Logic C tương ứng |
| :--- | :--- | :--- | :--- | :--- |
| **Hàm leaf** | | | | |
| `L1` | `400540` | `leaq 0x2(%rdi), %rax` | `lea rax, [rdi+2]` | `return y + 2` |
| `L2` | `400544` | `retq` | `retn` | |
| **Hàm top** | | | | |
| `T1` | `400545` | `subq $0x5, %rdi` | `sub rdi, 5` | `x - 5` |
| `T2` | `400549` | `callq 400540 <leaf>` | `call leaf` | `leaf(x - 5)` |
| `T3` | `40054e` | `addq %rax, %rax` | `add rax, rax` | `double result` |
| `T4` | `400551` | `retq` | `retn` | |
| **Hàm main** | | | | |
| `M1` | `40055b` | `callq 400545 <top>` | `call top` | `top(100)` |
| `M2` | `400560` | `movq %rax, %rdx` | `mov rdx, rax` | **Địa chỉ trả về** |

---

#### (b) Bảng truy vết thực thi (Execution Trace)

Bảng này mô tả trạng thái của các thanh ghi và ngăn xếp **trước khi** mỗi lệnh được thực thi:

| Nhãn | PC (RIP) | Lệnh thực thi | %rdi | %rax | %rsp | *%rsp (Đỉnh Stack) | Mô tả |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **M1** | `40055b` | `call top` | 100 | — | `...e820` | — | Gọi `top(100)` |
| **T1** | `400545` | `sub $5, %rdi` | 100 | — | `...e818` | `400560` | Vào `top`, Stack lưu `M2` |
| **T2** | `400549` | `call leaf` | 95 | — | `...e818` | `400560` | Chuẩn bị gọi `leaf(95)` |
| **L1** | `400540` | `lea 2(%rdi), %rax` | 95 | — | `...e810` | `40054e` | Vào `leaf`, Stack lưu `T3` |
| **L2** | `400544` | `retq` | 95 | **97** | `...e810` | `40054e` | `leaf` xong, rax = 97 |
| **T3** | `40054e` | `add %rax, %rax` | — | 97 | `...e818` | `400560` | Về `top`, PC lấy từ Stack |
| **T4** | `400551` | `retq` | — | **194** | `...e818` | `400560` | `top` xong, rax = 194 |
| **M2** | `400560` | `mov %rax, %rdx` | — | 194 | `...e820` | — | Về `main`, Stack sạch |

---

### Phân tích vai trò của Ngăn xếp (Stack)

Nội dung bảng truy vết trên chứng minh vai trò sống còn của ngăn xếp thực thi (run-time stack):

1.  **Lưu trữ địa chỉ trả về (Return Address):**
    *   Khi lệnh `L2` (trong `leaf`) thực thi, nó lấy (pop) địa chỉ `0x40054e` từ đỉnh ngăn xếp và nạp vào `%rip`. Nhờ đó, chương trình biết đường quay lại lệnh `T3` của hàm `top`.
    *   Khi lệnh `T4` (trong `top`) thực thi, nó lấy địa chỉ `0x400560` để quay về lệnh `M2` của hàm `main`.
2.  **Tính đối xứng:** 
    *   Bạn có thể thấy thanh ghi `%rsp` giảm xuống khi gọi hàm (`call`) và tăng lên chính xác về giá trị cũ khi trả về (`ret`).
    *   Tại nhãn **M2**, con trỏ ngăn xếp đã được khôi phục về `...e820`, hoàn toàn sạch sẽ như trước khi gọi `top`.
3.  **Cơ chế LIFO:** Các địa chỉ trả về được đẩy vào và lấy ra theo đúng thứ tự "Vào sau, Ra trước", giúp hỗ trợ các lời gọi hàm lồng nhau một cách tự nhiên.

---

### IDA Pro Insights (Kỹ năng Debug hàm lồng nhau)

Khi bạn sử dụng trình Debugger trong IDA Pro (như Local Windows Debugger hoặc GDB):

*   **Step Into (F7) vs Step Over (F8):** 
    *   Tại lệnh `call top` (M1), nếu nhấn **F7**, IDA sẽ đưa bạn tới địa chỉ `400545` (T1). 
    *   Nếu nhấn **F8**, IDA sẽ chạy toàn bộ hàm `top` và dừng lại ở `400560` (M2).
*   **Cửa sổ Call Stack:** Trong lúc đang dừng ở hàm `leaf`, hãy mở cửa sổ **Call Stack**. Bạn sẽ thấy IDA liệt kê:
    1. `leaf` (đang chạy)
    2. `top` (đang đợi)
    3. `main` (đang đợi)
    *   Danh sách này thực chất được IDA trích xuất bằng cách đọc các địa chỉ trả về đang nằm trên Stack (`0x40054e` và `0x400560`).
*   **Mũi tên PC:** Trong Graph View, mũi tên màu vàng chỉ vào lệnh sắp thực thi chính là giá trị hiện tại của thanh ghi `%rip`.

---

### Practice Problem 3.32: Truy vết thực thi lời gọi hàm lồng nhau

**Mã Assembly của các hàm:**

| Nhãn | Địa chỉ | Lệnh (Intel/IDA Pro) | Logic C tương ứng |
| :--- | :--- | :--- | :--- |
| **Hàm `last`** | | `u` in `%rdi`, `v` in `%rsi` | `long last(long u, long v)` |
| `L1` | `400540` | `mov rax, rdi` | `rax = u` |
| `L2` | `400543` | `imul rax, rsi` | `rax = u * v` |
| `L3` | `400547` | `retn` | `return rax` |
| **Hàm `first`** | | `x` in `%rdi` | `long first(long x)` |
| `F1` | `400548` | `lea rsi, [rdi+1]` | `arg2 = x + 1` |
| `F2` | `40054c` | `sub rdi, 1` | `arg1 = x - 1` |
| `F3` | `400550` | `call last` | `last(x-1, x+1)` |
| `F4` | `400555` | `retn` | `return` |
| **Hàm `main`** | | | |
| `M1` | `400560` | `call first` | `first(10)` |
| `M2` | `400565` | `mov rdx, rax` | **Địa chỉ trả về** |

---

### Bảng truy vết thực thi (Execution Trace)

Trạng thái được ghi nhận **ngay trước khi** lệnh tại dòng đó thực thi:

| Nhãn | PC (RIP) | Lệnh thực thi | %rdi | %rsi | %rax | %rsp | *%rsp (Đỉnh Stack) | Mô tả |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **M1** | `0x400560` | `call first` | 10 | — | — | `...e820` | — | `main` gọi `first(10)` |
| **F1** | `0x400548` | `lea 1(%rdi),%rsi` | 10 | — | — | `...e818` | `0x400565` | Vào `first`, lưu `M2` |
| **F2** | `0x40054c` | `sub $1, %rdi` | 10 | **11** | — | `...e818` | `0x400565` | `rsi = 10 + 1 = 11` |
| **F3** | `0x400550` | `call last` | **9** | 11 | — | `...e818` | `0x400565` | `rdi = 10 - 1 = 9` |
| **L1** | `0x400540` | `mov %rdi, %rax` | 9 | 11 | — | `...e810` | `0x400555` | Vào `last`, lưu `F4` |
| **L2** | `0x400543` | `imul %rsi, %rax`| 9 | 11 | **9** | `...e810` | `0x400555` | `rax = 9` |
| **L3** | `0x400547` | `retq` | 9 | 11 | **99** | `...e810` | `0x400555` | `rax = 9 * 11 = 99` |
| **F4** | `0x400555` | `repz retq` | 9 | 11 | 99 | `...e818` | `0x400565` | Về `first`, pop `F4` |
| **M2** | `0x400565` | `mov %rax, %rdx` | 9 | 11 | 99 | `...e820` | — | Về `main`, pop `M2` |

---

### IDA Pro Insights (Phân tích tham số và sự đè chồng)

1.  **Sự thay đổi thanh ghi tham số:**
    *   Hãy để ý ở nhãn **F1** và **F2**. Hàm `first` nhận `x` qua `%rdi`. Để gọi hàm `last(u, v)`, nó cần chuẩn bị `u` vào `%rdi` và `v` vào `%rsi`.
    *   Trình biên dịch tính `v` trước (`rsi = x + 1`) rồi mới ghi đè `x` để tính `u` (`rdi = x - 1`). 
    *   Trong IDA, nếu bạn thấy các thanh ghi tham số (`rdi`, `rsi`, `rdx`...) bị thay đổi liên tục ngay trước một lệnh `call`, đó là bước **chuẩn bị đối số** cho hàm sắp được gọi.

2.  **Địa chỉ trả về lồng nhau:**
    *   Tại thời điểm hàm `last` đang chạy (nhãn **L1, L2**), trên Stack đang có **2 địa chỉ trả về**:
        *   `0x400555` (đỉnh stack): Để quay về `first`.
        *   `0x400565` (nằm ngay trên đó): Để quay về `main`.
    *   IDA Pro hiển thị điều này trong cửa sổ **Call Stack** (Ctrl+Alt+S). Việc hiểu bảng trace này giúp bạn không bị rối khi thấy RSP thay đổi liên tục trong lúc Step Into.

3.  **Lệnh `repz retq`:**
    *   Như đã lưu ý, đây là một biến thể của `ret` (mã máy `f3 c3`). IDA hiển thị nó là `retn`. Nó thực hiện đúng một nhiệm vụ: lấy giá trị từ đỉnh stack đưa vào thanh ghi lệnh PC.

---

## 3.7.3 Data Transfer (Truyền dữ liệu)

Ngoài việc chuyển giao điều khiển, các lời gọi hàm còn bao gồm việc truyền dữ liệu dưới dạng đối số (arguments) và nhận về kết quả (return value). Trong x86-64, phần lớn việc truyền dữ liệu này diễn ra thông qua các thanh ghi để đạt tốc độ tối ưu.

### Quy ước truyền tham số qua thanh ghi

x86-64 có thể truyền tối đa **6 tham số số nguyên** (bao gồm cả con trỏ) thông qua các thanh ghi được chỉ định theo một thứ tự nghiêm ngặt. Tên thanh ghi được sử dụng tùy thuộc vào kích thước của kiểu dữ liệu (8, 16, 32 hoặc 64 bits).

#### Hình 3.28: Các thanh ghi truyền tham số

| Thứ tự tham số | 64-bit | 32-bit | 16-bit | 8-bit |
| :--- | :--- | :--- | :--- | :--- |
| **Tham số 1** | `%rdi` | `%edi` | `%di` | `%dil` |
| **Tham số 2** | `%rsi` | `%esi` | `%si` | `%sil` |
| **Tham số 3** | `%rdx` | `%edx` | `%dx` | `%dl` |
| **Tham số 4** | `%rcx` | `%ecx` | `%cx` | `%cl` |
| **Tham số 5** | `%r8` | `%r8d` | `%r8w` | `%r8b` |
| **Tham số 6** | `%r9` | `%r9d` | `%r9w` | `%r9b` |

---

### Truyền tham số qua Ngăn xếp (Stack)

Khi một hàm có nhiều hơn 6 tham số số nguyên, các tham số còn lại (từ thứ 7 trở đi) sẽ được truyền qua ngăn xếp.

1.  **Vị trí:** Các tham số 7 đến $n$ được đặt trong khung ngăn xếp của **hàm gọi (P)**.
2.  **Căn chỉnh bộ nhớ:** Khi truyền qua ngăn xếp, tất cả các kích thước dữ liệu đều được làm tròn lên bội số của **8 bytes** để đảm bảo hiệu năng truy cập.
3.  **Thứ tự:** Tham số thứ 7 sẽ nằm ở đỉnh ngăn xếp (ngay phía trên địa chỉ trả về).
4.  **Truy cập:** Hàm bị gọi (Q) có thể truy cập các tham số này thông qua các địa chỉ tương đối so với `%rsp`.

---

### Giá trị trả về (Return Value)

*   Thanh ghi **`%rax`** (hoặc các phần con của nó như `%eax`, `%ax`, `%al`) luôn được dùng để chứa giá trị mà hàm trả về cho hàm gọi.

---

### IDA Pro Insights (Quy ước gọi hàm thực tế)

1.  **Nhận diện tham số:** 
    *   Khi bạn vừa nhảy vào một hàm trong IDA, hãy nhìn vào các thanh ghi `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`. Nếu chúng được sử dụng trước khi được gán giá trị mới, đó chính là các tham số của hàm.
    *   IDA thường tự động đổi tên các thanh ghi này thành `a1`, `a2`, `a3`,... hoặc `arg0`, `arg1`,...
2.  **Tham số trên Stack:**
    *   Nếu bạn thấy lệnh `mov rax, [rsp+8]` hoặc `mov eax, [rbp+10h]` ở đầu hàm, đó là cách chương trình lấy **tham số thứ 7** từ Stack.
    *   IDA thường gắn nhãn các vị trí này là `arg_0`, `arg_8`,... (lưu ý: `arg_0` trong khung stack của IDA thường tương ứng với tham số thứ 7 thực tế vì 6 cái đầu đã nằm ở thanh ghi).
3.  **Kích thước tham số:** 
    *   Nếu hàm nhận một biến `char`, bạn sẽ thấy nó sử dụng `%dil` hoặc `%sil`.
    *   Nếu hàm nhận biến `int`, bạn sẽ thấy dùng `%edi` hoặc `%esi`.
    *   Việc quan sát kích thước thanh ghi giúp bạn xác định đúng kiểu dữ liệu trong mã nguồn C gốc.

---

### Ví dụ: Hàm có nhiều tham số với các kiểu dữ liệu khác nhau

Xét hàm `proc` nhận 8 tham số có kích thước khác nhau (từ 1 đến 8 bytes). Điều này giúp chúng ta thấy rõ cách x86-64 phối hợp giữa thanh ghi và ngăn xếp.

#### 1. Mã nguồn C (Hình 3.29a)
```c
void proc(long a1, long *a1p,
          int a2, int *a2p,
          short a3, short *a3p,
          char a4, char *a4p)
{
    *a1p += a1;
    *a2p += a2;
    *a3p += a3;
    *a4p += a4;
}
```

#### 2. Phân tích mã Assembly (Hình 3.29b)
**Ánh xạ tham số theo quy ước ABI:**
*   6 tham số đầu nằm trong thanh ghi: 
    `a1` (%rdi), `a1p` (%rsi), `a2` (%edx), `a2p` (%rcx), `a3` (%r8w), `a3p` (%r9).
*   2 tham số cuối nằm trên Stack:
    `a4` tại `8(%rsp)`, `a4p` tại `16(%rsp)`.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `proc:` | `proc:` | Nhãn hàm. |
| 2 | `movq 16(%rsp), %rax` | `mov rax, [rsp+16]` | **Lấy tham số 8** (`a4p`) từ Stack. |
| 3 | `addq %rdi, (%rsi)` | `add [rsi], rdi` | `*a1p += a1` (8 bytes). |
| 4 | `addl %edx, (%rcx)` | `add [rcx], edx` | `*a2p += a2` (4 bytes). |
| 5 | `addw %r8w, (%r9)` | `add [r9], r8w` | `*a3p += a3` (2 bytes). |
| 6 | `movl 8(%rsp), %edx` | `mov edx, [rsp+8]` | **Lấy tham số 7** (`a4`) từ Stack. |
| 7 | `addb %dl, (%rax)` | `add [rax], dl` | `*a4p += a4` (1 byte). |
| 8 | `ret` | `retn` | Trả về. |

---

### Hình 3.30: Cấu trúc Stack Frame của hàm `proc`

Khi hàm `proc` đang thực thi, ngăn xếp của nó trông như sau:

<img width="967" height="284" alt="image" src="https://github.com/user-attachments/assets/a740a9ed-9b09-4c39-83ae-eed392b1431a" />

**Lưu ý quan trọng:** 
* Biến `a4p (char *)` là một con trỏ, luôn có kích thước 8 bytes trong kiến trúc 64-bit
* `a4 (char)`: Là kiểu ký tự. Trong C, `char` chỉ có kích thước 1 byte.
* Mặc dù `a4` kiểu `char` chỉ cần 1 byte, nhưng khi truyền qua Stack, nó vẫn chiếm **8 bytes** để đảm bảo mọi tham số trên ngăn xếp luôn bắt đầu tại địa chỉ là bội số của 8.

---

### Practice Problem 3.33: Xác định thứ tự và kiểu dữ liệu tham số

**Đề bài:** Hàm `procprob` có 4 tham số `u, a, v, b`. Mỗi cái là một số nguyên có dấu hoặc con trỏ.
Logic C: `*u += a; *v += b; return sizeof(a) + sizeof(b);`

**Mã Assembly:**
```assembly
procprob:
    movslq %edi, %rdi       ; Lệnh 1
    addq   %rdi, (%rdx)     ; Lệnh 2
    addb   %sil, (%rcx)     ; Lệnh 3
    movl   $6, %eax         ; Lệnh 4
    ret
```

<details>
<summary><b>Nhấn để xem lời giải phân tích tham số</b></summary>

Dựa vào thứ tự thanh ghi mặc định (`%rdi`, `%rsi`, `%rdx`, `%rcx`):

1.  **Xác định `a` và `u`:**
    *   Lệnh 2: `addq %rdi, (%rdx)`. `%rdi` được cộng vào địa chỉ nằm trong `%rdx`. 
    *   Điều này có nghĩa là tham số thứ 1 (`a`) được cộng vào nơi tham số thứ 3 (`u`) trỏ tới.
    *   Lệnh 1: `movslq %edi, %rdi` cho thấy `a` ban đầu là 4 bytes (`int`) rồi được mở rộng dấu lên 8 bytes để cộng vào một giá trị `long`.
    *   $\Rightarrow$ **`a` là `int` (4 bytes)**, **`u` là `long *`**.

2.  **Xác định `b` và `v`:**
    *   Lệnh 3: `addb %sil, (%rcx)`. Thanh ghi `%sil` (8-bit thấp của `%rsi` - tham số thứ 2) được cộng vào địa chỉ nằm trong `%rcx` (tham số thứ 4).
    *   $\Rightarrow$ **`b` là tham số thứ 2**, **`v` là tham số thứ 4**.
    *   Lệnh 4: `return 6`. Ta có `sizeof(a) + sizeof(b) = 6`. Vì `sizeof(a) = 4`, nên `sizeof(b)` phải là **2**.
    *   $\Rightarrow$ **`b` là `short` (2 bytes)**, **`v` là `char *`** (vì lệnh `addb` tác động lên 1 byte tại đích).

**Kết luận prototype:**
`long procprob(int a, short b, long *u, char *v)`

</details>

---

### IDA Pro Insights (Cách đọc tham số hỗn hợp)

1.  **Căn chỉnh Stack (Alignment):** Trong IDA, bạn sẽ thấy các tham số stack luôn cách nhau 8 đơn vị (`arg_0`, `arg_8`, `arg_10h`) bất kể kiểu dữ liệu là `char` hay `int`. Đây là quy tắc tối ưu hóa truy cập bộ nhớ.
2.  **Kích thước lệnh (`addq` vs `addl` vs `addb`):** 
    *   Nếu IDA hiển thị `add [rax], dl` $\rightarrow$ Đích là con trỏ `char *`.
    *   Nếu IDA hiển thị `add [rax], edx` $\rightarrow$ Đích là con trỏ `int *`.
    *   Đây là manh mối quan trọng nhất để bạn khôi phục lại các kiểu dữ liệu `long`, `int`, `short`, `char` trong C.
3.  **Hàm `sizeof` trong Assembly:** Trong IDA, bạn sẽ không thấy lệnh tính `sizeof`. Kết quả của `sizeof` là một **hằng số** đã được trình biên dịch tính sẵn và nạp trực tiếp vào `%eax` (như lệnh `mov eax, 6` ở dòng 4).

---

# 3.7.4 Local Storage on the Stack (Lưu trữ cục bộ trên Ngăn xếp)

Thông thường, trình biên dịch x86-64 sẽ cố gắng giữ toàn bộ biến cục bộ trên các thanh ghi đa năng của CPU để đạt tốc độ tối đa. Tuy nhiên, có **3 trường hợp bắt buộc phải lưu trữ dữ liệu cục bộ trên RAM (Stack)**:

1.  **Thiếu thanh ghi:** Số lượng biến cục bộ vượt quá số lượng thanh ghi đa năng khả dụng của CPU.
2.  **Sử dụng toán tử lấy địa chỉ (`&`):** Khi mã nguồn C lấy địa chỉ của một biến cục bộ (ví dụ: `&x`), biến đó bắt buộc phải nằm trên RAM (Stack) vì các thanh ghi CPU không có địa chỉ bộ nhớ.
3.  **Dữ liệu dạng mảng hoặc cấu trúc (Arrays / Structs):** Các cấu trúc dữ liệu này yêu cầu truy cập thông qua chỉ số (index) hoặc độ lệch (offset) địa chỉ, do đó bắt buộc phải được cấp phát liên tục trên bộ nhớ.

**Cơ chế cấp phát:** Thủ tục sẽ cấp phát không gian trên khung ngăn xếp (stack frame) bằng cách giảm con trỏ ngăn xếp `%rsp` (ví dụ: `subq $16, %rsp`). Vùng nhớ này nằm ở phần "Local variables" trong Hình 3.25.

---

### Ví dụ 1: Hàm `caller` và toán tử lấy địa chỉ `&`

Hãy xem xét cách xử lý khi một hàm lấy địa chỉ của biến cục bộ và truyền sang hàm khác.

#### 1. Mã nguồn C:
```c
long swap_add(long *xp, long *yp) {
    long x = *xp;
    long y = *yp;
    *xp = y;
    *yp = x;
    return x + y;
}

long caller() {
    long arg1 = 534;
    long arg2 = 1057;
    long sum = swap_add(&arg1, &arg2); // Lấy địa chỉ biến cục bộ
    long diff = arg1 - arg2;
    return sum * diff;
}
```

#### 2. Phân tích mã Assembly của hàm `caller` (ATT vs Intel/IDA Pro)

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích kỹ thuật |
| :--- | :--- | :--- | :--- |
| 1 | `caller:` | `caller:` | Nhãn hàm. |
| 2 | `subq $16, %rsp` | `sub rsp, 10h` | **Cấp phát 16 bytes** trên Stack cho `arg1` và `arg2`. |
| 3 | `movq $534, (%rsp)` | `mov qword ptr [rsp], 216h` | Ghi `arg1 = 534` vào đỉnh Stack (`rsp + 0`). |
| 4 | `movq $1057, 8(%rsp)` | `mov qword ptr [rsp+8], 421h` | Ghi `arg2 = 1057` vào địa chỉ `rsp + 8`. |
| 5 | `leaq 8(%rsp), %rsi` | `lea rsi, [rsp+8]` | Lấy địa chỉ `arg2` đưa vào `%rsi` (Tham số 2). |
| 6 | `movq %rsp, %rdi` | `mov rdi, rsp` | Lấy địa chỉ `arg1` đưa vào `%rdi` (Tham số 1). |
| 7 | `call swap_add` | `call swap_add` | Gọi hàm `swap_add(&arg1, &arg2)`. |
| 8 | `movq (%rsp), %rdx` | `mov rdx, [rsp]` | Đọc lại giá trị mới của `arg1` (đã bị swap) vào `%rdx`. |
| 9 | `subq 8(%rsp), %rdx` | `sub rdx, [rsp+8]` | Lấy `arg1` trừ đi `arg2` mới $\rightarrow$ `diff = arg1 - arg2`. |
| 10 | `imulq %rdx, %rax` | `imul rax, rdx` | Nhân `sum` (trả về trong `%rax`) với `diff`. |
| 11 | `addq $16, %rsp` | `add rsp, 10h` | **Giải phóng 16 bytes** trên Stack. |
| 12 | `ret` | `retn` | Trả về kết quả. |

---

### Ví dụ 2: Hàm `call_proc` (Khung Stack phức tạp và > 6 Tham số)

Đây là một ví dụ toàn diện minh họa cách cấp phát Stack cho các biến cục bộ có **kích thước khác nhau** (`long`, `int`, `short`, `char`) đồng thời chuẩn bị tham số trên Stack cho hàm `proc` (hàm nhận đến 8 tham số).

#### 1. Mã nguồn C:
```c
long call_proc() {
    long x1 = 1;      int x2 = 2;
    short x3 = 3;     char x4 = 4;
    // proc nhận 8 tham số (truyền địa chỉ của x1, x2, x3, x4)
    proc(x1, &x1, x2, &x2, x3, &x3, x4, &x4);
    return (x1 + x2) * (x3 - x4);
}
```

#### 2. Phân tích mã Assembly của hàm `call_proc` (ATT vs Intel/IDA Pro)

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích kỹ thuật |
| :--- | :--- | :--- | :--- |
| 1 | `call_proc:` | `call_proc:` | Nhãn hàm. |
| 2 | `subq $32, %rsp` | `sub rsp, 20h` | **Cấp phát 32 bytes** trên Stack. |
| 3 | `movq $1, 24(%rsp)` | `mov qword ptr [rsp+18h], 1` | Khởi tạo `x1 = 1` (8 bytes, tại offset 24-31). |
| 4 | `movl $2, 20(%rsp)` | `mov dword ptr [rsp+14h], 2` | Khởi tạo `x2 = 2` (4 bytes, tại offset 20-23). |
| 5 | `movw $3, 18(%rsp)` | `mov word ptr [rsp+12h], 3` | Khởi tạo `x3 = 3` (2 bytes, tại offset 18-19). |
| 6 | `movb $4, 17(%rsp)` | `mov byte ptr [rsp+11h], 4` | Khởi tạo `x4 = 4` (1 byte, tại offset 17). |
| | | | **Bắt đầu chuẩn bị tham số cho hàm `proc`:** |
| 7 | `leaq 17(%rsp), %rax` | `lea rax, [rsp+11h]` | Lấy địa chỉ `x4`. |
| 8 | `movq %rax, 8(%rsp)` | `mov [rsp+8], rax` | **Tham số 8 (`&x4`)**: Lưu vào Stack tại offset 8. |
| 9 | `movl $4, (%rsp)` | `mov dword ptr [rsp], 4` | **Tham số 7 (`x4` = 4)**: Lưu vào Stack tại offset 0 (chiếm 8 bytes). |
| 10 | `leaq 18(%rsp), %r9` | `lea r9, [rsp+12h]` | **Tham số 6 (`&x3`)**: Truyền qua thanh ghi `%r9`. |
| 11 | `movl $3, %r8d` | `mov r8d, 3` | **Tham số 5 (`x3` = 3)**: Truyền qua thanh ghi `%r8d`. |
| 12 | `leaq 20(%rsp), %rcx` | `lea rcx, [rsp+14h]` | **Tham số 4 (`&x2`)**: Truyền qua thanh ghi `%rcx`. |
| 13 | `movl $2, %edx` | `mov edx, 2` | **Tham số 3 (`x2` = 2)**: Truyền qua thanh ghi `%edx`. |
| 14 | `leaq 24(%rsp), %rsi` | `lea rsi, [rsp+18h]` | **Tham số 2 (`&x1`)**: Truyền qua thanh ghi `%rsi`. |
| 15 | `movl $1, %edi` | `mov edi, 1` | **Tham số 1 (`x1` = 1)**: Truyền qua thanh ghi `%edi`. |
| 16 | `call proc` | `call proc` | Gọi hàm `proc`. |
| | | | **Sau khi từ hàm `proc` trở về:** |
| 17 | `movslq 20(%rsp), %rdx`| `movsxd rdx, dword ptr [rsp+14h]` | Đọc `x2` mới từ Stack, mở rộng dấu lên 64-bit. |
| 18 | `addq 24(%rsp), %rdx` | `add rdx, [rsp+18h]` | Cộng với `x1` mới $\rightarrow$ `rdx = x1 + x2`. |
| 19 | `movswl 18(%rsp), %eax`| `movsx eax, word ptr [rsp+12h]` | Đọc `x3` mới, mở rộng dấu lên 32-bit. |
| 20 | `movsbl 17(%rsp), %ecx`| `movsx ecx, byte ptr [rsp+11h]` | Đọc `x4` mới, mở rộng dấu lên 32-bit. |
| 21 | `subl %ecx, %eax` | `sub eax, ecx` | Tính `x3 - x4`. |
| 22 | `cltq` | `cdqe` | Mở rộng dấu `%eax` lên `%rax` (64-bit). |
| 23 | `imulq %rdx, %rax` | `imul rax, rdx` | Nhân kết quả $\rightarrow$ `(x1 + x2) * (x3 - x4)`. |
| 24 | `addq $32, %rsp` | `add rsp, 20h` | **Giải phóng 32 bytes** trên Stack. |
| 25 | `ret` | `retn` | Trả về. |

---

### Hình 3.33: Cấu trúc chi tiết Stack Frame của `call_proc` (32 Bytes)

Sơ đồ dưới đây biểu diễn cách phân bổ bộ nhớ chính xác trên ngăn xếp của `call_proc`:

<img width="796" height="253" alt="image" src="https://github.com/user-attachments/assets/58983136-0ba5-4603-9706-e5afe37c1ca4" />

**Sự dịch chuyển địa chỉ khi gọi hàm `proc`:**
*   Khi `call_proc` chuẩn bị gọi `proc`, nó nạp sẵn **Tham số 7** và **Tham số 8** vào vùng `[rsp + 0]` và `[rsp + 8]`.
*   Khi lệnh `call proc` được thực thi, bộ xử lý tự động đẩy địa chỉ trả về (8 bytes) vào đỉnh Stack.
*   **Kết quả:** Bên trong hàm `proc`, toàn bộ đỉnh Stack bị dịch xuống 8 bytes. Lúc này, từ góc nhìn của hàm `proc`, **Tham số 7** sẽ nằm ở `8(%rsp)` và **Tham số 8** sẽ nằm ở `16(%rsp)`. Điều này hoàn toàn khớp với những gì ta đã học ở Mục 3.7.3!

---

### IDA Pro Insights (Kỹ năng phân tích Stack Frame nâng cao)

Khi bạn dịch ngược hàm có cấu trúc Stack phức tạp như `call_proc` trong IDA Pro:

1.  **Nhận diện các kiểu dữ liệu khác nhau:**
    *   IDA Pro sẽ tạo một Stack View chứa danh sách biến. Bạn sẽ thấy các biến được định nghĩa kích thước khác nhau nằm liền kề:
        *   `var_18` (8 bytes) $\rightarrow$ tương ứng với `x1` kiểu `long`.
        *   `var_14` (4 bytes) $\rightarrow$ tương ứng với `x2` kiểu `int`.
        *   `var_12` (2 bytes) $\rightarrow$ tương ứng với `x3` kiểu `short`.
        *   `var_11` (1 byte)  $\rightarrow$ tương ứng với `x4` kiểu `char`.
2.  **Sự xuất hiện của Biến đệm (Padding):**
    *   Trong Stack View của IDA, bạn sẽ thấy giữa `var_11` (offset 17) và vùng đối số `arg_0` (offset 8) có một vùng trống không tên (ở offset 16). Đây là **vùng đệm căn chỉnh (Alignment Padding)** của trình biên dịch để đảm bảo dữ liệu luôn được căn lề chẵn địa chỉ, giúp CPU đọc nhanh hơn.
3.  **Nhận diện "Argument Build Area":**
    *   Nếu ở đầu hàm có lệnh `sub rsp, 20h` (32 bytes), nhưng vùng biến cục bộ được IDA nhận diện chỉ chiếm khoảng 16 bytes, thì 16 bytes trống ở đáy ngăn xếp chính là **vùng dựng tham số (Argument Build Area)** để chuẩn bị cho các tham số thứ 7 và thứ 8 của hàm tiếp theo.
4.  **Cặp đôi `lea` và `mov`:**
    *   Trong IDA, khi thấy lệnh `lea rax, [rsp+20h+var_11]` theo sau bởi một lệnh nạp vào stack `mov [rsp+8], rax` ngay trước lệnh `call`, đó chính là thao tác **lấy địa chỉ của một biến cục bộ để truyền làm tham số cho hàm khác**.

---

## 3.7.5 Local Storage in Registers (Lưu trữ cục bộ trong Thanh ghi)

Tập hợp các thanh ghi chương trình đóng vai trò là một tài nguyên duy nhất được chia sẻ bởi tất cả các thủ tục. Mặc dù tại một thời điểm chỉ có một thủ tục được kích hoạt, chúng ta phải đảm bảo rằng khi một thủ tục (hàm gọi - **caller**) gọi một thủ tục khác (hàm bị gọi - **callee**), hàm bị gọi không được ghi đè lên các giá trị thanh ghi mà hàm gọi dự định sử dụng sau đó. 

Vì lý do này, x86-64 áp dụng một tập hợp các **quy ước sử dụng thanh ghi** đồng nhất mà tất cả các thủ tục phải tuân thủ.

### 1. Thanh ghi do hàm bị gọi lưu (Callee-Saved Registers)

Theo quy ước, các thanh ghi **`%rbx`**, **`%rbp`**, và **`%r12`–`%r15`** được phân loại là các thanh ghi **callee-saved**. 

*   **Quy tắc:** Khi hàm **P** gọi hàm **Q**, hàm **Q** phải bảo toàn giá trị của các thanh ghi này. 
*   **Cách thực hiện:** Hàm **Q** có thể bảo toàn giá trị bằng hai cách:
    1.  Không thay đổi chúng.
    2.  Đẩy giá trị cũ vào ngăn xếp (**push**) khi bắt đầu hàm, sử dụng thanh ghi để tính toán, và sau đó lấy lại giá trị cũ từ ngăn xếp (**pop**) trước khi trả về.
*   Việc đẩy các thanh ghi này tạo ra phần "Saved registers" trong Stack Frame (Hình 3.25).

### 2. Thanh ghi do hàm gọi lưu (Caller-Saved Registers)

Tất cả các thanh ghi khác (ngoại trừ con trỏ ngăn xếp `%rsp`) đều được phân loại là **caller-saved**.

*   **Quy tắc:** Các thanh ghi này có thể bị sửa đổi bởi bất kỳ hàm nào.
*   **Cách hiểu:** Nếu hàm **P** có dữ liệu quan trọng trong một thanh ghi caller-saved và muốn gọi hàm **Q**, thì **P** phải có trách nhiệm tự cất dữ liệu đó đi (thường là đẩy vào Stack) trước khi gọi **Q**.

---

### Hình 3.34: Ví dụ minh họa sử dụng thanh ghi Callee-saved

Xét hàm `P` gọi hàm `Q` hai lần. Trong lần gọi đầu tiên, nó phải giữ giá trị của `x`. Trong lần gọi thứ hai, nó phải giữ kết quả của `Q(y)`.

#### (a) Mã nguồn C
```c
long P(long x, long y) {
    long u = Q(y);
    long v = Q(x);
    return u + v;
}
```

#### (b) Phân tích mã Assembly (ATT vs Intel/IDA Pro)
*Quy ước: x trong %rdi, y trong %rsi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `P:` | `P:` | Nhãn hàm. |
| 2 | `pushq %rbp` | `push rbp` | **Save rbp**: Lưu giá trị cũ của `%rbp`. |
| 3 | `pushq %rbx` | `push rbx` | **Save rbx**: Lưu giá trị cũ của `%rbx`. |
| 4 | `subq $8, %rsp` | `sub rsp, 8` | Căn chỉnh ngăn xếp (Stack alignment). |
| 5 | `movq %rdi, %rbp` | `mov rbp, rdi` | **Lưu x vào rbp**: `%rbp` là callee-saved, an toàn qua lệnh `call`. |
| 6 | `movq %rsi, %rdi` | `mov rdi, rsi` | Đưa `y` vào tham số thứ nhất để gọi `Q(y)`. |
| 7 | `call Q` | `call Q` | Gọi `Q(y)`. Kết quả `u` trả về trong `%rax`. |
| 8 | `movq %rax, %rbx` | `mov rbx, rax` | **Lưu u vào rbx**: Đảm bảo `u` không bị mất khi gọi `Q(x)`. |
| 9 | `movq %rbp, %rdi` | `mov rdi, rbp` | Lấy `x` từ `%rbp` để làm tham số cho `Q(x)`. |
| 10 | `call Q` | `call Q` | Gọi `Q(x)`. Kết quả `v` trả về trong `%rax`. |
| 11 | `addq %rbx, %rax` | `add rax, rbx` | `rax = v + u`. |
| 12 | `addq $8, %rsp` | `add rsp, 8` | Giải phóng vùng đệm. |
| 13 | `popq %rbx` | `pop rbx` | **Restore rbx**: Khôi phục giá trị cho hàm gọi P. |
| 14 | `popq %rbp` | `pop rbp` | **Restore rbp**: Khôi phục giá trị cho hàm gọi P. |
| 15 | `ret` | `retn` | Trả về. |

---

### IDA Pro Insights (Kỹ năng nhận diện biến cục bộ)

Khi bạn nhìn vào một hàm trong IDA Pro, các quy ước này giúp bạn nhận diện logic rất nhanh:

1.  **Function Prologue & Epilogue:**
    *   Sự xuất hiện của `push rbx`, `push rbp`, `push r12` ở đầu hàm (Prologue) và các lệnh `pop` tương ứng ở cuối hàm (Epilogue) là minh chứng cho việc hàm này đang sử dụng các thanh ghi **callee-saved** để làm việc.
2.  **Nhận diện Biến quan trọng:**
    *   Nếu bạn thấy một giá trị được chuyển từ tham số (`rdi`, `rsi`...) sang một thanh ghi như `rbx` hoặc `r12` ngay đầu hàm (như dòng 5), hãy hiểu rằng: **Đây là một biến quan trọng cần được bảo toàn qua các lời gọi hàm khác.** 
    *   Trong IDA, bạn nên đổi tên các thanh ghi này thành tên biến (như `x_saved` hoặc `u_result`) để dễ theo dõi.
3.  **Thứ tự Stack:**
    *   Lưu ý thứ tự `pop` (dòng 13-14) luôn **ngược lại** với thứ tự `push` (dòng 2-3). Đây là nguyên tắc LIFO của ngăn xếp mà trình biên dịch phải tuân thủ để không làm hỏng dữ liệu.
4.  **Tối ưu hóa (Leaf Functions):**
    *   Nếu một hàm không gọi hàm nào khác, bạn sẽ thấy nó ít khi dùng `push`/`pop` các thanh ghi này, mà thay vào đó sử dụng các thanh ghi caller-saved (`rax`, `rcx`, `rdx`...) để tiết kiệm lệnh.

---

### Practice Problem 3.34: Phân bổ biến cục bộ (Registers vs. Stack)

**Đề bài:** Hàm `P` tạo ra các giá trị cục bộ từ `a0` đến `a8`. Sau đó nó gọi hàm `Q`. Dựa vào mã Assembly, hãy xác định cách các biến này được lưu trữ.

#### 1. Phân tích mã Assembly (ATT vs Intel/IDA Pro)
*Quy ước: x nằm trong %rdi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 2-7 | `pushq %r15...%rbx` | `push r15...rbx` | **Save Callee-saved**: Sao lưu 6 thanh ghi. |
| 8 | `subq $24, %rsp` | `sub rsp, 18h` | Cấp phát 24 bytes trên Stack cho biến cục bộ. |
| 9 | `movq %rdi, %rbx` | `mov rbx, rdi` | **a0 = x** (Lưu vào `%rbx`) |
| 10 | `leaq 1(%rdi), %r15` | `lea r15, [rdi+1]` | **a1 = x + 1** (Lưu vào `%r15`) |
| 11 | `leaq 2(%rdi), %r14` | `lea r14, [rdi+2]` | **a2 = x + 2** (Lưu vào `%r14`) |
| 12 | `leaq 3(%rdi), %r13` | `lea r13, [rdi+3]` | **a3 = x + 3** (Lưu vào `%r13`) |
| 13 | `leaq 4(%rdi), %r12` | `lea r12, [rdi+4]` | **a4 = x + 4** (Lưu vào `%r12`) |
| 14 | `leaq 5(%rdi), %rbp` | `lea rbp, [rdi+5]` | **a5 = x + 5** (Lưu vào `%rbp`) |
| 15-16| `leaq 6(%rdi), %rax` | `lea rax, [rdi+6]` | Tính `a6`. |
| | `movq %rax, (%rsp)` | `mov [rsp], rax` | **a6 = x + 6** (Lưu vào Stack offset 0). |
| 17-18| `leaq 7(%rdi), %rdx` | `lea rdx, [rdi+7]` | Tính `a7`. |
| | `movq %rdx, 8(%rsp)` | `mov [rsp+8], rdx` | **a7 = x + 7** (Lưu vào Stack offset 8). |
| 19-20| `movl $0, %eax` | `mov eax, 0` | Chuẩn bị gọi hàm `Q`. |

---

### Trả lời câu hỏi:

**A. Những giá trị cục bộ nào được lưu trong các thanh ghi callee-saved?**
Dựa vào các lệnh từ dòng 9 đến 14, các biến sau nằm trong thanh ghi:
*   **`a0`** (trong `%rbx`), **`a1`** (trong `%r15`), **`a2`** (trong `%r14`), **`a3`** (trong `%r13`), **`a4`** (trong `%r12`), **`a5`** (trong `%rbp`).

**B. Những giá trị cục bộ nào được lưu trên Ngăn xếp (Stack)?**
Dựa vào các lệnh `movq` vào địa chỉ tương đối của `%rsp` (dòng 16, 18):
*   **`a6`** (tại `%rsp`), **`a7`** (tại `8(%rsp)`). 
*   *Ghi chú:* Biến **`a8`** (không xuất hiện trong đoạn code cụ thể nhưng theo logic sẽ nằm tại `16(%rsp)` vì vùng stack đã cấp phát 24 bytes).

**C. Tại sao chương trình không thể lưu tất cả các giá trị cục bộ vào thanh ghi callee-saved?**
*   **Lý do:** Kiến trúc x86-64 chỉ có đúng **6 thanh ghi** được phân loại là callee-saved dành cho mục đích chung (general purpose): `%rbx, %rbp, %r12, %r13, %r14, %r15`. 
*   Vì hàm `P` cần lưu trữ tới 9 giá trị cục bộ (`a0-a8`) để sử dụng sau lời gọi hàm `Q`, nên sau khi dùng hết 6 thanh ghi này, trình biên dịch buộc phải đưa các biến còn lại xuống bộ nhớ (Stack).

---

### IDA Pro Insights (Tư duy ánh xạ biến)

Khi bạn gặp một hàm dài như thế này trong IDA Pro:

1.  **Đếm số lượng thanh ghi Callee-saved:** Nếu bạn thấy đầu hàm `push` rất nhiều thanh ghi (như dòng 2-7), hãy chuẩn bị tinh thần rằng đây là một hàm có rất nhiều biến cục bộ "sống dai" (cần tồn tại qua các lệnh `call`).
2.  **Xác định biến chính yếu:** Trình biên dịch thường ưu tiên đưa các biến được truy cập nhiều nhất vào thanh ghi. Trong IDA, các thanh ghi như `r15, r14...` sẽ được IDA tự động phân tích luồng dữ liệu. Bạn nên Rename chúng thành `var_a0, var_a1...` dựa trên logic tính toán.
3.  **Vùng Stack hỗn hợp:** IDA sẽ hiển thị vùng `sub rsp, 18h` này trong Stack View. Bạn sẽ thấy:
    *   `var_18`: tương ứng `a6`.
    *   `var_10`: tương ứng `a7`.
    *   `var_8`: tương ứng `a8`.
    *   Việc thấy một phần biến nằm ở thanh ghi, một phần nằm ở Stack là dấu hiệu của việc "tràn thanh ghi" (register spilling).

---

## 3.7.6 Recursive Procedures (Các thủ tục đệ quy)

Các quy ước sử dụng thanh ghi và ngăn xếp mà chúng ta đã tìm hiểu cho phép các thủ tục x86-64 gọi chính chúng một cách đệ quy. 

**Tại sao đệ quy hoạt động được ở mức máy?**
1.  **Sự cô lập của Ngăn xếp:** Mỗi lần gọi hàm (invocation) đều có không gian riêng tư (private space) trên ngăn xếp. Do đó, các biến cục bộ của nhiều lần gọi đang thực thi không hề xâm phạm lẫn nhau.
2.  **Quy tắc Stack:** Cơ chế ngăn xếp cung cấp một chính sách tự nhiên để cấp phát lưu trữ cục bộ khi hàm được gọi và giải phóng nó khi hàm trả về, khớp hoàn toàn với thứ tự thực thi của đệ quy.

---

### Ví dụ: Hàm Giai thừa đệ quy (`rfact`)

Hãy phân tích cách trình biên dịch triển khai hàm tính giai thừa đệ quy.

#### 1. Mã nguồn C (Hình 3.35a)
```c
long rfact(long n) {
    long result;
    if (n <= 1)
        result = 1;
    else
        result = n * rfact(n - 1);
    return result;
}
```

#### 2. Phân tích mã Assembly (Hình 3.35b)
*Quy ước: n nằm trong %rdi*

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `rfact:` | `rfact:` | Nhãn hàm. |
| 2 | `pushq %rbx` | `push rbx` | **Lưu %rbx**: Vì %rbx là callee-saved. |
| 3 | `movq %rdi, %rbx` | `mov rbx, rdi` | **Lưu n vào %rbx**: Để giữ `n` không bị mất khi gọi đệ quy. |
| 4 | `movl $1, %eax` | `mov eax, 1` | Mặc định `result = 1`. |
| 5 | `cmpq $1, %rdi` | `cmp rdi, 1` | So sánh `n` với 1 (Điều kiện dừng). |
| 6 | `jle .L35` | `jle short loc_done` | **Base case**: Nếu $n \le 1$, nhảy tới kết thúc. |
| 7 | `leaq -1(%rdi), %rdi`| `lea rdi, [rdi-1]` | Chuẩn bị tham số `n-1` cho lần gọi tới. |
| 8 | `call rfact` | `call rfact` | **Recursive call**: Gọi chính nó. |
| 9 | `imulq %rbx, %rax` | `imul rax, rbx` | `result = n * rfact(n-1)`. |
| 10 | `.L35:` | `loc_done:` | Nhãn kết thúc. |
| 11 | `popq %rbx` | `pop rbx` | Khôi phục lại giá trị `%rbx` ban đầu. |
| 12 | `ret` | `retn` | Trả về. |

---

### Phân tích sâu: Tại sao phải dùng `%rbx`?

Trong lập trình đệ quy, giá trị của `n` trong lần gọi hiện tại phải được bảo toàn để sử dụng sau khi hàm đệ quy trả về (nhằm thực hiện phép nhân ở dòng 9).
*   Nếu trình biên dịch để `n` trong thanh ghi `%rdi` (caller-saved), khi lệnh `call rfact` thực thi, hàm bị gọi có thể (và chắc chắn sẽ) ghi đè giá trị mới lên `%rdi`, làm mất `n` của lần gọi trước.
*   **Giải pháp:** Trình biên dịch lưu `n` vào **`%rbx`**. Vì `%rbx` là **callee-saved**, quy ước đảm bảo rằng sau khi lệnh `call` hoàn thành, giá trị trong `%rbx` vẫn y nguyên như trước khi gọi. 
*   Để dùng `%rbx`, hàm `rfact` phải có trách nhiệm `push` nó lên Stack ở dòng 2 và `pop` lại ở dòng 11 để không làm hỏng dữ liệu của hàm đã gọi nó.

---

### IDA Pro Insights (Kỹ năng nhận diện Đệ quy)

1.  **Lệnh Call chính nó:** Đây là dấu hiệu rõ ràng nhất. Trong cửa sổ hàm của IDA, bạn sẽ thấy mũi tên từ lệnh `call` quay ngược lại chính địa chỉ bắt đầu của hàm đó.
2.  **Sử dụng thanh ghi Callee-saved:** Khi bạn thấy đầu hàm `push rbx` và sau đó `mov rbx, rdi`, hãy hiểu ngay rằng biến tham số đầu tiên đang được "đóng băng" vào một vùng an toàn để chuẩn bị cho một lời gọi hàm khác (có thể là đệ quy hoặc gọi hàm thư viện).
3.  **Cấu trúc Graph View:** 
    *   Một nhánh dẫn tới lệnh `ret` với giá trị hằng số (thường là 1 hoặc 0) $\rightarrow$ Đây là **Base Case**.
    *   Một nhánh chứa lệnh `call` chính nó $\rightarrow$ Đây là **Recursive Step**.
4.  **Tác động lên Ngăn xếp:** Với mỗi lần đệ quy, IDA Debugger sẽ cho thấy Stack phình ra thêm 16 bytes (8 byte cho địa chỉ trả về + 8 byte cho lệnh `push rbx`).

---

### Practice Problem 3.35: Dịch ngược hàm đệ quy `rfun`

**Đề bài:** Hoàn thiện mã nguồn C của hàm đệ quy `rfun` dựa trên mã Assembly được cung cấp.

#### 1. Phân tích mã Assembly (Ánh xạ ATT và Intel/IDA Pro)
*Quy ước: x nằm trong %rdi, kết quả trả về nằm trong %rax*

| Dòng | AT&T Syntax | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 2 | `pushq %rbx` | `push rbx` | **Save rbx**: Lưu giá trị cũ của `%rbx`. |
| 3 | `movq %rdi, %rbx` | `mov rbx, rdi` | **Lưu x vào rbx**: Giữ giá trị `x` gốc để dùng sau khi gọi đệ quy. |
| 4 | `movl $0, %eax` | `mov eax, 0` | Mặc định trả về **0**. |
| 5 | `testq %rdi, %rdi` | `test rdi, rdi` | Kiểm tra `x == 0`. |
| 6 | `je .L2` | `jz short loc_L2` | **Base Case**: Nếu `x == 0`, nhảy tới kết thúc để trả về 0. |
| 7 | `shrq $2, %rdi` | `shr rdi, 2` | **nx = x >> 2**: Dịch phải 2 bit (tương đương `x / 4`). |
| 8 | `call rfun` | `call rfun` | **rv = rfun(nx)**: Gọi đệ quy với tham số mới. |
| 9 | `addq %rbx, %rax` | `add rax, rbx` | **return x + rv**: Cộng `x` gốc (trong `%rbx`) vào kết quả đệ quy. |
| 10 | `.L2:` | `loc_L2:` | Nhãn kết thúc. |
| 11 | `popq %rbx` | `pop rbx` | **Restore rbx**: Khôi phục thanh ghi cho hàm gọi. |
| 12 | `ret` | `retn` | Trả về. |

---

### Trả lời câu hỏi:

**A. Giá trị nào mà `rfun` lưu trữ trong thanh ghi callee-saved `%rbx`?**
*   **Trả lời:** `%rbx` lưu trữ giá trị của tham số **`x`** được truyền vào hàm trong lần gọi hiện tại. Trình biên dịch làm vậy vì `%rdi` (chứa `x`) sẽ bị ghi đè khi chuẩn bị tham số cho lệnh `call` ở dòng 7-8.

**B. Điền vào các phần còn thiếu của mã C:**
```c
long rfun(unsigned long x) {
    if (x == 0)             // Dòng 5-6
        return 0;           // Dòng 4
    unsigned long nx = x >> 2; // Dòng 7
    long rv = rfun(nx);     // Dòng 8
    return x + rv;          // Dòng 9
}
```

---

### IDA Pro Insights (Kỹ năng phân tích đệ quy thực tế)

1.  **Xác định Base Case:** Trong IDA Graph View, hãy tìm khối lệnh có lệnh `ret` mà giá trị `%rax` được nạp hằng số (như `xor eax, eax` hoặc `mov eax, 0`). Đó chính là điều kiện dừng của đệ quy.
2.  **Sự bảo toàn dữ liệu:** Khi thấy mẫu hình `mov rbx, rdi` ngay đầu hàm, hãy tự hỏi: "Tại sao nó phải cất `rdi` đi?". Câu trả lời luôn là: "Vì nó sắp gọi một hàm khác và cần dùng lại giá trị này sau khi hàm đó trả về".
3.  **Toán tử dịch bit:** Lệnh `shl`/`shr` hằng số trong IDA thường đại diện cho các phép nhân/chia lũy thừa của 2 trong C. Ở đây `shr rdi, 2` chính là `x / 4`.
4.  **Luồng trả về tích lũy:** Lệnh `add rax, rbx` sau lệnh `call` là dấu hiệu của một hàm tính toán kiểu tích lũy (ví dụ: tính tổng các phần tử, tính giai thừa, hoặc như bài này là tổng các giá trị dịch bit).

---

## 3.8 Array Allocation and Access (Cấp phát và Truy cập mảng)

Mảng trong C là một phương thức để gộp các dữ liệu vô hướng (scalar data) thành các kiểu dữ liệu lớn hơn. C sử dụng một cách triển khai mảng đặc biệt đơn giản, do đó việc chuyển dịch sang mã máy khá trực diện. Một đặc điểm khác biệt của C là chúng ta có thể tạo ra các con trỏ tới các phần tử trong mảng và thực hiện các phép toán số học trên con trỏ đó. Các phép toán này được dịch trực tiếp thành các phép tính toán địa chỉ trong mã máy.

---

## 3.8.1 Basic Principles (Các nguyên tắc cơ bản)

Với kiểu dữ liệu $T$ và một hằng số nguyên $N$, xét khai báo có dạng:
`T A[N];`

Khai báo này có hai tác động:
1.  **Cấp phát bộ nhớ:** Nó cấp phát một vùng nhớ liên tục (contiguous region) có kích thước $L \cdot N$ bytes, trong đó $L$ là kích thước (tính bằng byte) của kiểu dữ liệu $T$.
2.  **Tạo định danh:** Nó giới thiệu định danh `A` có thể được sử dụng như một con trỏ trỏ đến điểm bắt đầu của mảng. Giá trị của con trỏ này là địa chỉ bắt đầu $x_A$.

**Công thức truy cập phần tử:**
Các phần tử mảng có thể được truy cập bằng chỉ số nguyên $i$ nằm trong khoảng từ $0$ đến $N-1$. Phần tử mảng thứ $i$ sẽ được lưu trữ tại địa chỉ:
$$\text{Address}(A[i]) = x_A + L \cdot i$$

---

### Ví dụ về các loại khai báo mảng

Dưới đây là các tham số được tạo ra cho một số khai báo mảng phổ biến trên x86-64:

| Khai báo C | Kích thước phần tử ($L$) | Tổng kích thước | Địa chỉ bắt đầu | Địa chỉ phần tử $i$ |
| :--- | :---: | :---: | :---: | :--- |
| `char A[12]` | 1 | 12 | $x_A$ | $x_A + i$ |
| **`char *B[8]`** | 8 | 64 | $x_B$ | $x_B + 8i$ |
| `int C[6]` | 4 | 24 | $x_C$ | $x_C + 4i$ |
| **`double *D[5]`**| 8 | 40 | $x_D$ | $x_D + 8i$ |

**Phân tích kỹ thuật:**
*   Mảng **A** gồm 12 ký tự 1-byte.
*   Mảng **C** gồm 6 số nguyên 4-byte.
*   Mảng **B** và **D** là các mảng chứa **con trỏ** (pointer), do đó mỗi phần tử đều chiếm 8 bytes trong kiến trúc 64-bit.

---

### Truy cập mảng trong x86-64 (Optimized Access)

Các lệnh tham chiếu bộ nhớ của x86-64 được thiết kế để đơn giản hóa việc truy cập mảng. 

**Ví dụ:** Giả sử `E` là một mảng kiểu `int` (4 bytes).
*   Địa chỉ bắt đầu của `E` nằm trong `%rdx`.
*   Chỉ số `i` nằm trong `%rcx`.

Để thực hiện phép gán `eax = E[i]`, trình biên dịch chỉ cần dùng một lệnh duy nhất:

| Cú pháp ATT (Sách) | **Cú pháp Intel (IDA Pro)** | Logic tính toán |
| :--- | :--- | :--- |
| `movl (%rdx, %rcx, 4), %eax` | **`mov eax, [rdx + rcx*4]`** | $x_E + 4 \cdot i$ |

**Cơ chế:** Bộ vi xử lý thực hiện tính toán địa chỉ $x_E + 4i$ ngay trong chu kỳ đọc bộ nhớ. Các hệ số tỉ lệ (scaling factors) cho phép là **1, 2, 4, và 8**, bao phủ chính xác kích thước của tất cả các kiểu dữ liệu nguyên thủy phổ biến.

---

### IDA Pro Insights (Cách nhận diện mảng)

1.  **Scaled Indexing:** Khi bạn thấy một lệnh truy cập bộ nhớ có dạng `[reg1 + reg2 * Scale]`, gần như chắc chắn bạn đang đối mặt với một thao tác truy cập mảng.
    *   `Scale = 1`: Mảng `char` hoặc `byte`.
    *   `Scale = 2`: Mảng `short` hoặc `word`.
    *   `Scale = 4`: Mảng `int` hoặc `float`.
    *   `Scale = 8`: Mảng `long`, `double` hoặc mảng các con trỏ.
2.  **Global Arrays:** Nếu mảng là biến toàn cục, IDA Pro sẽ hiển thị địa chỉ bắt đầu dưới dạng nhãn (ví dụ: `mov eax, ds:global_array[rcx*4]`).
3.  **Local Arrays:** Nếu mảng nằm trên Stack, bạn sẽ thấy địa chỉ tính từ `%rsp` (ví dụ: `lea rax, [rsp+10h]`, sau đó dùng `rax` làm địa chỉ cơ sở).
4.  **Pointer Arithmetic:** Đôi khi trình biên dịch không dùng `Scale`. Nếu bạn thấy `add rdi, 4` bên trong một vòng lặp đang duyệt mảng, đó là cách nó dịch chuyển con trỏ sang phần tử tiếp theo của mảng `int`.

---

### Practice Problem 3.36: Tính toán đặc tính của Mảng trên x86-64

Dựa trên các khai báo C đã cho, chúng ta tính toán kích thước phần tử ($L$), tổng kích thước vùng nhớ được cấp phát, và công thức tính địa chỉ của phần tử thứ $i$.

**Quy tắc kích thước trên x86-64:**
*   `short`: 2 bytes.
*   `int`: 4 bytes.
*   `double`: 8 bytes.
*   **Con trỏ (mọi loại - `*`, `**`):** Luôn là **8 bytes**.

---

### Bảng lời giải chi tiết

| Mảng (Array) | Khai báo C | Kích thước phần tử ($L$) | Tổng kích thước (Bytes) | Địa chỉ bắt đầu | Địa chỉ phần tử $i$ |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **P** | `int P[5]` | 4 | $4 \times 5 = \mathbf{20}$ | $x_P$ | $x_P + 4i$ |
| **Q** | `short Q[2]` | 2 | $2 \times 2 = \mathbf{4}$ | $x_Q$ | $x_Q + 2i$ |
| **R** | `int **R[9]` | **8** | $8 \times 9 = \mathbf{72}$ | $x_R$ | $x_R + 8i$ |
| **S** | `double *S[10]`| **8** | $8 \times 10 = \mathbf{80}$ | $x_S$ | $x_S + 8i$ |
| **T** | `short *T[2]` | **8** | $8 \times 2 = \mathbf{16}$ | $x_T$ | $x_T + 8i$ |

---

### Phân tích kỹ thuật (Lưu ý để không bị nhầm lẫn)

1.  **Mảng con trỏ (Arrays of Pointers):**
    *   Hãy nhìn vào các mảng **R, S, T**. Dù kiểu dữ liệu gốc là `int`, `double` hay `short`, nhưng vì chúng được khai báo là mảng con trỏ (`*` hoặc `**`), kích thước mỗi phần tử trong mảng đó **luôn là 8 bytes** trên kiến trúc 64-bit.
    *   Sai lầm phổ biến là lấy kích thước của kiểu dữ liệu đích (ví dụ lấy 2 bytes cho `short *T[2]`). Hãy nhớ: Mảng chứa cái gì thì tính kích thước của cái đó. Ở đây nó chứa địa chỉ.

2.  **Tính toán Offset:**
    *   Hệ số nhân ($Scale$) trong công thức địa chỉ chính là kích thước phần tử $L$. 
    *   Trong mã máy, điều này tương ứng với các giá trị scale của x86-64 (1, 2, 4, 8).

---

### IDA Pro Insights (Kỹ năng nhận diện mảng con trỏ)

Khi bạn gặp một lệnh truy cập mảng trong IDA Pro:

1.  **Dấu hiệu mảng dữ liệu vs Mảng con trỏ:**
    *   `mov eax, [rdx + rcx * 4]` $\rightarrow$ Scale là 4. Khả năng cao đây là mảng `int` hoặc `float` (Mảng dữ liệu).
    *   `mov rax, [rdx + rcx * 8]` $\rightarrow$ Scale là 8. Đây có thể là mảng `long` hoặc là một **mảng con trỏ**. 
    *   Nếu sau lệnh này, giá trị trong `rax` lại tiếp tục được dùng làm địa chỉ (ví dụ: `mov rsi, [rax]`), thì chắc chắn mảng ban đầu là một mảng con trỏ (như các mảng R, S, T trong bài tập).

2.  **Xác định kích thước mảng:**
    *   IDA thường hiển thị các mảng toàn cục dưới dạng nhãn. Nếu bạn thấy nhãn `ArrayP` và ngay dưới đó là `ArrayQ` cách nhau 20 bytes (0x14), bạn có thể suy luận mảng P có tổng kích thước là 20 bytes. 
    *   Kết hợp với lệnh truy cập dùng scale 4, bạn tính được số phần tử: $20 / 4 = 5$ phần tử. Đây là cách chúng ta khôi phục lại khai báo `int P[5]` từ file binary.

---

## 3.8.2 Pointer Arithmetic (Số học con trỏ)

Ngôn ngữ C cho phép thực hiện các phép toán cộng/trừ trên con trỏ. Giá trị tính toán được sẽ được **tỉ lệ hóa (scaled)** theo kích thước của kiểu dữ liệu mà con trỏ trỏ tới. 
*   Nếu `p` là con trỏ kiểu `T*` và có giá trị địa chỉ là $x_p$.
*   Biểu thức `p + i` trong C sẽ có giá trị địa chỉ thực tế là: $x_p + L \cdot i$ (với $L$ là kích thước của kiểu dữ liệu `T`).

Các toán tử đơn phân `&` (lấy địa chỉ) và `*` (giải mã con trỏ) cho phép chuyển đổi qua lại giữa con trỏ và giá trị:
*   Nếu `Expr` là một đối tượng, thì `&Expr` là con trỏ tới đối tượng đó.
*   Nếu `AExpr` là một địa chỉ, thì `*AExpr` là giá trị tại địa chỉ đó.
*   **Định đẳng thức mảng:** Biểu thức mảng `A[i]` hoàn toàn tương đương với `*(A + i)`.

---

### Bảng phân tích các biểu thức mảng và con trỏ

Giả sử: Địa chỉ bắt đầu của mảng số nguyên `E` (kiểu `int`) nằm trong `%rdx`, và chỉ số `i` nằm trong `%rcx`.

| Biểu thức C | Kiểu trả về | Giá trị logic | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) |
| :--- | :--- | :--- | :--- | :--- |
| `E` | `int *` | $x_E$ | `movq %rdx, %rax` | `mov rax, rdx` |
| `E[0]` | `int` | $M[x_E]$ | `movl (%rdx), %eax` | `mov eax, [rdx]` |
| `E[i]` | `int` | $M[x_E + 4i]$ | `movl (%rdx,%rcx,4), %eax`| `mov eax, [rdx + rcx*4]` |
| `&E[2]` | `int *` | $x_E + 8$ | `leaq 8(%rdx), %rax` | `lea rax, [rdx + 8]` |
| `E + i - 1`| `int *` | $x_E + 4i - 4$ | `leaq -4(%rdx,%rcx,4), %rax`| `lea rax, [rdx + rcx*4 - 4]` |
| `*(E + i - 3)`| `int` | $M[x_E + 4i - 12]$| `movl -12(%rdx,%rcx,4), %eax`| `mov eax, [rdx + rcx*4 - 12]` |
| `&E[i] - E`| `long` | $i$ | `movq %rcx, %rax` | `mov rax, rcx` |

---

### Phân tích kỹ thuật chuyên sâu

1.  **Sự khác biệt giữa `mov` và `lea`:**
    *   Hãy nhìn vào `&E[2]`: Trình biên dịch dùng lệnh **`leaq`**. Nó chỉ tính toán địa chỉ $x_E + 8$ rồi nạp vào `%rax`. Không có thao tác truy cập RAM nào xảy ra.
    *   Hãy nhìn vào `E[i]`: Trình biên dịch dùng lệnh **`movl`**. Nó tính toán địa chỉ $x_E + 4i$, sau đó **truy cập vào RAM** tại địa chỉ đó để lấy giá trị nạp vào `%eax`.

2.  **Kích thước thanh ghi:**
    *   Các phép toán trả về **giá trị** của mảng `int` (4 bytes) sử dụng thanh ghi 32-bit như **`%eax`**.
    *   Các phép toán trả về **địa chỉ/con trỏ** (8 bytes) sử dụng thanh ghi 64-bit như **`%rax`**.

3.  **Phép trừ hai con trỏ (`&E[i] - E`):**
    *   Trong C, hiệu của hai con trỏ cùng kiểu dữ liệu sẽ trả về khoảng cách tính theo **số lượng phần tử** (kiểu `long`), không phải số lượng byte.
    *   Vì `&E[i]` có địa chỉ $x_E + 4i$ và `E` có địa chỉ $x_E$, khoảng cách byte là $4i$. Khi chia cho kích thước kiểu `int` (4 bytes), kết quả trả về đúng bằng chỉ số **`i`**. Trình biên dịch chỉ đơn giản là copy giá trị `i` từ `%rcx` sang `%rax`.

---

### IDA Pro Insights (Kỹ năng nhận diện logic con trỏ)

1.  **LEA là "Máy tính địa chỉ":** Trong IDA, khi bạn thấy lệnh `lea`, hãy hiểu ngay mã nguồn C đang dùng toán tử `&` hoặc đang tính toán vị trí của một phần tử mảng nhưng **chưa lấy giá trị** tại đó.
2.  **Scale Factor (Hệ số tỉ lệ):** Trình biên dịch tự động nhúng kích thước của kiểu dữ liệu vào lệnh máy. 
    *   Nếu bạn thấy `[rdx + rcx*4]`, bạn biết đó là mảng các phần tử 4-byte (`int`, `float`). 
    *   Nếu bạn thấy `[rdx + rcx*8]`, đó là mảng các phần tử 8-byte (`long`, `double`, `pointer`).
3.  **Offset âm:** Trong lệnh `mov eax, [rdx + rcx*4 - 12]`, giá trị `-12` chính là kết quả của $3 \times 4 \text{ bytes}$. Trình biên dịch đã tính toán trước các hằng số để gộp vào một lệnh duy nhất nhằm tối ưu tốc độ. Khi dịch ngược, bạn lấy hằng số này chia cho Scale (12/4 = 3) để tìm ra chỉ số lùi trong mã C.

---

## 3.8.3 Nested Arrays (Mảng lồng nhau / Mảng đa chiều)

Trong C, mảng đa chiều thực chất là "mảng của các mảng". Các nguyên tắc cấp phát và tham chiếu mảng vẫn giữ nguyên ngay cả khi chúng ta lồng chúng lại.

**Ví dụ:** Khai báo `int A[5][3];`
*   Tương đương với việc định nghĩa một kiểu dữ liệu mới là mảng 3 số nguyên: `typedef int row3_t[3];` sau đó khai báo `row3_t A[5];`.
*   **Tổng kích thước:** $4 \text{ (int)} \times 5 \text{ (hàng)} \times 3 \text{ (cột)} = 60$ bytes.

### Thứ tự lưu trữ: Row-Major Order (Thứ tự ưu tiên hàng)
Kiến trúc C sắp xếp các mảng đa chiều trong bộ nhớ theo hàng. Nghĩa là toàn bộ các phần tử của hàng 0 (`A[0]`) được đặt trước, sau đó đến toàn bộ hàng 1 (`A[1]`), và cứ thế tiếp tục.

### Công thức tính địa chỉ tổng quát
Cho mảng được khai báo là `T D[R][C];` (Kiểu dữ liệu `T`, `R` hàng, `C` cột):
Địa chỉ của phần tử `D[i][j]` được tính theo công thức:
$$\&D[i][j] = x_D + L(C \cdot i + j)$$
Trong đó:
*   $x_D$: Địa chỉ bắt đầu của mảng.
*   $L$: Kích thước (byte) của kiểu dữ liệu $T$.
*   $C$: Số cột (quan trọng nhất, dùng để xác định độ dài một hàng).

---

### Hình 3.36: Minh họa truy cập mảng $5 \times 3$ trong Assembly

Giả sử chúng ta muốn truy cập phần tử `A[i][j]` của mảng `int A[5][3]`.
*   **Địa chỉ gốc $x_A$:** nằm trong `%rdi`.
*   **Chỉ số hàng $i$:** nằm trong `%rsi`.
*   **Chỉ số cột $j$:** nằm trong `%rdx`.

Trình biên dịch GCC thực hiện phép tính địa chỉ $x_A + 4(3i + j)$ bằng chuỗi lệnh sau:

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `leaq (%rsi,%rsi,2), %rax` | `lea rax, [rsi + rsi*2]` | Tính $3i$ (3 là số cột). |
| 2 | `leaq (%rdi,%rax,4), %rax` | `lea rax, [rdi + rax*4]` | Tính $x_A + 4(3i) = x_A + 12i$. |
| 3 | `movl (%rax,%rdx,4), %eax` | `mov eax, [rax + rdx*4]` | Lấy giá trị tại $x_A + 12i + 4j$. |

**Kết luận:** Mã máy sử dụng khả năng tỉ lệ hóa (scaling) và cộng của kiến trúc x86-64 để tính toán địa chỉ phần tử một cách tối ưu.

---

### IDA Pro Insights (Kỹ năng nhận diện mảng đa chiều)

Khi bạn gặp một chuỗi lệnh như trên trong IDA Pro, đây là cách để "đọc vị" cấu trúc mảng:

1.  **Nhận diện số Cột (Columns):** 
    *   Hãy nhìn vào lệnh `lea` đầu tiên. Nếu bạn thấy nó nhân một chỉ số với một hằng số (ví dụ `rsi * 2 + rsi` là nhân 3), thì **3** chính là số cột của mảng.
    *   Trong công thức mã máy, số cột luôn đi kèm với chỉ số hàng (`i`).
2.  **Xác định địa chỉ cơ sở:**
    *   Thanh ghi được cộng vào ở bước thứ 2 (ở đây là `rdi`) chính là địa chỉ bắt đầu của mảng đa chiều.
3.  **Hệ số Scale cuối cùng:**
    *   Lệnh `mov` cuối cùng sử dụng `rdx * 4`. Hệ số **4** cho biết kích thước mỗi phần tử là 4-byte (mảng `int` hoặc `float`).
4.  **Mẹo dịch ngược:** 
    *   Nếu bạn thấy: `Address = Base + L*C*i + L*j`
    *   Hãy viết lại trong C là: `Base[i][j]` với số cột là `C`. 
    *   Trong IDA, nếu đây là mảng trên Stack, các lệnh `lea` có thể trông phức tạp hơn vì nó cộng thêm cả offset của Frame (ví dụ `[rsp + 18h + rax*4]`).

---

### Practice Problem 3.38: Xác định kích thước mảng đa chiều

**Mã nguồn C:**
```c
long P[M][N];
long Q[N][M];

long sum_element(long i, long j) {
    return P[i][j] + Q[j][i];
}
```

#### 1. Phân tích mã Assembly (ATT vs Intel/IDA Pro)
*Quy ước: i trong %rdi, j trong %rsi*

| Dòng | AT&T Syntax | Intel Syntax (IDA Pro) | Phân tích logic |
| :--- | :--- | :--- | :--- |
| 2 | `leaq 0(,%rdi,8), %rdx` | `lea rdx, [rdi*8]` | `rdx = 8 * i` |
| 3 | `subq %rdi, %rdx` | `sub rdx, rdi` | `rdx = 8i - i = 7i` |
| 4 | `addq %rsi, %rdx` | `add rdx, rsi` | **`rdx = 7i + j`** |
| 5 | `leaq (%rsi,%rsi,4), %rax` | `lea rax, [rsi + rsi*4]` | `rax = 5 * j` |
| 6 | `addq %rax, %rdi` | `add rdi, rax` | **`rdi = 5j + i`** |
| 7 | `movq Q(,%rdi,8), %rax` | `mov rax, Q[rdi*8]` | Đọc từ mảng Q: $Q + 8(5j + i)$ |
| 8 | `addq P(,%rdx,8), %rax` | `add rax, P[rdx*8]` | Cộng với mảng P: $P + 8(7i + j)$ |

---

#### 2. Xác định giá trị $N$ (Số cột của mảng P)
*   Công thức truy cập `P[i][j]` là: $\text{Base}_P + 8(N \cdot i + j)$.
*   Từ dòng 8 và dòng 4, ta thấy mã máy tính toán địa chỉ: $P + 8(7i + j)$.
*   Đối chiếu hai công thức: $N \cdot i + j = 7i + j \Rightarrow \mathbf{N = 7}$.

#### 3. Xác định giá trị $M$ (Số cột của mảng Q)
*   Công thức truy cập `Q[j][i]` (lưu ý $j$ là hàng, $i$ là cột) là: $\text{Base}_Q + 8(M \cdot j + i)$.
*   Từ dòng 7 và dòng 6, ta thấy mã máy tính toán địa chỉ: $Q + 8(5j + i)$.
*   Đối chiếu hai công thức: $M \cdot j + i = 5j + i \Rightarrow \mathbf{M = 5}$.

**Kết luận:** $M = 5$ và $N = 7$.

---

### IDA Pro Insights (Kỹ năng suy luận cấu trúc mảng)

Khi bạn gặp các phép tính địa chỉ mảng đa chiều trong IDA Pro:

1.  **Nhìn vào các phép nhân/LEA:**
    *   Trình biên dịch rất ít khi dùng lệnh `imul` để tính chỉ số mảng vì nó chậm. Thay vào đó, nó dùng chuỗi `lea`, `shl`, `sub` để nhân hằng số.
    *   Khi thấy `i` bị nhân với một số (như $7$ hoặc $5$ trong bài này), con số đó chính là **số lượng cột** ($C$) của mảng.
2.  **Mẹo tính hệ số nhân:** 
    *   `lea rdx, [rdi*8]; sub rdx, rdi` $\rightarrow$ Nhân với $(8-1) = 7$.
    *   `lea rax, [rsi + rsi*4]` $\rightarrow$ Nhân với $(1+4) = 5$.
3.  **Xác định chiều mảng:** 
    *   Trong mã máy, thông tin về số hàng ($R$) thường bị mất (nếu không có kiểm tra biên). Nhưng số cột ($C$) bắt buộc phải xuất hiện trong mã máy để CPU biết được độ dài của mỗi hàng khi "nhảy" qua lại.
    *   Nếu bạn thấy `P[i][j]`, hãy tìm hằng số đi kèm với `i`.
    *   Nếu bạn thấy `Q[j][i]`, hãy tìm hằng số đi kèm với `j`.

---

## 3.8.4 Fixed-Size Arrays (Mảng kích thước cố định)

Trình biên dịch C có khả năng thực hiện nhiều tối ưu hóa cho các đoạn mã vận hành trên mảng đa chiều có kích thước cố định. Ở mức tối ưu hóa `-O1`, GCC sẽ chuyển đổi cách truy cập mảng từ việc sử dụng chỉ số (`i, j`) sang sử dụng các **con trỏ** để giảm thiểu các phép tính nhân phức tạp bên trong vòng lặp.

### Ví dụ: Nhân vô hướng hàng và cột của Ma trận

Giả sử chúng ta có kiểu dữ liệu ma trận $16 \times 16$:
```c
#define N 16
typedef int fix_matrix[N][N];
```

Hàm dưới đây tính toán phần tử $(i, k)$ của tích hai ma trận A và B (tương đương với tích vô hướng của hàng $i$ trong A và cột $k$ trong B).

---

### Hình 3.37: So sánh mã C gốc và mã C đã tối ưu hóa

#### (a) Original C code (Mã gốc)
```c
int fix_prod_ele(fix_matrix A, fix_matrix B, long i, long k) {
    long j;
    int result = 0;
    for (j = 0; j < N; j++)
        result += A[i][j] * B[j][k];
    return result;
}
```

#### (b) Optimized C code (Mã tối ưu hóa - Mô phỏng logic máy)
Trình biên dịch loại bỏ biến chạy `j`, thay thế bằng 3 con trỏ:
1.  **`Aptr`**: Trỏ vào các phần tử liên tiếp trong hàng $i$ của A.
2.  **`Bptr`**: Trỏ vào các phần tử liên tiếp trong cột $k$ của B.
3.  **`Bend`**: Địa chỉ dừng của vòng lặp (vị trí kết thúc cột $k$ của B).

```c
int fix_prod_ele_opt(fix_matrix A, fix_matrix B, long i, long k) {
    int *Aptr = &A[i][0];    // Trỏ vào đầu hàng i của A
    int *Bptr = &B[0][k];    // Trỏ vào đầu cột k của B
    int *Bend = &B[N][k];    // Điểm dừng (sau phần tử cuối cột k)
    int result = 0;
    do {
        result += *Aptr * *Bptr;
        Aptr++;              // Tiến 1 phần tử (4 bytes)
        Bptr += N;           // Nhảy xuống hàng tiếp theo (16 * 4 = 64 bytes)
    } while (Bptr != Bend);
    return result;
}
```

---

### Phân tích mã Assembly (Hình 3.37c)

**Ánh xạ thanh ghi:** `A` (%rdi), `B` (%rsi), `i` (%rdx), `k` (%rcx).
**Thanh ghi tối ưu:** `%eax` (result), `%rdi` (Aptr), `%rcx` (Bptr), `%rsi` (Bend).

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `fix_prod_ele:` | `fix_prod_ele:` | Nhãn hàm. |
| 2 | `salq $6, %rdx` | `shl rdx, 6` | `rdx = i * 64`. (1 hàng = 16 int * 4B = 64B). |
| 3 | `addq %rdx, %rdi` | `add rdi, rdx` | **`Aptr = A + 64*i`**. |
| 4 | `leaq (%rsi,%rcx,4), %rcx` | `lea rcx, [rsi + rcx*4]` | **`Bptr = B + 4*k`**. |
| 5 | `leaq 1024(%rcx), %rsi` | `lea rsi, [rcx + 1024]` | **`Bend = Bptr + 1024`**. ($16 \times 64 = 1024$). |
| 6 | `movl $0, %eax` | `mov eax, 0` | `result = 0`. |
| 7 | `.L7:` | `loc_loop:` | **Thân vòng lặp (loop):** |
| 8 | `movl (%rdi), %edx` | `mov edx, [rdi]` | Lấy giá trị `*Aptr`. |
| 9 | `imull (%rcx), %edx` | `imul edx, [rcx]` | Nhân với `*Bptr`. |
| 10| `addl %edx, %eax` | `add eax, edx` | `result += ...` |
| 11| `addq $4, %rdi` | `add rdi, 4` | **`Aptr++`** (Tiến 4 bytes). |
| 12| `addq $64, %rcx` | `add rcx, 64` | **`Bptr += 16`** (Tiến 16 hàng = 64 bytes). |
| 13| `cmpq %rsi, %rcx` | `cmp rcx, rsi` | So sánh `Bptr` với `Bend`. |
| 14| `jne .L7` | `jnz short loc_loop` | Nếu chưa bằng, tiếp tục lặp. |
| 15| `rep; ret` | `retn` | Trả về `result`. |

---

### IDA Pro Insights (Kỹ năng đọc code ma trận)

1.  **Hằng số dịch chuyển (Stride):** 
    *   Trong IDA, khi thấy lệnh `add rcx, 64` bên trong vòng lặp, đây là dấu hiệu rõ rệt của việc truy cập **theo cột** trong một ma trận có độ rộng hàng là 64 bytes ($16 \times 4$ bytes).
    *   Lệnh `add rdi, 4` cho thấy việc truy cập **theo hàng** (tiến từng phần tử liền kề).
2.  **Nhận diện kích thước ma trận:**
    *   Hằng số `1024` trong lệnh `lea rsi, [rcx + 1024]` là chìa khóa. Nó cho biết tổng số byte từ phần tử đầu tiên đến phần tử cuối cùng của cột. Vì mỗi bước nhảy là 64, ta tính được số hàng: $1024 / 64 = 16$ hàng.
    *   Kết hợp với bước nhảy hàng 64 bytes, ta suy ra ma trận là $16 \times 16$.
3.  **Tối ưu hóa con trỏ:** 
    *   Trình biên dịch đã biến vòng lặp `for` (kiểm tra ở đầu) thành `do-while` (kiểm tra ở cuối) vì nó biết chắc chắn $N=16 > 0$. Điều này làm Graph View trong IDA trông rất gọn (chỉ có một mũi tên quay ngược duy nhất).
4.  **Sử dụng lại thanh ghi:** 
    *   Lưu ý cách thanh ghi `%rsi` ban đầu chứa địa chỉ `B`, nhưng sau đó bị ghi đè để chứa địa chỉ `Bend` (dòng 5). IDA sẽ theo dõi sự thay đổi này, bạn nên đổi tên nó từ `arg_B` thành `var_Bend` sau dòng 5.

---

### Practice Problem 3.39: Chứng minh công thức tính toán địa chỉ

**Đề bài:** Sử dụng Phương trình 3.1 ($\&D[i][j] = x_D + L(C \cdot i + j)$) để giải thích cách tính các giá trị khởi tạo cho `Aptr`, `Bptr`, và `Bend` trong mã Assembly của hàm `fix_prod_ele`.

**Thông số ma trận:** $L = 4$ (kiểu `int`), $C = 16$ (số cột $N=16$).

<details>
<summary><b>Nhấn để xem phân tích toán học</b></summary>

1.  **`Aptr` (&A[i][0]):**
    *   Áp dụng công thức: $x_A + 4(16 \cdot i + 0) = x_A + 64i$.
    *   Trong Assembly (Hình 3.37c): Lệnh 2 tính `64*i` (`salq $6, %rdx`), lệnh 3 cộng vào `x_A` (`addq %rdx, %rdi`). **Khớp.**

2.  **`Bptr` (&B[0][k]):**
    *   Áp dụng công thức: $x_B + 4(16 \cdot 0 + k) = x_B + 4k$.
    *   Trong Assembly: Lệnh 4 tính `x_B + 4*k` (`leaq (%rsi,%rcx,4), %rcx`). **Khớp.**

3.  **`Bend` (&B[N][k]):**
    *   Áp dụng công thức: $x_B + 4(16 \cdot 16 + k) = x_B + 4 \cdot 256 + 4k = (x_B + 4k) + 1024$.
    *   Trong Assembly: Lệnh 5 cộng thêm `1024` vào `Bptr` (`leaq 1024(%rcx), %rsi`). **Khớp.**

</details>

---

### Practice Problem 3.40: Tối ưu hóa truy cập đường chéo ma trận

**Đề bài:** Phân tích mã Assembly của hàm `fix_set_diag` và viết lại phiên bản C tối ưu hóa (`fix_set_diag_opt`) mô phỏng đúng logic của trình biên dịch.

#### 1. Phân tích mã Assembly (GCC -O1)
*Quy ước: A in %rdi, val in %rsi*

| Dòng | AT&T Syntax | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 2 | `movl $0, %eax` | `mov eax, 0` | **`offset = 0`** (Dùng làm bước nhảy byte). |
| 4 | `movl %esi, (%rdi,%rax)` | `mov [rdi + rax], esi` | **`*(A + offset) = val`**. |
| 5 | **`addq $68, %rax`** | **`add rax, 68`** | **`offset += 68`**. |
| 6 | `cmpq $1088, %rax` | `cmp rax, 1088` | So sánh offset với 1088. |
| 7 | `jne .L13` | `jnz short loc_loop` | Nếu chưa tới 1088, tiếp tục lặp. |

#### 2. Tại sao hằng số lại là 68 và 1088?
*   Địa chỉ của phần tử $A[i][i]$ là: $x_A + 4(16 \cdot i + i) = x_A + 4(17i) = x_A + \mathbf{68}i$.
*   Khoảng cách giữa hai phần tử đường chéo liên tiếp ($A[i][i]$ và $A[i+1][i+1]$) là **68 bytes**.
*   Vòng lặp chạy 16 lần, nên điểm dừng là $16 \times 68 = \mathbf{1088}$ bytes.

#### 3. Viết lại mã C tối ưu hóa (`fix_set_diag_opt`)

```c
void fix_set_diag_opt(fix_matrix A, int val) {
    int *Aptr = &A[0][0];      // Khởi tạo con trỏ tại phần tử đầu tiên
    int *Aend = &A[N][N];      // Điểm kết thúc (A + 1088 bytes)
    do {
        *Aptr = val;           // Gán giá trị
        Aptr += (N + 1);       // Nhảy đến phần tử đường chéo tiếp theo (16 + 1 = 17 int)
    } while (Aptr != Aend);
}
```

---

### IDA Pro Insights (Kỹ năng nhận diện hằng số ma trận)

1.  **Hằng số bước nhảy (Stride):**
    *   Trong IDA, nếu bạn thấy một vòng lặp truy cập mảng mà chỉ số tăng một lượng không phải 1, 2, 4, hay 8 (ví dụ `add rax, 68`), đây là dấu hiệu của việc truy cập các phần tử có quy luật đặc biệt (như đường chéo hoặc một trường cụ thể trong mảng các Struct).
    *   Để tìm ra logic C, hãy lấy số đó chia cho kích thước kiểu dữ liệu: $68 / 4 = 17$. 
    *   Trong ma trận $N \times N$, bước nhảy $N+1$ luôn là **đường chéo chính**.
2.  **Nhận diện "Flat access":**
    *   Trình biên dịch đã loại bỏ hoàn toàn các biến `i, j` và các phép nhân. Nó chỉ dùng một thanh ghi duy nhất (`rax`) để lưu trữ độ lệch byte (byte offset) từ đầu mảng.
    *   Mẫu hình `[base + offset]` với `offset` tăng tiến là cách IDA hiển thị mã đã được tối ưu hóa cực hạn.
3.  **Hằng số dừng (1088):**
    *   Khi thấy số `1088`, hãy chuyển sang Hex (`440h`) hoặc thực hiện phép chia để đoán kích thước mảng. $1088 / 68 = 16$. Bạn biết ngay vòng lặp này chạy đúng 16 lần.

---

## 3.8.5 Variable-Size Arrays (Tiếp theo)

Trong ví dụ về hàm `var_prod_ele`, trình biên dịch tối ưu hóa vòng lặp bằng cách sử dụng các thanh ghi để lưu trữ các giá trị trung gian, tránh việc nhân lặp đi lặp lại trong mỗi bước.

### Phân tích mã Assembly của vòng lặp `var_prod_ele`

**Ánh xạ thanh ghi:**
*   `n`: `%rdi` (Dùng để kiểm tra biên vòng lặp).
*   `Arow` (Con trỏ tới hàng $i$ của mảng A): `%rsi`.
*   `Bptr` (Con trỏ tới phần tử trong cột $k$ của mảng B): `%rcx`.
*   **`4n`** (Giá trị $n$ đã nhân tỉ lệ 4 bytes): **`%r9`** (Dùng để tăng `Bptr`).
*   `result`: `%eax`.
*   `j` (Chỉ số vòng lặp): `%rdx`.

| Dòng | AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `.L24:` | `loc_L24:` | **Nhãn vòng lặp.** |
| 2 | `movl (%rsi,%rdx,4), %r8d` | `mov r8d, [rsi+rdx*4]` | Đọc `Arow[j]` (4 bytes). |
| 3 | `imull (%rcx), %r8d` | `imul r8d, [rcx]` | Nhân với `*Bptr`. |
| 4 | `addl %r8d, %eax` | `add eax, r8d` | `result += ...` |
| 5 | `addq $1, %rdx` | `add rdx, 1` | `j++` |
| 6 | `addq %r9, %rcx` | `add rcx, r9` | **`Bptr += n`** (Nhảy xuống hàng tiếp theo). |
| 7 | `cmpq %rdi, %rdx` | `cmp rdx, rdi` | So sánh `j` với `n`. |
| 8 | `jne .L24` | `jnz short loc_L24` | Nếu `j != n`, tiếp tục lặp. |

---

### Phân tích kỹ thuật: Sự xuất hiện của hai giá trị $n$

Một điểm đặc biệt trong đoạn mã này là trình biên dịch duy trì hai hình thái của biến `n` trong các thanh ghi khác nhau:
1.  **Giá trị thực `n` (`%rdi`):** Được dùng ở dòng 7 để so sánh với chỉ số `j`, quyết định khi nào vòng lặp kết thúc.
2.  **Giá trị tỉ lệ `4n` (`%r9`):** Được tính toán trước khi vào vòng lặp và dùng ở dòng 6. Vì mỗi hàng của mảng `int` chứa `n` phần tử, mỗi phần tử chiếm 4 bytes, nên để nhảy từ hàng này sang hàng tiếp theo ở cùng một cột, con trỏ phải tăng một lượng là $4n$ bytes.

Việc chuẩn bị sẵn `4n` giúp loại bỏ phép nhân bên trong thân vòng lặp, giúp chương trình thực thi nhanh hơn đáng kể.

---

### IDA Pro Insights (Kỹ năng nhận diện bước nhảy mảng)

Khi bạn gặp một vòng lặp duyệt mảng đa chiều trong IDA Pro:

1.  **Xác định Stride (Bước nhảy):** Lệnh `add rcx, r9` (hoặc bất kỳ thanh ghi nào cộng vào một thanh ghi địa chỉ) là dấu hiệu của việc duyệt mảng theo cột. 
    *   Nếu thanh ghi được cộng vào (`r9`) có giá trị phụ thuộc vào một tham số đầu vào (`n`), bạn có thể khẳng định đây là **mảng có kích thước biến đổi (Variable-size array)**.
2.  **Tính toán kích thước hàng:** Trong IDA, hãy trace ngược lại xem thanh ghi `r9` được tính như thế nào. Nếu bạn thấy `shl r9, 2` (dịch trái 2 bit = nhân 4), bạn biết mỗi phần tử mảng chiếm 4 bytes (kiểu `int`).
3.  **Sự thông minh của Compiler:** Trình biên dịch có thể nhận diện các mẫu hình (patterns) khi chương trình duyệt qua các phần tử của mảng đa chiều và tự động tạo ra mã dựa trên con trỏ (pointer-based) để tránh phép nhân theo công thức địa chỉ tiêu chuẩn. Điều này giúp cải thiện hiệu suất nhưng làm mã assembly khó đọc hơn so với mã C ban đầu.

---

# 3.9 Heterogeneous Data Structures

Ngôn ngữ C cung cấp hai cơ chế để tạo ra các kiểu dữ liệu bằng cách kết hợp các đối tượng thuộc các kiểu khác nhau:
1.  **Structures (Cấu trúc):** Khai báo bằng từ khóa `struct`, gộp nhiều đối tượng vào một đơn vị duy nhất.
2.  **Unions (Liên hợp):** Khai báo bằng từ khóa `union`, cho phép một đối tượng được tham chiếu bằng nhiều kiểu dữ liệu khác nhau.

---

## 3.9.1 Structures

Cấu trúc `struct` tạo ra một kiểu dữ liệu nhóm các đối tượng (các trường - fields) có thể khác kiểu nhau. 

**Nguyên tắc triển khai mức máy:**
*   **Vùng nhớ liên tục:** Tất cả các thành phần của một struct được lưu trữ trong một vùng nhớ liên tục.
*   **Địa chỉ gốc:** Con trỏ trỏ tới struct chính là địa chỉ của byte đầu tiên của nó.
*   **Offset (Độ lệch):** Trình biên dịch duy trì thông tin về từng kiểu struct để xác định độ lệch byte của mỗi trường. Nó sử dụng các offset này làm hằng số (displacement) trong các lệnh tham chiếu bộ nhớ.

---

### Ví dụ minh họa: `struct rec`

Xét khai báo cấu trúc sau:
```c
struct rec {
    int i;     // 4 bytes
    int j;     // 4 bytes
    int a[2];  // 8 bytes (2 * 4 bytes)
    int *p;    // 8 bytes
};
```

**Sơ đồ phân bổ bộ nhớ (Layout):**
Tổng kích thước struct là **24 bytes**.

| Trường (Field) | Offset (Byte) | Kích thước | Nội dung |
| :--- | :---: | :---: | :--- |
| `i` | **0** | 4 | Số nguyên `int` |
| `j` | **4** | 4 | Số nguyên `int` |
| `a[0]` | **8** | 4 | Phần tử mảng `int` |
| `a[1]` | **12** | 4 | Phần tử mảng `int` |
| `p` | **16** | 8 | Con trỏ `int *` |

---

### Truy cập các trường trong Assembly (ATT vs Intel/IDA Pro)

Giả sử biến `r` kiểu `struct rec *` nằm trong thanh ghi **`%rdi`**.

#### 1. Truy cập trường đơn giản: `r->j = r->i;`
| AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- |
| `movl (%rdi), %eax` | `mov eax, [rdi]` | Lấy `r->i` (offset 0). |
| `movl %eax, 4(%rdi)` | `mov [rdi+4], eax` | Lưu vào `r->j` (offset 4). |

#### 2. Lấy địa chỉ phần tử mảng: `&r->a[i]` (với `i` nằm trong `%rsi`)
| AT&T Syntax (Sách) | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- |
| `leaq 8(%rdi,%rsi,4), %rax`| `lea rax, [rdi+rsi*4+8]` | **Base + Offset_a + i*4**. |

#### 3. Ví dụ phức tạp: `r->p = &r->a[r->i + r->j];`
Chuỗi lệnh thực hiện phép tính này:

| Dòng | AT&T Syntax | Intel Syntax (IDA Pro) | Ý nghĩa logic |
| :--- | :--- | :--- | :--- |
| 1 | `movl 4(%rdi), %eax` | `mov eax, [rdi+4]` | Lấy `r->j`. |
| 2 | `addl (%rdi), %eax` | `add eax, [rdi]` | Cộng thêm `r->i`. |
| 3 | `cltq` | `cdqe` | Mở rộng `int` lên 64-bit. |
| 4 | `leaq 8(%rdi,%rax,4), %rax`| `lea rax, [rdi+rax*4+8]` | Tính địa chỉ phần tử mảng. |
| 5 | `movq %rax, 16(%rdi)` | `mov [rdi+16], rax` | Lưu vào `r->p` (offset 16). |

---

### Bài học then chốt về Reverse Engineering:
Việc lựa chọn các trường khác nhau của một struct được xử lý **hoàn toàn tại thời điểm biên dịch**. Trong mã máy (binary) mà IDA Pro hiển thị:
*   **Không có tên trường:** Mọi thứ chỉ là các con số offset (`+4`, `+8`, `+16`).
*   **Không có khai báo struct:** Mã máy không biết dữ liệu đó thuộc về một `struct` hay một mảng. Nó chỉ thấy các thao tác tính toán địa chỉ trên bộ nhớ.

---

### IDA Pro Insights (Kỹ năng khôi phục Struct)

Khi bạn thấy một thanh ghi địa chỉ (như `rdi`) liên tục được cộng với các hằng số cố định (`[rdi+4]`, `[rdi+16]`), đó là dấu hiệu của một **Structure**.

1.  **Tạo Struct:** Trong IDA, nhấn **Shift+F9** (Structures window) để tạo một struct mới dựa trên các offset bạn tìm thấy.
2.  **Áp dụng Struct:** Quay lại mã giả/mã assembly, chọn thanh ghi đó và nhấn phím **"T"**. IDA sẽ thay thế các offset vô hồn (`+16h`) bằng tên trường cụ thể (`->p`), làm code dễ đọc hơn rất nhiều.
3.  **Hậu tố kích thước:** Hãy để ý lệnh `movl` (4 bytes) vs `movq` (8 bytes). Đây là cách để bạn xác định kiểu dữ liệu của trường đó là `int` hay là con trỏ/`long`.
4.  **Lệnh `lea`:** Nếu thấy `lea` cộng hằng số vào địa chỉ struct, hãy nghĩ ngay đến toán tử **`&`** (lấy địa chỉ trường hoặc phần tử mảng bên trong struct).

---

### Kiến thức cho người mới học C: Biểu diễn đối tượng bằng Struct (Aside)

Khai báo `struct` trong C là thứ gần gũi nhất với các "đối tượng" (objects) trong C++ và Java. Nó cho phép lập trình viên giữ thông tin về một thực thể trong một cấu trúc dữ liệu duy nhất và tham chiếu đến thông tin đó bằng tên.

#### Ví dụ: Cấu trúc Hình chữ nhật (`struct rect`)
Một chương trình đồ họa có thể biểu diễn một hình chữ nhật bằng cấu trúc sau:

```c
struct rect {
    long llx;             /* Tọa độ X của góc dưới bên trái (8 bytes) */
    long lly;             /* Tọa độ Y của góc dưới bên trái (8 bytes) */
    unsigned long width;  /* Chiều rộng (pixels) (8 bytes)             */
    unsigned long height; /* Chiều cao (pixels) (8 bytes)              */
    unsigned color;       /* Mã màu (4 bytes)                          */
};
```

**Cách khai báo và truy cập:**
*   Dùng toán tử dấu chấm (`.`) để truy cập trường: `r.llx = 0;`.
*   Có thể khởi tạo nhanh: `struct rect r = { 0, 0, 0xFF00FF, 10, 20 };`.

---

### Truyền con trỏ Struct và toán tử Mũi tên (`->`)

Trong thực tế, người ta thường truyền **con trỏ** trỏ tới struct (`struct rect *rp`) thay vì sao chép toàn bộ struct vào hàm để tiết kiệm bộ nhớ và thời gian.

#### 1. Hàm tính diện tích (`area`)
```c
long area(struct rect *rp) {
    return (*rp).width * (*rp).height;
}
```
*   **Lưu ý cú pháp:** Cần dùng ngoặc đơn `(*rp).width` vì toán tử dấu chấm có độ ưu tiên cao hơn toán tử giải mã con trỏ. Nếu viết `*rp.width`, trình biên dịch sẽ hiểu lầm là `*(rp.width)`.
*   **Toán tử `->`:** Để thuận tiện, C cung cấp toán tử mũi tên. `rp->width` hoàn toàn tương đương với `(*rp).width`.

#### 2. Hàm xoay hình chữ nhật (`rotate_left`)
Hàm này xoay hình chữ nhật ngược chiều kim đồng hồ 90 độ:
```c
void rotate_left(struct rect *rp) {
    /* Tráo đổi chiều rộng và chiều cao */
    long t = rp->height;
    rp->height = rp->width;
    rp->width = t;
    /* Dịch chuyển sang góc dưới bên trái mới */
    rp->llx -= t;
}
```

---

### Sự khác biệt giữa C Struct và Đối tượng (C++/Java)
Các đối tượng trong C++ và Java phức tạp hơn struct trong C vì chúng liên kết cả các **phương thức** (methods - các hàm có thể được gọi để thực hiện tính toán) trực tiếp với đối tượng. Trong C, chúng ta chỉ có dữ liệu trong struct, còn các hàm (như `area` và `rotate_left`) được viết riêng biệt và nhận con trỏ struct làm tham số.

---

### IDA Pro Insights (Tư duy ánh xạ Struct thực tế)

Mặc dù trang này không có mã máy, nhưng cấu trúc `struct rect` trên sẽ giúp bạn hiểu cách IDA Pro tính toán địa chỉ:

1.  **Tính toán Offset:** Trình biên dịch sẽ tính sẵn vị trí các trường trong bộ nhớ:
    *   `llx`: Offset 0
    *   `lly`: Offset 8
    *   `width`: Offset 16 (10h)
    *   `height`: Offset 24 (18h)
    *   `color`: Offset 32 (20h)

2.  **Logic trong IDA:** Khi bạn xem hàm `area` trong IDA, bạn sẽ thấy:
    ```assembly
    mov rax, [rdi+10h]  ; Lấy rp->width (offset 16)
    imul rax, [rdi+18h] ; Nhân với rp->height (offset 24)
    retn
    ```
3.  **Kích thước dữ liệu:**
    *   Lưu ý `llx`, `lly`, `width`, `height` đều là 8 bytes, nên IDA sẽ dùng lệnh `mov rax` (64-bit).
    *   Trường `color` là `unsigned int` (4 bytes), nên IDA sẽ dùng lệnh `mov eax` (32-bit). Việc chú ý kích thước thanh ghi giúp bạn đoán đúng kiểu dữ liệu của trường trong struct.
4.  **Sự biến mất của tên:** Trong file thực thi thực tế, tên `llx` hay `width` biến mất hoàn toàn. IDA chỉ thấy `[rdi+10h]`. Việc định nghĩa Struct trong IDA (**Shift+F9**) và gán kiểu (**phím Y**) là cách duy nhất để mang những cái tên này quay trở lại mã giả.

---

### Practice Problem 3.42: Nghịch đảo mã cấu trúc Danh sách liên kết

**Khai báo cấu trúc C:**
```c
struct ACE {
    short v;      /* Giá trị: Offset 0, Size 2 bytes */
    struct ACE *p; /* Con trỏ kế tiếp: Offset 2, Size 8 bytes */
};
```

#### 1. Phân tích mã Assembly (ATT vs Intel/IDA Pro)
*Quy ước: `ptr` nằm trong `%rdi`, kết quả trả về `val` nằm trong `%rax`*

| Dòng | AT&T Syntax | Intel Syntax (IDA Pro) | Giải thích logic |
| :--- | :--- | :--- | :--- |
| 1 | `test:` | `test:` | Nhãn hàm. |
| 2 | `movl $1, %eax` | `mov eax, 1` | Khởi tạo **`val = 1`**. |
| 3 | `jmp .L2` | `jmp short loc_check` | **Jump-to-middle**: Nhảy tới phần kiểm tra. |
| 4 | `.L3:` | `loc_loop:` | **Thân vòng lặp (Body)**: |
| 5 | `imulq (%rdi), %rax` | `imul rax, [rdi]` | **`val *= ptr->v`**. (Nhân giá trị hiện tại). |
| 6 | `movq 2(%rdi), %rdi` | `mov rdi, [rdi+2]` | **`ptr = ptr->p`**. (Nhảy sang node kế tiếp). |
| 7 | `.L2:` | `loc_check:` | **Phần kiểm tra (Test)**: |
| 8 | `testq %rdi, %rdi` | `test rdi, rdi` | Kiểm tra **`ptr == NULL`**? |
| 9 | `jne .L3` | `jnz short loc_loop` | Nếu **`ptr != NULL`**, tiếp tục lặp. |
| 10 | `rep; ret` | `retn` | Trả về `val`. |

---

### Trả lời câu hỏi:

**A. Viết mã nguồn C cho hàm `test`:**

```c
short test(struct ACE *ptr) {
    short val = 1;
    while (ptr) {
        val *= ptr->v;
        ptr = ptr->p;
    }
    return val;
}
```

**B. Mô tả cấu trúc dữ liệu và thao tác mà hàm thực hiện:**
*   **Cấu trúc dữ liệu:** Đây là một **Danh sách liên kết đơn (Singly Linked List)**, trong đó mỗi node (`struct ACE`) chứa một giá trị số nguyên kiểu `short` và một con trỏ trỏ đến phần tử tiếp theo.
*   **Thao tác:** Hàm này thực hiện việc duyệt qua toàn bộ danh sách liên kết và tính **tích (product)** của tất cả các giá trị `v` có trong danh sách.

---

### IDA Pro Insights (Kỹ năng nhận diện Linked List)

Khi bạn gặp mẫu hình này trong IDA Pro, đây là những dấu hiệu nhận biết cấu trúc danh sách:

1.  **Mẫu hình "Self-Pointer Update":** 
    *   Lệnh `mov rdi, [rdi + offset]` (dòng 6) bên trong một vòng lặp là bằng chứng đanh thép của việc duyệt danh sách hoặc cây. Nó lấy địa chỉ ở node hiện tại để ghi đè vào chính con trỏ duyệt, nhằm "nhảy" sang node tiếp theo.
2.  **Kiểm tra NULL làm điều kiện lặp:**
    *   Lệnh `test rdi, rdi` theo sau bởi `jnz` dẫn ngược lên đầu vòng lặp là cách mã máy thực hiện `while (ptr != NULL)`.
3.  **Xác định Offset của trường `next`:**
    *   Trong bài này, `v` ở offset 0 và `p` ở offset 2. IDA sẽ hiển thị `[rdi+2]`. 
    *   *Lưu ý thực tế:* Trong các hệ thống hiện đại, do quy tắc căn lề (alignment), trường con trỏ `p` thường sẽ nằm ở offset 8 thay vì offset 2 để đạt hiệu năng tốt nhất. Nhưng trong ví dụ học thuật này, trình biên dịch đã nén chúng lại.
4.  **Lệnh nhân tích lũy:**
    *   Lệnh `imul rax, [rdi]` lấy dữ liệu tại địa chỉ node và nhân vào `rax`. IDA sẽ nhận diện `rax` là biến tích lũy (accumulator) vì nó được khởi tạo bằng 1 (`mov eax, 1`) - phần tử đơn vị của phép nhân.

