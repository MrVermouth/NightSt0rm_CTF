# Level_3

- Bin: là file PE 32bit. Chỉ có duy nhất 1 file cho 3 flag

Phew, xong 2 level mà đã hơn 1h30p trôi qua, chỉ cò lại hơn 3h để giải nốt bài còn lại và khám phá các challenge ở mảng khác nữa.
Tôi tiếp tục follow theo string đã tìm được ở level_2, đến đoạn lấy dữ liệu và kiểm tra của nó.

Đoạn code đầu tiên rất thú vị
```C
v4 = GetDlgItem(hDlg, 1006);
GetWindowTextA(v4, String, 10);
your_coin = string_to_hex(String);
v6 = GetDlgItem(hDlg, 1007);
GetWindowTextA(v6, String, 10);
if ( my_coin + your_coin) >= 500 )
```

Sau khi dùng **Resource Hacker** để kiểm tra các hằng số cho hàm **GetDlgItem** tôi thấy nó tương ứng với giá trị của your_coin và my_coin của chương trình.
(Thanh niên ra đề ám ảnh Coin quá rồi :)))

Ở level_2 khi tôi nhập đúng flag, your_coin đã được cộng thêm 200(0->200), và ô input của level bị disable. Tôi đoán ý đồ của tác giả
là muốn nhập đúng level_2 rồi làm level_3 nhưng phải sửa lại tham số của ô input level_3. Khá rắc rối vì tôi rất lười và cứ phải nhập lại
key của level_2 cho mỗi lần debug, nên tôi quyết định path luôn giá trị tại your_coin ( ahihi ).

Sau khi đã vượt qua được bước đó, tôi phân tích chương trình cũng là dạng bài **XOR**(cảm thấy choáng váng XOR, XOẠC oh noooooo).

Input có độ dài 24 và tiếp tục chia là các block với len như sau : 2,4,4,4,4,6 để tính toán.

#### Block 1

```C
if ( !memcmp(&input, "h4", 2u) )
```
2 ký tự đầu tiên dễ nhìn thấy là **h4**.

### Block 2 & 3 & 4 & 5

```C
md5(&input[2], md5_output, 4u);
string_to_hex_(hex_input, "%x%x%x%x", input[2], input[3], input[4], input[5]);
for ( i = 0; i < 16; ++i )
  mem[i] = hex_input[i % 8] ^ md5_output[i];
if ( !memcmp(&hardcode1, mem, 0x10u) )
{
  md5(&input[6], md5_output, 4u);
  string_to_hex(hex_input, "%x%x%x%x", input[6], input[7], input[8], input[9]);
  for ( j = 0; j < 16; ++j )
    mem[j] = hex_input[j % 8] ^ md5_output[j];
  if ( !memcmp(&hardcode2, mem, 0x10u) )
  {
    string_to_hex(hex_input, "%x%x%x%x", input[10], input[11], input[12], input[13]);
    md5(&input[10], md5_output, 4u);
    for ( k = 0; k < 16; ++k )
      mem[k] = hex_input[k % 8] ^ md5_output[k];
    if ( !memcmp(&hardcode3, mem, 0x10u) )
    {
      string_to_hex(hex_input, "%x%x%x%x", input[14], input[15], input[16], input[17]);
      md5(&input[14], md5_output, 4u);
      for ( l = 0; l < 16; ++l )
        mem[l] = hex_input[l % 8] ^ md5_output[l];
      if ( !memcmp(&hardcode4, mem, 0x10u) )
```

Thuật toán là **md5(4byte input) xor hex(4byte input)**. Sau khi hiểu được thuật toán không còn cách nào khác ngoài bruteforce.
Vì kí tự in được nên việc bruteforce 4 kí tự cũng tương đối nhẹ nhàng. Tôi code python vì nó nhanh trong việc xử lý chuỗi nhưng lại chậm trong quá trình tính toán.
Tôi mất hơn 2h để brute được 16 kí tự của input.

```python
import hashlib
import string
def md5(string):
    m = hashlib.md5()
    m.update(string)
    return m.hexdigest()
count = 0
check = "7fdb7d6c0b45fd2da58b00d14004bedb".decode("hex")
check1 = "3d5d78e7d6588fd3f717f00c7ed4c2fa".decode("hex")
check2 = "c42b9254b2040f71990bee42fe3940ca".decode("hex")
check3 = "658fa7627f14ce467e4f452e5b8edd9d".decode("hex")
for i1 in string.printable:
    for i2 in string.printable:
        for i3 in string.printable:
            for i4 in string.printable:
                xor = ""
                key = ""
                key = i1+i2+i3+i4
                key1 = key.encode("hex")
                md5_string = md5(key).decode("hex")
                for j in range(len(md5_string)):
                    xor += chr((ord(md5_string[j])^ord(key1[j%8]))&0xff)
                if xor == check:
                    print "check"
                    print key
                elif xor == check1:
                    print "check1"
                    print key
                elif xor == check2:
                    print "check2"
                    print key
                elif xor == check3:
                    print "check3"
                    print key
print "DONE"
#V6tx
#cQnF
#AoQU
#UdiG
```

#### Block 6

Block này là 1 hệ 6 phương trình 6 ẩn. Khá dễ dàng cho việc giải quyết phương trình này.

```C
if ( input[18] + input[19] + input[20] + input[21] + input[22] + input[23] == 511
&& (input[18] ^ input[19]) == 38
&& input[20] + input[21] + input[22] == 243
&& input[20] - input[21] == -37
&& input[21] - input[23] == 32
&& input[22] + input[19] == 192 )
```
Có rất nhiều cách để giả phương trình này. Nhưng bruteforce là 1 ý tưởng tồi vì 6 kí tự rất lớn.
Ở đây tôi dùng z3(module thần thánh do Microsoft phát triển) chỉ xử lý trong 1 nốt nhạc.

```python
from z3 import *

s = Solver()

v18 = BitVec("v18",8)
v19 = BitVec("v19",8)
v20 = BitVec("v20",8)
v21 = BitVec("v21",8)
v22 = BitVec("v22",8)
v23 = BitVec("v23",8)
s.add(v18+v19+v20+v21+v22+v23 == 511)
s.add(v18 ^ v19 == 38)
s.add(v20 + v21 + v22 == 243)
s.add(v20 - v21 == 0xFFFFFFDB)
s.add(v21 - v23 == 32)
s.add(v22 + v19 == 192)
s.add(v18>0x20,v18<0x7e)
s.add(v19>0x20,v19<0x7e)
s.add(v20>0x20,v20<0x7e)
s.add(v21>0x20,v21<0x7e)
s.add(v22>0x20,v22<0x7e)
s.add(v23>0x20,v23<0x7e)

while s.check() == sat:
    a1 = "%s"%(s.model()[v18])
    a2 = "%s" % (s.model()[v19])
    a3 = "%s" % (s.model()[v20])
    a4 = "%s" % (s.model()[v21])
    a5 = "%s" % (s.model()[v22])
    a6 = "%s" % (s.model()[v23])
    print chr(int(a1))+chr(int(a2))+chr(int(a3))+chr(int(a4))+chr(int(a5))+chr(int(a6))
    s.add(Or(v18 != s.model()[v18],v19 != s.model()[v19], v20 != s.model()[v20], v21 != s.model()[v21], v22 != s.model()[v22], v23 != s.model()[v23]))
#RtAfLF
```

Yehhh!!!!!! Tìm được đủ 24 kí tự rồi nhưng flag ra không có nghĩa(có gì đó không đúng) tôi đã bỏ qua phần nào.

Sau khi check lại code, có 1 đoạn xor phía trước block 6 với mục đích là xor **filename** với chuỗi đã được xor qua mấy lượt ở trên. Lại còn cả antidebug nữa !!!(khoai quá)

```C
for ( i = 0; i < 18; ++i )
	out[i]= (antidebug + xor[i]) ^ filename[i % len(filename)]);
```

> antidebug: là dạng IsDebuggerPresent

```C
mov     eax, large fs:30h
movzx   eax, byte ptr [eax+2]
```

Nhưng tôi dùng trên file của ban tổ chức, vượt qua antidebug(path lúc debug or chạy trước tiếp chương trình không debug) nhưng vẫn không ra flag đẹp.
Năm trước tôi có chơi **flareon** tuy được vài level đầu nhưng có 1 bài khá thú vị và giống với bài level_3 này. Áp dụng từ bài của flareon,
tôi load lại binary vào IDA vào để ý lúc nó cho gợi ý, yehhh tên file đây rồi.

<img src="https://i.imgur.com/Uad8GiH.png">

Đổi tên file, chạy chương trình trực tiếp không debug tôi nhận được Flag: **NightSt0rm{Y0u_h4v3_d0na_It_R1ght!!}**




