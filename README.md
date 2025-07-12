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