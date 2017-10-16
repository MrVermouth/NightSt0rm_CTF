# Level_2

- Bin: là file PE 32bit. Chỉ có duy nhất 1 file cho 3 flag

Sau khi mất 30p ở level_1, tôi bắt đầu với level_2. Với thói quen, tôi load binary vào IDA nhìn 1 vòng thì thấy nó được code bằng **C API**, 
dễ dàng tìm thấy hàm **WinMain** và **CallBack** của nó, vì chả biết tìm cái gì tôi dùng tổ hợp phím thần thành **Shift+F12**.

Rất nhiều chuỗi được hiển thị lên, đảo mắt 1 vòng thì đây là chuỗi quen thuộc của vc++. Sau đó tôi tập trung vào 2 chuỗi để in ra flag.

>The flag of level 2 is: NightSt0rm{%s3}

>The flag of level 3 is: NightSt0rm{%s}

Follow theo chuỗi level 2, tôi tìm được đoạn code làm việc liên quan tới việc lấy dữ liệu và kiểm tra để in ra chuỗi.

Sau khi chuyển sang mã giả và tiến hành phân tích, tôi thấy rằng đây là 1 bài toán **XOR**( người ra đề rất thích **XOR** ).
Input vào có độ dài là **len(input) = 24**, và được chia thành 3 block với mỗi block là 8 byte của input để tính toán.

#### Block 1

```C
if ( &input[strlen(input) + 1] - &input[1] == 24 )
          {
            input_base64 = *input;
            *v54 = *v83;
            base64_mode(&input_base64, &out_base64);
            check_base64 = strcmp(&out_base64, "olCOkyDvq7i=");
            if ( check_base64 )
              check_base64 = -(check_base64 < 0) | 1;
            if ( check_base64 )
            {
              MessageBoxA(0, "Your wallet  invalid \n Please try again", "Fail", 0);
            }
```

Ở hàm **strcmp**, dễ dàng thấy rằng **olCOkyDvq7i=** là chuỗi của base64 nhưng lại không decrypt được. Tôi đoán hàm phía trên là hàm encrypt base64 đã được sửa đổi.
Đây là một dạng cơ bản của việc sửa đổi base64 đó là thây đổi thứ tự bảng chữ cái của nó.

Để giải quyết vấn đề này: code lại base64 và đổi lại bảng chữ cái.

Nhưng tôi lại rất lười và thời gian có hạn. Nếu bạn không giỏi trong khả năng tìm kiếm google(như tôi) sẽ rất mất thời gian để tìm được 1 decode base64 chuẩn theo ngôn ngữ bạn mong muốn.
Vì gặp khá nhiều bài dạng này, và họ thường dùng python để thây đổi bảng chữ cái nên tôi cũng dùng cách này.

```python
import base64
import string

my_b64 = "0123456789+/abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
std_b64 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

def decode(str):
    str = str.translate(string.maketrans(my_b64, std_b64))
    return base64.b64decode(str)
	
print decode("olCOkyDvq7i=")
#output = iz4ZJapu
```

#### Block 2

Để vào được block 2 & 3 thì bạn phải vượt qua 1 antidebug là **IsDebuggerPresent**, người ra đề muốn chúng ta phải debug thì mới vào được phần code của block 2 & 3.

```C
for ( i = 8; i < 16; ++i )
    mem_output[i] = aKhangkito[i % 8] ^ input[i];
if ( !memcmp(&harcode, &mem_output, 8u) )
```

Đoạn code này khá đơn giản, là sử dụng 8 byte tiếp theo của input và xor nó với chuỗi **khangkito** và sau đó là so sánh với 1 chuỗi hardcode.
Easy quá nhỉ, chỉ việc xor ngược lại thì tôi đã có thể tìm được 8 byte tiếp theo của flag rồi. yeh yeh

```python
harcode = [0x1,0x5c,0x27,0x2d,0xc,0x59,0xd,0x32]
key = "khangkito"
print "".join(chr(ord(key[i])^harcode[i])for i in range(len(harcode)))
#output = j4FCk2dF
```

#### Block 3

Cũng tương tự như block 2, 8 byte input cuối cùng sẽ được xor với chuỗi "12121212", chuỗi này được tạo là giờ của máy tính.

```python
harcode = [0x4b,0x1,0x44,0x61,0x46,0x42,0x68,0x6a]
key = "12121212"
print "".join(chr(ord(key[i])^harcode[i])for i in range(len(harcode)))
#output = z3uSwpYX
```

Sau cùng, tôi có chuỗi cần tìm là **iz4ZJapuj4FCk2dFz3uSwpYX**.

Nhập vào và tôi nhận được flag: NightSt0rm{Y0u_w4nt_t0_c01N_0r_Pr1z}




