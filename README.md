# RSA на Python3

## Входные данные

Для шифрования необходимы следующие значения:  

`p` — первое простое число \
`q` — второе простое число \
`n` — модуль \
`e` — отрытая экспонента \
`d` — секретная экcпонента

#### На выходе получим:
`e` - открытый ключ \
`d` - закрытый ключ

### Генерация простых чисел (p, q):

1. Генерируем не четное число длиной от 511 до 512 бит.
2. Проводим тест Миллера — Рабина. Он определит: число составное или <u>возможно</u> простое. Алгоритм взял с wiki.
3. Если тест Миллера — Рабина показывает, что число не простое, тогда прибавляем к нему 2 (только не четные числа).
4. Если нашли простое число - возвращаем его.

Идеально иметь разницу между p и q не меньше 8к бит.

### Выбор экспонент (e, d)

`е` = 65537. Т.к. в двоичной системе счисления она имеет вид: `100...(15)...001` 
состоящее из 17 байт. Является взимно простым с `fi`.

`d` - это число обратное `e`. По формуле: `e = d mod(fi)`. 
Это число - наибольший общий делитель для `e` и `fi`.

#### Как найти обратное число?

Передаем `e` и `fi` расширенному алгоритму Евклида.

Алгоритм Евклида создает 2 массива.
```python
u = (e,  1, 0)
v = (fi, 0, 1)
```
Пока fi не станет меньше 0, проделываем операции:
```python
q = u[0] // v[0]
t = (u[0] % v[0], u[1] - q * v[1], u[2] - q * v[2])
u = v
v = t
```
Переменные будут перезаписывать в себя цифры. И когда fi станет меньше 0, тогда мы возвращаем `e`.
Из этого `e` вычисляем `e mod(fi)` и получаем результат. НОД (наибольший общий делитель).

В python НОД можно найти с помощью функции `gcd(a,b)`.

## Шифрование

```text
c=m^e (mod n)
```

Зашифровать может любой, а расшифровать нет. 
`e` - общедоступно. Но для рашифровки понадобится `d`.

`d` - получается по формуле: `e = d mod(fi)`. \
`fi` - результат перемножения `p` и `q`. Чтобы вычислит эти 2 числа,
необходимо `fi` - факторизовать. `d = invert(e, fi)`
```bash 
factor 123456789
```
Если удастся проделать эту операцию, то злоумышленник сможет расшифровать сообщение. 

## Расшифрование

```text
m=c^d (mod n)
```

## Про стандарт PKCS#1

Сигнатура зашифрованного файла имеет вид:

`EM = 0x00 || 0x02 || PS || 0x00 || M.`

здесь `PS` — это псевдослучайная последовательность, состоящая из ненулевых октетов,\
`M` — кодируемый текст в виде байтовой последовательности,\
символ `’||’` обозначает операцию конкатенации (объединение последовательностей).

Это необходимо, чтобы обезопасить себя от подбора `M`. Если злоумышленник вычислит перебором `M`,
тогда сможет подобрать ключ и расшифровать все остальное.

## Различные хитрости при работе с байтами и числами

Преобразование текстовой строки в последовательность байтов:
```python
b = bytes("abcабв", encoding="utf-8")
```
Обратное преобразование последовательности байтов в текстовую строку:
```python
s = str(b, encoding="utf-8")
```
Преобразование байтовой последовательности в целое число:
```python
k = int.from_bytes(b, byteorder="big")
```
Преобразование числа в байтовую последовательность:
```python
b = k.to_bytes(num_size, byteorder="big")
```
где `num_size` — размер получаемого массива в байтах.


# Автоматическая генерация ключа

Генерация 1024-битного ключа в формате PEM средствами OpenSSL.

```bash
openssl genrsa -out mykey.pem 1024
```

Распечатка секретного ключа в текстовом представлении.
```bash
openssl rsa -in mykey.pem -noout -text
```

Чтение ключа в python с помощью модуля PyCrypto (модуль pycryptodome или pycryptodomex):

```python
from Crypto.PublicKey import RSA
with open("mykey.pem", "r") as f:
    key = RSA.importKey(f.read())
```

Вызываем теперь с помощью:

```python
key.n — модуль
key.p — первое простое число
key.q — второе простое число
key.e — отрытая экспонента
key.d — секретная экcпонента
```

Если возникают проблемы с переводом строки в зависимости от семейства ОС:

```python
from Crypto.PublicKey import RSA
with open("mykey.pem","r") as f:
    key = RSA.importKey("\n".join(
            [x.strip() for x in f.read().split("\n")]))
```

После зашифрования с помощью функции `encr_with_openssl()` результат можно проверить на другом компьютере.
Взяв зашифрованный файл `message.pem` можно проверить командой

```bash
hexdump -C /home/student/3агрузки/message.pem
```

Используя наш закрытый ключ `teacher.pem` - расшифровываем командой:

```bash
openssl rsautl -in /home/student/3агрузки/message.pem -inkey /run/media/student/BORMICRO/teacher.pem -decrypt
# или
openssl rsautl -in /home/student/3агрузки/message.pem -inkey /run/media/student/BORMICRO/teacher.pem -decrypt -raw -hexdump
```

