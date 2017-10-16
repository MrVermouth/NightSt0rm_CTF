# Level_1

- Bin: là file PE 32bit. Chỉ có duy nhất 1 file cho 3 flag

Mở binary lên, tôi thấy người ra đề rất có đam mê với Bitcoin. Vì ở các cuộc thi bài đầu thường rất dễ, tôi bấm hết các nút ở trên chương trình và thấy điều bất ngờ.

<img src="https://i.imgur.com/IXePPvJ.png">

Ở phần **About Us**, là phần mô tả của nhóm **NightSt0rm** và có một đoạn code **C**. 
Yeh, đây rồi đoạn xử lý của flag level_1. Khá dễ dàng để code lại vì đây là thuật toán **XOR** đơn giản.
Nhưng đôi khi dễ đến thế nhưng vẫn submit không được, tôi đã nhầm lẫn ở về dấu cách trong chuỗi so sánh.


```python
key = "S2^g1s^c2f0o2s  "
print "".join(chr(ord(i)^1) for i in key)
```

Yeh, Flag: Nightst0rm{R3_f0r_b3g1n3r!!}
