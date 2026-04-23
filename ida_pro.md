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

*(Nội dung bức ảnh dừng lại ở câu dẫn nhập vào mã nguồn file `.s`)*.

---

<tạm dừng>
