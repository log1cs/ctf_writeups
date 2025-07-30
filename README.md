# BlitzCTF 2025

?

# Misc
## Broken encoding or what? (50 pts/127 solves)
### Description
Our systems intercepted a suspicious string during a data breach, our "men" are "working" on it. but the encoding seems… wrong. It's not Base64, not hex, not rot13. Nothing standard.

We suspect it was "typed" under unusual conditions. Can you decipher it and retrieve the message?

Flag format: Blitz{some_text}

### Solve
```
Given string: Voubz[mabmy_lr_ut_jpf_mak_qdrwbj_euhs]
```

Sử dụng dCode để analyze loại các loại Cipher "likely" happening in this challenge - ta có thể thấy được các rate như sau: 

![dcode](https://github.com/log1cs/writeups/raw/blitz-ctf-2025/images/dcode_suggestions.png)

Ở challenge này, nếu người giải chỉ chăm chăm để ý vào những cipher có khả năng dính vào cao nhất - thì xin chúc mừng, đó là 1 bước đi vào lòng đất rồi đấy :D

Đây là một dạng Keyboard Change Cipher, từ dạng `workman ↦ qwerty`.

### Flag
```
Blitz{catch_me_if_you_can_qwerty_kids}
```

## BlitzBot (323 pts/45 solves)
### Description
There are 3 types of BlitzBot: Blue, Yellow, and Red Every day they make other BlitzBot to populate BlitzWorld Each Blue BlitzBot make 2 Yellow BlitzBot and 3 Red BlitzBot Each Yellow BlitzBot make 2 Red BlitzBot and 3 Blue BlitzBot Each Red BlitzBot make 2 Blue BlitzBot and 3 Yellow BlitzBot This proccess continue every day How many Blue, Yellow, and Red BlitzBot after N-th day each Modulo by 1e9 + 7

Input Format: B0 Y0 R0 N Where B0 is Blue BlitzBot, Y0 is Yellow BlitzBot, and R0 is Red BlitzBot on the first day (day 0)

Output Format: BN YN RN Where BN is Blue BlitzBot, YN is Yellow BlitzBot, and RN is Red BlitzBot on the N-th day (day N)

Time Limit: 3 Seconds for all test cases

```
nc pwn.blitzhack.xyz 1234
```

### Solve
Bài này có 1 quy tắc nhân như sau:
* 1 xanh thì bằng 2 vàng và 1 đỏ
* 1 vàng thì bằng 2 đỏ và 3 xanh
* 1 đỏ thì bằng 2 xanh và 3 vàng

Ví dụ đơn giản:
```
Question: 1 2 3 1 (B0 Y0 R0 0)
Answer: 13 13 10 (B0 Y0 R0) - Note: Quá 3s sẽ tính timeout.
```

Với dữ liệu quan trọng là số lượng bot sản sinh ra không được biến mất mà phải được tiếp tục cộng vào sau mỗi ngày.

Vậy nên công việc của chúng ta ở mỗi ngày sẽ là:
* Giữ lại toàn bộ số bot đang có
* Tính số bot mới được tạo ra
* Cộng lại với số bot đang có để ra số robot ngày tiếp theo

Chúng ta có công thức sau:

![blitz1](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/images/blitzbot_1.png)

Với B0/Y0/R0 = 1/2/3, và ngày 0:

![blitz2](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/images/blitzbot_2.png)

Và chúng ta cứ tính với công thức đó, và cộng với số lượng bot sản sinh ra được ở hôm trước đó là ra được kết quả đúng.

Từ những dữ kiện trên, chúng ta có thể craft 1 script chạy như dưới để bảo đảm việc tính toán nhanh và đáp ứng được nhu cầu.
```
import socket

MOD = 10**9 + 7

def mat_mult(A, B):
    res = [[0]*3 for _ in range(3)]
    for i in range(3):
        for j in range(3):
            for k in range(3):
                res[i][j] = (res[i][j] + A[i][k]*B[k][j]) % MOD
    return res

def mat_pow(matrix, power):
    result = [[1 if i == j else 0 for j in range(3)] for i in range(3)]
    while power:
        if power % 2:
            result = mat_mult(result, matrix)
        matrix = mat_mult(matrix, matrix)
        power //= 2
    return result

def blitzbot(B0, Y0, R0, N):
    M = [
        [1, 3, 2],
        [2, 1, 3],
        [3, 2, 1]
    ]
    M_pow = mat_pow(M, N)
    BN = (M_pow[0][0]*B0 + M_pow[0][1]*Y0 + M_pow[0][2]*R0) % MOD
    YN = (M_pow[1][0]*B0 + M_pow[1][1]*Y0 + M_pow[1][2]*R0) % MOD
    RN = (M_pow[2][0]*B0 + M_pow[2][1]*Y0 + M_pow[2][2]*R0) % MOD
    return BN, YN, RN

host = 'pwn.blitzhack.xyz'
port = 1234

with socket.create_connection((host, port)) as sock:
    sock_file = sock.makefile('rw', buffering=1, encoding='utf-8')
    
    try:
        while True:
            line = sock_file.readline()
            if not line:
                break
            print("Received:", line.strip())

            parts = line.strip().split()
            if len(parts) != 4:
                continue

            try:
                B0, Y0, R0, N = map(int, parts)
                BN, YN, RN = blitzbot(B0, Y0, R0, N)
                result = f"{BN} {YN} {RN}"
                print("Sending:", result)
                sock_file.write(result + '\n')
            except Exception as e:
                print("Parse or calc error:", e)
                continue
    except KeyboardInterrupt:
        print("Exited.")
```

### Flag
```
Blitz{SUD0_M4K3_B07***}
```

## Chamber of Secrets (423 pts/30 solves)
### Description
Deep within the ancient, enchanted halls of Hogwarts, a long-forgotten secret lies hidden in layers of chambers, waiting for a daring soul to uncover it. The castle’s magic has concealed its deepest mysteries, guarded by cryptic puzzles, codes, and ancient traps—much like the legendary Chamber of Secrets. Only those with the sharpest minds and the courage of a true wizard can decipher the clues and break through the layers of magic that protect the truth. As you venture deeper into the hidden chambers, the darkness grows, and the mysteries become more complex. Will you crack the codes and reveal the secret before the magic consumes you?

The challenge is set — Dare to look closer?

### Attachment
Tải attachment cho Chamber of Secrets tại [đây](https://github.com/log1cs/writeups/tree/blitz-ctf-2025/attachments/chamber_of_secrets)

### Solve
Đầu tiên, analyze file bằng `binwalk`:

![binwalk](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/images/binwalk.png)

Trong file attachment được cho sau khi analyze bằng binwalk - có 1 file JPG và 1 file zip.

Cụ thể với file zip, chúng ta được yêu cầu cấp mật khẩu để giải nén file. Tuy nhiên vì đây là 1 misc challenge, cho nên việc "tồn tại" những yếu tố quan trọng giúp chúng ta có thể tìm được mật khẩu gần như là không có - cụ thể là trong bài này, hexview/strings file `text` hoặc file `zip` không đem lại nhiều kết quả khả thi cho lắm :D

Cách giải quyết đơn giản nhất trong bài này chính là... đọc truyện :trollface:

![chamber](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/attachments/chamber_of_secrets/analyzed/chamber.jpg)

Sử dụng hết các từ để tạo thành một wordlist và bruteforce các từ đó để giải nén file zip. May mắn thay khi nó chạm tới từ `exchanged`, thì file đã được giải nén thành công. Trong file có 1 đường link như sau, và chắc chắn đường link này không hề valid :trollface:

```
https://lastchamberofsecrect.com/url-decode/base?galf=QmxpdHp7aDFkZDNuXzFuXzdoM19kMzNwX3hEfQ%3D%3D
```

### Flag
Decode bằng base64 với `QmxpdHp7aDFkZDNuXzFuXzdoM19kMzNwX3hEfQ`:

```
Blitz{h1dd3n_1n_7h3_d33p_xD}
```

# IOT
# IOT Sucks (493 pts/10 solves)
### Description
I regret learning about IOT - should have studied about some communication methods instead..... Wait a minute why don't I do both manually!? 

### Attachment
Tải attachment cho IoT sucks tại [đây](https://github.com/log1cs/writeups/tree/blitz-ctf-2025/attachments/chamber_of_secrets).

### Solve
Ban đầu, tất cả những gì chúng ta có chỉ là 1 file video. Trong đó có chiếc Arduino Uno, và cái LED đỏ blinking trong suốt cả cái video đó.

Điều này có thể khiến chúng ta liên tưởng đến morse code.

Sau đó, cuối cùng [source code](https://raw.githubusercontent.com/log1cs/writeups/refs/heads/blitz-ctf-2025/attachments/iot_sucks/src.ino) cũng được release, giúp cho việc giải thuận tiện hơn rất nhiều (vì làm gì có ai biết morse blinking sequence như thế nào đâu :trollface:)

Chúng ta sẽ tìm hiểu rõ hơn về source code. Để ý kĩ hàm `void loop()` với các case kí tự. Ví dụ với kí tự `a`, thì các chu kì LED sẽ hiển thị như sau:

```
    case 'a':				//.-
     
    digitalWrite(x,HIGH);  // digitalWrite(amount, HIGH) = bật đèn LED lên
    delay(500);            // delay 500 ms (LED vẫn mở)
    digitalWrite(x,LOW);   // digitalWrite(amount, LOW) = Tắt đền LED đi
    delay(400);            // Đèn LED được tắt trong 400 ms
```

Tương tự với các hàm còn lại. Sau cùng thì challenge này cũng chỉ loanh quanh LED blinking và đối chiếu kí tự.

Bài này sẽ có 2 hướng giải, một là kiểu chày cối (ngồi nhìn video và đối chiếu :trollface:) và 2 là kiểu lấy plot chu kì và đối chiếu kí tự. 

Intended solve sẽ là cách 2, nhưng ở đây mình bày ra 2 cách luôn cho những ai thích chày cối level max :trollface:

Mở video lên, đối chiếu kí tự và làm 1 cái bảng blinking sequences:

Ở đây mình cứ mặc định `delay(500)` + `delay(400)` là nhanh - và `delay(700)` + `delay(900)` là chậm.

| Ký tự  | Sequence                         | Tổng số LED đã blink | Timestamps     |
| ------ | -------------------------------- | -------------------- | -------------- |
| **m**  | 2 chậm                           | 2                    | 00:07–00:09    |
| **0**  | 5 chậm                           | 7                    | 00:10–00:18    |
| **r**  | 3 nhanh, chậm, nhanh             | 10                   | 00:19–00:21    |
| **s**  | 3 nhanh                          | 13                   | 00:22–00:25    |
| **3**  | 3 nhanh, 2 chậm                  | 18                   | 00:26–00:32    |
| **i**  | 2 nhanh                          | 20                   | 00:33–00:35    |
| **0**  | 5 chậm                           | 25                   | 00:36–00:43.50 |
| **t**  | 1 nhanh                          | 26                   | 00:44–00:45    |
| **\_** | 2 nhanh, 2 chậm, 1 nhanh, 1 chậm | 32                   | 00:46–00:53    |
| **1**  | 1 nhanh, 1 chậm                  | 34                   | 00:54–00:56    |
| **5**  | 5 nhanh + delay 500ms            | 39                   | 00:57–01:04    |
| **\_** | 2 nhanh, 2 chậm, 1 nhanh, 1 chậm | 45                   | 01:05–01:09    |
| **b**  | 1 chậm, 3 nhanh                  | 49                   | 01:10–N/A      |
| **4**  | 4 nhanh, 1 chậm                  | 54                   | N/A            |
| **d**  | 1 chậm, 2 nhanh                  | 57                   | N/A            |

Thực ra ban đầu flag của mình là `Blitz{m0rs3i0t_4r3_b4d}` nhưng trong Discord họ đã thông báo trên announcement là nếu ai ra được chữ `a` trong flag thì nó thực ra là `1`, số còn lại thì không ảnh hưởng.

![announcement](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/images/announcement.png)

Ghép lại các thông tin trên thì mọi người sẽ có [flag](https://github.com/log1cs/writeups/tree/blitz-ctf-2025?tab=readme-ov-file#flag-3).

Đương nhiên thì cách vừa rồi chỉ là troll thôi, nhưng thực sự thì mình đã làm thế trong suốt quá trình giải challenge này :trollface:. Tuy nhiên - như đã nói, nếu có plot thì mọi thứ sẽ trở nên dễ dàng hơn rất nhiều.

Nên mình đã dùng script này để generate plot ra:

```
# prompt: Plot "items" variable, it's in the format of [timestamp, value]. Make the graph fit screen width. Mark the timestamp at very 0.1 seconds

import matplotlib.pyplot as plt

# Extract timestamps and values from the items list
timestamps = [item[0] for item in items]
values = [item[1] for item in items]

# Create the plot
plt.figure(figsize=(50, 5))  # Adjust the figure size as needed
plt.plot(timestamps, values)

# Set x-axis limits to start from the beginning of the data
plt.xlim(timestamps[0], timestamps[-1])

# Set the x-axis tick locations at 0.1-second intervals
# Find the minimum and maximum timestamps to determine the range
min_ts = timestamps[0]
max_ts = timestamps[-1]

# Generate tick locations from min_ts to max_ts with a step of 0.1
tick_locations = [i * 0.1 for i in range(int(min_ts * 10), int(max_ts * 10) + 1)]

plt.xticks(tick_locations)

# Add labels and title
plt.xlabel('Timestamp (seconds)')
plt.ylabel('Value')
plt.title('Value over Time')

# Ensure the plot is displayed within the notebook
plt.grid(True)
plt.tight_layout()
plt.show()
```

Sử dụng nó với đoạn video, ta có được plot như sau:

![plot](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/images/plot.jpg)

Sử dụng code để đối chiếu, ta có thể dễ dàng nhận ra dựa vào các blink sequence của đèn LED:

![plot_modified](https://raw.githubusercontent.com/log1cs/writeups/blitz-ctf-2025/images/plot_modified.png)

Giờ thì flag đã rõ rành rành rồi. Enjoy!

### Flag

```
Blitz{m0rs3i0t_15_b4d}
```