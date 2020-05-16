---
title: "Operating System - Labwork 1"
author: \textbf{Do Duy Huy Hoang} \newline
date: "2020-05-16"
titlepage: true
...

## Why we need to install kernel-package

Gói kernel phát triển giúp tự động hóa các bước thường quy cần thiết để biên dịch và cài đặt kernel tùy chỉnh.

## Why we have to use another kernel source from the server such as http://www.kernel.org, can we compile the original kernel (the local kernel on the running OS) directly?

Bạn có thể biên dịch hạt nhân gốc (the original kernel) trong tệp của OS đang sử dụng, vì hật nhân mặc định được vận chuyển với Ubuntu xử lý hầu hết các cấu hình. Ngoài ra, Ubuntu thường cung cấp một số hạt nhân thay thế, bạn cần kiểm tra trước hạt nhân này tương thích tốt với cấu hình phần cứng của bạn, tuy nhiên những lợi ích cụ thể biên dịch từ hạt nhân (kernel) mới từ server là để:

- Xử lý các nhu cầu phần cứng đặc biệt, hoặc xung đột phần cứng với hạt nhân được cung cấp trước
- Sử dụng các tùy chọn sử dụng hạt nhân mà không được hỗ trợ trong các hạt nhân được cung cấp trước (chẳng hạn như hỗ trợ bộ nhớ cao)
- Tối ưu hóa hạt nhân bằng cách loại bỏ các trình điều khiển vô ích để tăng tốc độ khởi động
- Tạo ra một monolithic thay vì một hạt nhân đã được modularized
- Chạy một cập nhật hạt nhân hoặc dành cho nhà phát triển

## What is the meaning of other parts, i.e. i386, procsched, and sys procsched?

- **Number**: Tất cả các syscalls được xác định bởi một số duy nhất. Để gọi một syscall, 
chúng ta nói với kernel để gọi syscall theo số của nó chứ không phải bằng tên của nó.
- **i386[ABI]** : Application Binary Interface - là interface giữa hai chương trình modules, một trong số
đó thường là thư viện hoặc hệ điều hành, ở mức mã máy phổ biến là x64, x32, i386
- **name** : đây là tên của syscall
- **entry point** : Điểm truy cập, là tên của hàm để gọi để xử lý syscall. Quy ước đặt tên cho
hàm này là tên của syscall có tiền tố với sys_. Ví dụ, điểm truy nhập của syscall đọc là sys_read.
- **compat entry point**: điểm bắt đầu khi gọi hàm thực thi tác vụ khi người dùng sử dụng máy 64bit.

## What is the meaning of each line above?

- Hàm sử lý syscall kiểu trả về long 
- `asmlinkage` là một tag được định nghĩa (#define) với một số gcc biên dịch cho trình trình biên dịch biết rằng hàm không mong chờ tìm thấy tất cả đối số trong thanh ghi (registers) một cách tối ưu phổ biến. Nhưng chỉ trên stack của CPU (có thể hiểu đơn giản là trình biên dịch biết cách tìm các đối số trên stack CPU). `systemcall` tiếp nhận đối số number đầu tiên, và cho phép 4 đối số đưuọc truyền vào hệ thống thực, các đối số này nằm trên stack. tất cả `systemcall` đánh dấu bởi `asmlinkage` tag nên chúng nhìn stack để tìm đối số, Nó cũng được sử dụng để cho phép gọi một hàm từ assembly files.

## What is the meaning of these two stages, namely “make” and “make modules”?
- **make modules**: Lệnh `make modules` sẽ chỉ biên dịch các modules, để lại các compiled binaries đã biên dịch trong thư mục build. 
- **make**: bản 2.6 trở về trước, bạn sẽ cần `make modules` nhưng các bản sau này thì `make` cũng đã kèm theo `make modules`

## Why this program could indicate whether our system works or not?

Gọi một `system call` bằng lệnh `syscall ([ number_32], 1,printf("My MSSV: %ul\n", info [0])` với `number_32` là name của `procsched system call` mà ta đã khai báo. Chương trình sẽ trả về giá trị trong con trỏ info được truyền vào. Ngược lại, nếu build kernel thất bại, chương trình sẽ dừng lại nên ta có thể dùng chương trình như một test program để kiểm tra kernel build có thành công hay không.

## Why we have to re-define proc segs struct while we have already defined it inside the kernel?

Hai `proc segs struct` nằm ở các space khác nhau, khi mà `proc segs struct` được khai báo trong kernel thì nó chỉ được sử dụng trong `kernel space`, còn mục đích khai báo lại để người dùng có thể sử dụng trong `user space` tương đương như một module.

## Why root privilege (e.g. adding sudo before the cp command) is required to copy the header file to /usr/include?

Vì `/usr/include` thuộc quyền sở hữu của root user, và lệnh sudo cho phép user trong `wheel` group được execute any command. (như quyền của root user).

## Why we must put -share and -fpic option into gcc command?

Vì mình muốn tạo một file share object, ở đây là `libprocsched.so` cũng như sử dụng shared library này, nên ta cần sử dụng option `-share` và `-fpic` trong gcc (`-share` dùng để generate share lib, conf `-fpic` giúp tạo ra một position independent code với mục đích để sử dụng shared lib).
