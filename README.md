# DatabaseSecurity\_0052



\# DatabaseSecurity\_0052

\# Praktikum Keamanan Database — SQL Injection



\## Identitas

\- \*\*Mata Kuliah\*\*: Keamanan Siber

\- \*\*Repository\*\*: DatabaseSecurity\_0052



\---



\## Deskripsi



Praktikum ini membahas celah keamanan \*\*SQL Injection\*\* pada aplikasi web berbasis Flask (`web01.py`). Terdapat dua jenis serangan yang diujikan:

1\. \*\*Tautology Attack\*\* — menyisipkan kondisi yang selalu benar untuk membypass login

2\. \*\*Piggyback Attack\*\* — menumpangkan perintah SQL berbahaya untuk memanipulasi database



\---



\## File dalam Repository



| File | Keterangan |

|---|---|

| `web01\_original.py` | Kode asli yang \*\*rentan\*\* terhadap SQL Injection |

| `web01.py` | Kode yang sudah \*\*diperbaiki\*\* dan aman dari SQL Injection |

| `README.md` | Dokumentasi praktikum |



\---



\## 1. Tautology Attack



\### Penjelasan

Tautology attack menyisipkan kondisi yang \*\*selalu bernilai TRUE\*\* ke dalam query SQL sehingga attacker bisa login tanpa mengetahui password yang benar.



\### Kode Rentan (web01\_original.py)

```python

cur.execute(

&#x20;   "SELECT id, username FROM `user` WHERE username='%s' AND password='%s'"

&#x20;   % (username, password)

)

```



\### Payload yang Digunakan

\- \*\*Username\*\*: `user1' OR '1'='1`

\- \*\*Password\*\*: `123` (bebas)



\### Query yang Terbentuk

```sql

SELECT id, username FROM user 

WHERE username='user1' OR '1'='1' AND password='123'

```

Kondisi `'1'='1'` selalu TRUE sehingga WHERE clause selalu terpenuhi.



\### Hasil — SERANGAN BERHASIL (web01\_original.py)

!\[Tautology Berhasil](Screenshot\_2026-05-02\_191858.png)



\*\*Bukti di terminal\*\*: POST /login HTTP/1.1" 302 → redirect ke home (login berhasil)



\---



\## 2. Piggyback Attack



\### Penjelasan

Piggyback attack menumpangkan perintah SQL tambahan menggunakan tanda titik koma (`;`) untuk mengeksekusi perintah berbahaya seperti DELETE, DROP, atau INSERT.



\### Kode Rentan (web01\_original.py)

```python

cur.executescript(

&#x20;   "INSERT INTO `time\_line` VALUES (NULL, %d, '%s')" % (uid, content)

)

```



\### Payload yang Digunakan



`data'); DELETE FROM time\_line WHERE (content='World`



\### Query yang Terbentuk

```sql

INSERT INTO time\_line VALUES (NULL, 1, 'data');

DELETE FROM time\_line WHERE (content='World')

```



\### Hasil — SERANGAN BERHASIL (web01\_original.py)

!\[Piggyback Input](Screenshot\_2026-05-02\_192401.png)

!\[Piggyback Hasil](Screenshot\_2026-05-02\_193122.png)



\*\*Bukti\*\*: Sebelum serangan ada "World" di list, setelah submit payload "World" terhapus dari database.



\---



\## 3. Pencegahan SQL Injection



\### Solusi: Parameterized Query



\### Perbaikan Login — Mencegah Tautology

```python

\# SEBELUM (rentan):

cur.execute(

&#x20;   "SELECT id, username FROM `user` WHERE username='%s' AND password='%s'"

&#x20;   % (username, password)

)



\# SESUDAH (aman):

cur.execute(

&#x20;   'SELECT id, username FROM `user` WHERE username=? AND password=?',

&#x20;   (username, password)

)

```



\### Perbaikan Create Timeline — Mencegah Piggyback

```python

\# SEBELUM (rentan):

cur.executescript(

&#x20;   "INSERT INTO `time\_line` VALUES (NULL, %d, '%s')" % (uid, content)

)



\# SESUDAH (aman):

cur.execute(

&#x20;   'INSERT INTO `time\_line` VALUES (NULL, ?, ?)',

&#x20;   (int(uid), content)

)

```



\### Hasil — SERANGAN DITOLAK (web01.py)

!\[Tautology Ditolak](code2-5.png)



\*\*Bukti di terminal\*\*: POST /login HTTP/1.1" 200 → tetap di halaman login (ditolak)



\---



\## Ringkasan Perbandingan



| Serangan | File | Hasil |

|---|---|---|

| Tautology `user1' OR '1'='1` | `web01\_original.py` | Berhasil login tanpa password benar |

| Piggyback `data'); DELETE FROM...` | `web01\_original.py` | Data "World" terhapus dari database |

| Tautology `user1' OR '1'='1` | `web01.py` (aman) | Ditolak — "Username/password salah" |



\---



\## Kesimpulan



SQL Injection dapat dicegah dengan:

1\. \*\*Parameterized Query\*\* — gunakan `?` sebagai placeholder, bukan `%s`

2\. \*\*Gunakan `execute()` bukan `executescript()`\*\* — mencegah multiple statement

3\. \*\*Validasi tipe data\*\* — pastikan input integer dengan `int()`

4\. \*\*Secret key yang aman\*\* — gunakan `secrets.token\_hex(32)`

