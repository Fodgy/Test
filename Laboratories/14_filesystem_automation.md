# Тема: Filesystem Automation

**Всього задач:** 55 (Special Task-и не входять у лічильник)
**Розподіл:** Basic (10) + Elementary (9) = 19 (35%) · Intermediate (20) + Advanced (8) = 28 (50%) · Challenge (8) = 8 (15%)

> Розв'язань немає. Якщо потрібна допомога, попроси Hint 1/2/3 або Solution для конкретного Task X.

## Правила лабораторії

- **Пісочниця:** кожна задача працює лише в тимчасових директоріях (`tempfile`). Жодних операцій над домашніми чи системними папками.
- **Dry-run:** де є видалення, переміщення чи перезапис, потрібен режим `dry_run` / `--dry-run`.
- **Без `shell=True`.** Офлайн. Без сторонніх залежностей (крім `pytest` там, де сказано про тести).
- **Кросплатформеність:** задачі, що залежать від POSIX (симлінки, `chmod`, `flock`, сигнали), мають позначку **[POSIX]**. Запускай їх на Linux/macOS/WSL.
- **Сходинки підказок у Starter Code:** Basic дає пропуски в критичних місцях, Elementary дає каркас або сигнатуру. Починаючи з Intermediate, інструмент обираєш сам.
- **Нумерація наскрізна** (Task 1–55). Progress Check після кожних 10 задач, Special Task після кожного Progress Check (крім фінального).

## Зміст

| Рівень | Задачі |
|---|---|
| LEVEL 1 — Basic | 1–10 |
| LEVEL 2 — Elementary | 11–19 |
| LEVEL 3 — Intermediate | 20–39 |
| LEVEL 4 — Advanced | 40–47 |
| LEVEL 5 — Challenge | 48–55 |

---

# LEVEL 1 — BASIC (Task 1–10)

### Task 1 — Пошук усіх .txt у дереві
**Складність:** Basic

**Сценарій:** У папці проєкту нотатки розкидані по вкладених директоріях. Потрібен список усіх `.txt`, щоб передати його в скрипт обробки.

**Умова:** Поверни відсортований список усіх `.txt` файлів у папці та всіх підпапках.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="lab01_"))
(root / "a" / "b").mkdir(parents=True)
(root / "old.txt").mkdir()                       # директорія з «файловою» назвою
(root / "one.txt").write_text("1")
(root / "a" / "two.txt").write_text("2")
(root / "a" / "b" / "three.txt").write_text("3")
(root / "a" / "pic.jpg").write_bytes(b"\xff")
```

**Вхідні дані / Інтерфейс:** `find_txt(root: Path) -> list[Path]`

**Очікуваний результат:** відсортований список `Path` лише зі звичайних файлів `.txt`.

**Приклад:**
Input: `find_txt(root)`
Output (відносно `root`): `[a/b/three.txt, a/two.txt, one.txt]`

**Edge Cases:**
- порожня директорія дає порожній список
- директорія `old.txt` не потрапляє в результат

**Критерії приймання:**
- знайдено всі 3 файли на різній глибині
- `pic.jpg` і директорія `old.txt` відсутні в результаті
- результат відсортований

**Starter Code:**
```python
from pathlib import Path

def find_txt(root: Path) -> list[Path]:
    files = sorted(root.______("*.txt"))
    return [p for p in files if p.______()]
```

---

### Task 2 — Створення структури каталогів
**Складність:** Basic

**Сценарій:** Щомісяця скрипт готує папки для звітів. Його запускають і вручну, і за розкладом, тому повторний запуск не має падати.

**Умова:** Створи всі каталоги зі списку всередині `root`, включно з проміжними. Поверни кількість каталогів **зі списку**, яких до виклику не існувало (проміжні на кшталт `reports` чи `reports/2026` не рахуються).

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="lab02_"))
DIRS = ["reports/2026/09", "reports/2026/10", "logs"]
(root / "logs").mkdir()          # один каталог уже існує
```

**Вхідні дані / Інтерфейс:** `ensure_dirs(root: Path, dirs: list[str]) -> int`

**Очікуваний результат:** перший виклик повертає `2`, другий повертає `0`.

**Приклад:**
Input: `ensure_dirs(root, DIRS)` двічі
Output: `2`, потім `0`

**Edge Cases:**
- порожній список дає `0`
- шлях уже існує як **файл**: зрозуміла помилка

**Критерії приймання:**
- усі каталоги існують після виклику
- другий виклик не кидає виняток
- лічильник коректний в обох викликах

**Starter Code:**
```python
from pathlib import Path

def ensure_dirs(root: Path, dirs: list[str]) -> int:
    created = 0
    for rel in dirs:
        target = root / rel
        if not target.______():
            created += 1
        target.______(parents=______, exist_ok=______)
    return created
```

---

### Task 3 — Розмір і час зміни файлу
**Складність:** Basic

**Сценарій:** Для інвентаризації потрібен короткий звіт по файлу: ім'я, розмір, коли востаннє змінювався.

**Умова:** Поверни рядок формату `<ім'я> <N> bytes <mtime у UTC, ISO 8601>`.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab03_"))
f = d / "report.txt"
f.write_text("hello world")
os.utime(f, (1_700_000_000, 1_700_000_000))
```

**Вхідні дані / Інтерфейс:** `describe(path: Path) -> str`

**Очікуваний результат:** `report.txt 11 bytes 2023-11-14T22:13:20+00:00`

**Приклад:**
Input: `describe(f)`
Output: `report.txt 11 bytes 2023-11-14T22:13:20+00:00`

**Edge Cases:**
- файл на 0 байт: `0 bytes`
- файл не існує: `FileNotFoundError` (не приховуй його)

**Критерії приймання:**
- розмір збігається з довжиною вмісту
- час у UTC із суфіксом `+00:00`
- для відсутнього файлу піднято виняток

**Starter Code:**
```python
from datetime import datetime, timezone
from pathlib import Path

def describe(path: Path) -> str:
    st = path.______()
    mtime = datetime.fromtimestamp(st.______, tz=timezone.______).isoformat()
    return f"{path.name} {st.______} bytes {mtime}"
```

---

### Task 4 — Копіювання файлу зі збереженням метаданих
**Складність:** Basic

**Сценарій:** Резервна копія має зберігати час модифікації оригіналу, інакше інкрементальний бекап вважатиме все новим.

**Умова:** Скопіюй файл у каталог призначення (створи його, якщо немає) і поверни шлях до копії.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab04_"))
src = d / "data.bin"
src.write_bytes(b"\x00\x01\x02")
os.utime(src, (1_600_000_000, 1_600_000_000))
dst_dir = d / "backup" / "2026"      # ще не існує
```

**Вхідні дані / Інтерфейс:** `backup_file(src: Path, dst_dir: Path) -> Path`

**Очікуваний результат:** `backup/2026/data.bin` існує, вміст і `st_mtime` збігаються з оригіналом.

**Приклад:**
Input: `backup_file(src, dst_dir)`
Output: `<d>/backup/2026/data.bin`

**Edge Cases:**
- `src` не існує
- `dst_dir` уже існує

**Критерії приймання:**
- `dst_dir` створено автоматично
- `st_mtime` копії дорівнює `st_mtime` оригіналу
- оригінал не змінено
- повернений шлях указує на копію

**Starter Code:**
```python
import shutil
from pathlib import Path

def backup_file(src: Path, dst_dir: Path) -> Path:
    dst_dir.mkdir(______=True, ______=True)
    return Path(shutil.______(src, dst_dir))
```

---

### Task 5 — Перенос файлу без перезапису
**Складність:** Basic

**Сценарій:** Скрипт переносить оброблені файли в `processed/`. Якщо файл із такою назвою там уже лежить, його не можна затирати.

**Умова:** Перенеси файл у каталог. Якщо файл із таким іменем там уже є, нічого не роби й поверни `False`. Інакше перенеси й поверни `True`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab05_"))
(d / "inbox").mkdir(); (d / "processed").mkdir()
(d / "inbox" / "a.csv").write_text("new")
(d / "inbox" / "b.csv").write_text("new-b")
(d / "processed" / "b.csv").write_text("old-b")
```

**Вхідні дані / Інтерфейс:** `archive(src: Path, dst_dir: Path) -> bool`

**Очікуваний результат:** `archive(a.csv)` повертає `True`. `archive(b.csv)` повертає `False`, `processed/b.csv` лишається з вмістом `old-b`, `inbox/b.csv` теж на місці.

**Приклад:**
Input: `archive(inbox/b.csv, processed)`
Output: `False`

**Edge Cases:**
- `src` не існує
- `dst_dir` не існує

**Критерії приймання:**
- існуючий файл не перезаписано
- при `False` джерело не втрачено
- при `True` джерела в `inbox` більше немає

**Starter Code:**
```python
import shutil
from pathlib import Path

def archive(src: Path, dst_dir: Path) -> bool:
    target = dst_dir / src.______
    if target.______():
        return False
    shutil.______(src, target)
    return True
```

---

### Task 6 — SHA-256 файлу чанками
**Складність:** Basic

**Сценарій:** Перед завантаженням на сервер потрібна контрольна сума. Файли бувають великими, тому читати їх цілком не можна.

**Умова:** Поверни hex-рядок SHA-256 файлу, читаючи його блоками по 64 КіБ.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab06_"))
(d / "abc.txt").write_bytes(b"abc")
(d / "empty.bin").write_bytes(b"")
(d / "big.bin").write_bytes(b"\x00" * 5_000_000)
```

**Вхідні дані / Інтерфейс:** `sha256_of(path: Path) -> str`

**Очікуваний результат:**
- `abc.txt` дає `ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad`
- `empty.bin` дає `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`

**Приклад:**
Input: `sha256_of(d / "abc.txt")`
Output: `ba7816bf...0015ad`

**Edge Cases:**
- порожній файл
- файл, розмір якого не кратний розміру блока

**Критерії приймання:**
- обидва еталонні хеші збігаються
- файл відкрито в бінарному режимі
- `big.bin` хешується без завантаження цілком
- файл закривається (`with`)

**Starter Code:**
```python
import hashlib
from pathlib import Path

def sha256_of(path: Path) -> str:
    h = hashlib.______()
    with path.open("______") as f:
        while chunk := f.read(______):
            h.______(chunk)
    return h.______()
```

---

### Task 7 — Тимчасове робоче середовище
**Складність:** Basic

**Сценарій:** Скрипту потрібне прибирання за собою: робоча папка для проміжних файлів має зникати, навіть якщо щось пішло не так.

**Умова:** Створи тимчасовий каталог із префіксом `lab07_`, запиши туди файл `tmp.txt` зі вмістом `data`, прочитай його. Поверни кортеж `(вміст, чи_існує_каталог_після_виходу)`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path
# каталог створює сама функція
```

**Вхідні дані / Інтерфейс:** `scratch_demo() -> tuple[str, bool]`

**Очікуваний результат:** `("data", False)`

**Приклад:**
Input: `scratch_demo()`
Output: `("data", False)`

**Edge Cases:**
- усередині блоку піднято виняток: каталог усе одно видаляється

**Критерії приймання:**
- повернуто `("data", False)`
- каталог зникає й при винятку всередині блоку
- вручну `shutil.rmtree` не викликається

**Starter Code:**
```python
import tempfile
from pathlib import Path

def scratch_demo() -> tuple[str, bool]:
    with tempfile.______(prefix="lab07_") as tmp:
        work = Path(tmp)
        f = work / "tmp.txt"
        f.______("data")
        content = f.______()
    return content, work.______()
```

---

### Task 8 — Видалення порожніх файлів (з dry-run)
**Складність:** Basic

**Сценарій:** Після збою експорту в папці лежать файли на 0 байт. Їх треба прибрати, але спершу побачити, що саме буде видалено.

**Умова:** Видали в каталозі (без рекурсії) усі звичайні файли розміром 0 байт. У режимі `dry_run=True` лише виводь `would delete: <ім'я>` і нічого не чіпай. Поверни кількість знайдених (або видалених) файлів.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab08_"))
(d / "empty1.csv").write_bytes(b"")
(d / "empty2.csv").write_bytes(b"")
(d / "full.csv").write_text("x,y\n1,2\n")
(d / "emptydir").mkdir()
```

**Вхідні дані / Інтерфейс:** `remove_empty(folder: Path, dry_run: bool = False) -> int`

**Очікуваний результат:** `dry_run=True` виводить 2 рядки й повертає `2`, файли на місці. `dry_run=False` повертає `2`, `empty*.csv` зникають, `full.csv` і `emptydir` лишаються.

**Приклад:**
Input: `remove_empty(d, dry_run=True)`
Output: два рядки `would delete: emptyN.csv`, повернення `2`

**Edge Cases:**
- порожня директорія: `0`
- підкаталог не є «порожнім файлом»

**Критерії приймання:**
- dry-run нічого не змінює на диску
- `full.csv` та `emptydir` не зачеплено
- повторний запуск повертає `0`

**Starter Code:**
```python
from pathlib import Path

def remove_empty(folder: Path, dry_run: bool = False) -> int:
    count = 0
    for p in folder.______():
        if p.is_file() and p.stat().______ == 0:
            count += 1
            if dry_run:
                print(f"would delete: {p.name}")
            else:
                p.______()
    return count
```

---

### Task 9 — Упакувати папку в ZIP
**Складність:** Basic

**Сценарій:** Щоденний звіт треба відправити однією архівною одиницею зі збереженням структури підпапок.

**Умова:** Створи ZIP-архів з усіх файлів каталогу `src` (рекурсивно). Шляхи всередині архіву мають бути **відносними** до `src`. Архів лежить поза `src`. Поверни кількість файлів в архіві.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab09_"))
src = d / "report"
(src / "charts").mkdir(parents=True)
(src / "summary.txt").write_text("ok")
(src / "charts" / "q1.png").write_bytes(b"\x89PNG")
(src / "charts" / "q2.png").write_bytes(b"\x89PNG")
zip_path = d / "report.zip"
```

**Вхідні дані / Інтерфейс:** `make_zip(src: Path, zip_path: Path) -> int`

**Очікуваний результат:** повертає `3`; імена в архіві: `charts/q1.png`, `charts/q2.png`, `summary.txt`.

**Приклад:**
Input: `make_zip(src, zip_path)`
Output: `3`

**Edge Cases:**
- порожня директорія дає архів без файлів і `0`
- імена в архіві без абсолютних шляхів

**Критерії приймання:**
- `namelist()` містить 3 відносні імена
- `testzip()` повертає `None`
- файли стиснуто (`ZIP_DEFLATED`)

**Starter Code:**
```python
import zipfile
from pathlib import Path

def make_zip(src: Path, zip_path: Path) -> int:
    n = 0
    with zipfile.ZipFile(zip_path, "______", compression=zipfile.______) as zf:
        for p in sorted(src.rglob("*")):
            if p.______():
                zf.write(p, arcname=p.______(src))
                n += 1
    return n
```

---

### Task 10 — Атомарний запис файлу
**Складність:** Basic

**Сценарій:** Конфіг читає інший процес у будь-який момент. Якщо скрипт впаде посеред запису, напівзаписаного файлу лишитися не має.

**Умова:** Запиши текст у цільовий файл так, щоб читач бачив або старий вміст, або новий повністю. Тимчасовий файл створюй у тому ж каталозі, що й цільовий.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab10_"))
target = d / "config.ini"
target.write_text("old=1\n")
```

**Вхідні дані / Інтерфейс:** `atomic_write(target: Path, text: str) -> None`

**Очікуваний результат:** `config.ini` містить новий текст, інших файлів у каталозі не лишається.

**Приклад:**
Input: `atomic_write(target, "new=2\n")`
Output: `config.ini` = `new=2\n`, у каталозі один файл

**Edge Cases:**
- цільового файлу ще не існує
- помилка під час запису: старий вміст цілий, тимчасовий файл прибрано

**Критерії приймання:**
- після виклику в каталозі лише `config.ini`
- підміна відбувається однією атомарною операцією
- дані скинуто на диск перед підміною
- при винятку `target` лишається зі старим вмістом

**Starter Code:**
```python
import os, tempfile
from pathlib import Path

def atomic_write(target: Path, text: str) -> None:
    fd, tmp_name = tempfile.______(dir=target.parent, prefix=".tmp_")
    try:
        with os.fdopen(fd, "w", encoding="utf-8") as f:
            f.write(text)
            f.______()
            os.______(f.fileno())
        os.______(tmp_name, target)
    except BaseException:
        Path(tmp_name).unlink(missing_ok=True)
        raise
```

---

## Progress Check #1 (після Task 10)

**Що ти вже маєш вміти:**
- `Path.rglob/glob/iterdir`, `is_file/is_dir/exists`, `mkdir(parents, exist_ok)`, `unlink`
- `Path.stat()`: `st_size`, `st_mtime`; `os.utime`
- `shutil.copy2`, `shutil.move`
- `hashlib.sha256` із читанням чанками
- `tempfile.TemporaryDirectory`, `tempfile.mkstemp`
- `zipfile.ZipFile` (режим, `arcname`, стиснення)
- патерн «тимчасовий файл → `fsync` → `os.replace`»
- режим `dry_run` у деструктивних операціях

**Типові інженерні помилки:**
- `Path.rename` між різними файловими системами падає з `OSError`, а `shutil.move` це обробляє
- `f.read()` на великому файлі вичерпує пам'ять
- перевірка `exists()` перед дією створює гонку (TOCTOU)
- тимчасовий файл в іншому каталозі, ніж ціль: `os.replace` перестає бути атомарним
- `mkdir` без `exist_ok=True`: повторний запуск падає
- ZIP з абсолютними шляхами в `arcname` розпаковується «не туди»
- архів створюється всередині каталогу, який пакується: архів потрапляє сам у себе
- `rglob` повертає й директорії, а `is_file()` забули
- видалення без dry-run: дані втрачено без можливості перевірити

**Контрольна задача (без коду):**
Скрипт щоночі переносить файли з `/data/incoming` на змонтований мережевий диск і видаляє оригінали. Опиши словами:
1. чому простий `rename` тут ненадійний;
2. як зробити перенос безпечним, якщо два екземпляри скрипта запустилися одночасно;
3. які три сценарії збою ти перевіриш тестами (наприклад, обрив посеред копіювання).

---

## Special Task — CODING WITHOUT AI #1

**Складність:** Advanced (не входить у лічильник 55)

**Сценарій:** У папці Downloads хаос: файли різних типів, дублікати під різними іменами, однакові імена в різних місцях. Треба навести лад так, щоб нічого не загубилося й скрипт можна було безпечно запускати повторно.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="special01_"))
(root / "sub").mkdir()
(root / "photo.JPG").write_bytes(b"IMG-A")
(root / "photo_copy.jpg").write_bytes(b"IMG-A")      # дублікат за вмістом
(root / "sub" / "photo.jpg").write_bytes(b"IMG-B")   # те саме ім'я, інший вміст
(root / "report.pdf").write_bytes(b"PDF-1")
(root / "notes.txt").write_text("hello")
(root / "archive.tar.gz").write_bytes(b"GZ")
(root / "noext").write_bytes(b"???")
(root / ".hidden").write_text("secret")
```

**Умова:** Напиши функцію/скрипт, що розкладає всі файли з `root` (включно з підпапками) по каталогах за типом (`images`, `documents`, `text`, `archives`, `other`) усередині `root`. Правила:
- розширення порівнюються без урахування регістру; `.tar.gz` це архів;
- ідентичні за вмістом файли зберігаються **один раз**, решта дублікатів видаляється (у dry-run лише повідомляється);
- файли з однаковим іменем, але різним вмістом не затирають один одного;
- приховані файли (з крапкою) не чіпаємо;
- є режим `dry_run`, що нічого не змінює й друкує план;
- у кінці друкується звіт: скільки файлів перенесено, скільки дублікатів видалено, скільки пропущено.

**Очікуваний результат:** після запуску в `root` лише цільові каталоги та `.hidden`; повторний запуск нічого не змінює й повідомляє `0` дій.

**Критерії приймання:**
- `photo.JPG` і `photo_copy.jpg` зведено до одного файлу в `images`, а `sub/photo.jpg` (інший вміст) збережено під унікальним іменем
- dry-run залишає дерево без змін
- повторний запуск ідемпотентний
- порожні підпапки після переносу прибрано
- жодна операція не виходить за межі `root`

**Обмеження:** без підказок, шаблонів і стартового коду; працюй тільки всередині `root`.

---

# LEVEL 2 — ELEMENTARY (Task 11–19)

### Task 11 — Підрахунок файлів за розширенням
**Складність:** Elementary

**Сценарій:** Перед чисткою сховища треба зрозуміти, з чого воно складається.

**Умова:** Порахуй файли за розширенням у всьому дереві. Розширення приводь до нижнього регістру; файли без розширення рахуй під ключем `""`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab11_"))
(d / "sub").mkdir()
for name in ["a.TXT", "b.txt", "c.py", "d.PY", "e.py", "README", "sub/f.txt"]:
    (d / name).write_text("x")
```

**Вхідні дані / Інтерфейс:** `count_by_ext(root: Path) -> dict[str, int]`

**Очікуваний результат:** `{".txt": 3, ".py": 3, "": 1}` (порядок ключів неважливий)

**Приклад:**
Input: `count_by_ext(d)`
Output: `{".txt": 3, ".py": 3, "": 1}`

**Edge Cases:**
- порожня директорія дає `{}`
- директорії не рахуються

**Критерії приймання:**
- регістр розширень не впливає на підрахунок
- `README` потрапляє під ключ `""`
- вкладені файли враховано

**Starter Code:**
```python
from collections import Counter
from pathlib import Path

def count_by_ext(root: Path) -> dict[str, int]:
    counts: Counter[str] = Counter()
    for p in root.rglob("*"):
        if ______:
            counts[______] += 1
    return dict(counts)
```

---

### Task 12 — Масове перейменування з нумерацією
**Складність:** Elementary

**Сценарій:** Після імпорту з телефона фото мають назви `IMG_8841.jpg`, `scan.jpg` тощо. Треба привести їх до єдиного вигляду `vacation_001.jpg`.

**Умова:** Перейменуй усі файли каталогу (без рекурсії) у `<prefix>_NNN<розширення>`, нумеруючи від 1 у порядку відсортованих старих імен. Мінімум 3 цифри. Якщо будь-яка цільова назва вже існує, підніми `FileExistsError` **до** будь-яких змін. Підтримай `dry_run`. Поверни список пар `(старе, нове)`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab12_"))
for name in ["IMG_8841.jpg", "IMG_0002.jpg", "scan.jpg"]:
    (d / name).write_bytes(b"x")
```

**Вхідні дані / Інтерфейс:** `renumber(folder: Path, prefix: str, dry_run: bool = False) -> list[tuple[str, str]]`

**Очікуваний результат:** `[("IMG_0002.jpg","vacation_001.jpg"), ("IMG_8841.jpg","vacation_002.jpg"), ("scan.jpg","vacation_003.jpg")]`

**Приклад:**
Input: `renumber(d, "vacation")`
Output: список із 3 пар вище

**Edge Cases:**
- порожня папка: `[]`
- підкаталоги не чіпаємо
- у каталозі вже є `vacation_001.jpg`

**Критерії приймання:**
- порядок нумерації відповідає відсортованим старим іменам
- при конфлікті жодного файлу не перейменовано
- dry-run нічого не змінює

**Starter Code:**
```python
from pathlib import Path

def renumber(folder: Path, prefix: str, dry_run: bool = False) -> list[tuple[str, str]]:
    files = sorted(p for p in folder.iterdir() if p.is_file())
    plan = []
    for i, p in enumerate(files, start=1):
        new_name = ______
        plan.append((p, folder / new_name))
    # 1) перевірити конфлікти  2) виконати (якщо не dry_run)
    ______
    return [(src.name, dst.name) for src, dst in plan]
```

---

### Task 13 — Найбільші файли
**Складність:** Elementary

**Сценарій:** Диск заповнюється. Потрібен топ-N найбільших файлів, щоб вирішити, що видаляти.

**Умова:** Поверни `n` найбільших файлів дерева як список `(шлях, розмір)` за спаданням розміру. При рівному розмірі порядок за шляхом.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab13_"))
(d / "sub").mkdir()
for name, size in [("tiny.txt", 1), ("small.txt", 10), ("mid.bin", 2000),
                   ("big.bin", 5000), ("huge.bin", 9000), ("sub/deep.bin", 7000)]:
    (d / name).write_bytes(b"x" * size)
```

**Вхідні дані / Інтерфейс:** `largest(root: Path, n: int) -> list[tuple[Path, int]]`

**Очікуваний результат:** для `n=3`: `huge.bin` (9000), `sub/deep.bin` (7000), `big.bin` (5000).

**Приклад:**
Input: `largest(d, 3)`
Output: `[(huge.bin, 9000), (sub/deep.bin, 7000), (big.bin, 5000)]`

**Edge Cases:**
- `n` більше за кількість файлів
- `n = 0`
- директорії не мають потрапляти в список

**Критерії приймання:**
- порядок за розміром спадний
- глибокі файли враховано
- `n=0` дає `[]`

**Starter Code:**
```python
from pathlib import Path

def largest(root: Path, n: int) -> list[tuple[Path, int]]:
    items = [(p, p.stat().st_size) for p in root.rglob("*") if p.is_file()]
    items.sort(key=______)
    return ______
```

---

### Task 14 — Розпакування ZIP
**Складність:** Elementary

**Сценарій:** Щоночі приходить архів із даними. Його потрібно розпакувати в робочу теку й показати, що саме з'явилося.

**Умова:** Розпакуй архів у `dest` (створи, якщо немає). Поверни відсортований список розпакованих **файлів** (без записів-директорій) у вигляді POSIX-шляхів, відносних до `dest`.

**Підготовка середовища:**
```python
import tempfile, zipfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab14_"))
zp = d / "in.zip"
with zipfile.ZipFile(zp, "w") as zf:
    zf.writestr("a.txt", "A")
    zf.writestr("dir/", "")
    zf.writestr("dir/b.txt", "B")
dest = d / "out"     # ще не існує
```

**Вхідні дані / Інтерфейс:** `extract_all(zip_path: Path, dest: Path) -> list[str]`

**Очікуваний результат:** `["a.txt", "dir/b.txt"]`

**Приклад:**
Input: `extract_all(zp, dest)`
Output: `["a.txt", "dir/b.txt"]`

**Edge Cases:**
- порожній архів дає `[]`
- `dest` уже існує
- файл не є ZIP: зрозуміла помилка

**Критерії приймання:**
- `dest` створено автоматично
- у списку немає `dir/`
- вміст розпакованих файлів коректний

**Starter Code:**
```python
import zipfile
from pathlib import Path

def extract_all(zip_path: Path, dest: Path) -> list[str]:
    ______
```

---

### Task 15 — Порівняння двох директорій
**Складність:** Elementary

**Сценарій:** Треба швидко побачити, чим відрізняються дві папки: що є лише в одній, що в обох.

**Умова:** Порівняй два дерева **за відносними шляхами** (вміст не порівнюємо). Поверни словник із трьома відсортованими списками POSIX-шляхів: `only_a`, `only_b`, `common`. Враховуй лише файли.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab15_"))
a, b = d / "A", d / "B"
for base, names in [(a, ["x.txt", "y.txt", "sub/z.txt"]), (b, ["y.txt", "sub/z.txt", "w.txt"])]:
    for n in names:
        p = base / n
        p.parent.mkdir(parents=True, exist_ok=True)
        p.write_text(n)
```

**Вхідні дані / Інтерфейс:** `compare_dirs(a: Path, b: Path) -> dict[str, list[str]]`

**Очікуваний результат:** `{"only_a": ["x.txt"], "only_b": ["w.txt"], "common": ["sub/z.txt", "y.txt"]}`

**Приклад:**
Input: `compare_dirs(a, b)`
Output: словник вище

**Edge Cases:**
- обидві директорії порожні
- одна директорія не існує: `FileNotFoundError`

**Критерії приймання:**
- шляхи POSIX-стилю незалежно від ОС
- списки відсортовані
- директорії не потрапляють у результат

**Starter Code:**
```python
from pathlib import Path

def compare_dirs(a: Path, b: Path) -> dict[str, list[str]]:
    ______
```

---

### Task 16 — Видалення застарілих файлів
**Сценарій:** Логи в робочій теці старші за 30 днів більше нікому не потрібні, але спершу хочеться побачити, що піде під ніж.

**Складність:** Elementary

**Умова:** Видали у каталозі (без рекурсії) файли, старші за `days` діб (строго старші). Час «зараз» передається параметром `now` (для відтворюваності). `dry_run=True` лише збирає список. Поверни відсортований список імен.

**Підготовка середовища:**
```python
import os, tempfile, time
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab16_"))
now = time.time()
for name, age_days in [("old.log", 40), ("mid.log", 31), ("edge.log", 30), ("fresh.log", 5)]:
    p = d / name
    p.write_text("x")
    t = now - age_days * 86400
    os.utime(p, (t, t))
```

**Вхідні дані / Інтерфейс:** `purge_old(folder: Path, days: int, now: float, dry_run: bool = False) -> list[str]`

**Очікуваний результат:** `["mid.log", "old.log"]`; `edge.log` (рівно 30 діб) і `fresh.log` лишаються.

**Приклад:**
Input: `purge_old(d, 30, now)`
Output: `["mid.log", "old.log"]`

**Edge Cases:**
- порожня директорія
- підкаталоги не видаляємо
- `days = 0`

**Критерії приймання:**
- межа «строго старші» дотримана
- dry-run нічого не видаляє
- повторний виклик повертає `[]`

**Starter Code:**
```python
import os
from pathlib import Path

def purge_old(folder: Path, days: int, now: float, dry_run: bool = False) -> list[str]:
    limit = now - days * 86400
    removed = []
    for p in sorted(folder.iterdir()):
        if p.is_file() and ______ < limit:
            removed.append(p.name)
            if not dry_run:
                ______
    return removed
```

---

### Task 17 — Стиснення та розпакування gzip
**Складність:** Elementary

**Сценарій:** Великі текстові вивантаження займають місце. Їх треба стискати потоково й мати змогу відновити байт-у-байт.

**Умова:** Реалізуй `gzip_file`, що створює `<ім'я>.gz` поруч із оригіналом (оригінал лишається), і `gunzip_file`, що розпаковує в указаний файл. Обидві працюють блоками, без читання всього файлу в пам'ять.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab17_"))
src = d / "export.txt"
src.write_text("рядок даних 12345\n" * 200_000, encoding="utf-8")
```

**Вхідні дані / Інтерфейс:**
- `gzip_file(src: Path) -> Path`
- `gunzip_file(gz: Path, dst: Path) -> None`

**Очікуваний результат:** `export.txt.gz` істотно менший за оригінал; після `gunzip_file(gz, d/"restored.txt")` вміст ідентичний оригіналу.

**Приклад:**
Input: `gz = gzip_file(src); gunzip_file(gz, d / "restored.txt")`
Output: `restored.txt` збігається з `export.txt` байт-у-байт

**Edge Cases:**
- порожній вхідний файл
- `src` не існує

**Критерії приймання:**
- `.gz` менший за оригінал
- оригінал не видалено
- SHA-256 відновленого файлу збігається з оригіналом

**Starter Code:**
```python
import gzip, shutil
from pathlib import Path

def gzip_file(src: Path) -> Path:
    out = src.with_name(src.name + ".gz")
    with src.open("rb") as fin, gzip.open(out, "wb") as fout:
        ______
    return out

def gunzip_file(gz: Path, dst: Path) -> None:
    ______
```

---

### Task 18 — Розмір директорії та читабельний формат
**Складність:** Elementary

**Сценарій:** У звіті про диск «3536» нічого не каже. Потрібно `3.5 KiB`.

**Умова:** Реалізуй `dir_size` (сума розмірів звичайних файлів у дереві; симлінки не слідуємо й не рахуємо) та `human` (двійкові префікси B, KiB, MiB, GiB, TiB; до 1 KiB виводь ціле число з ` B`, інакше одна цифра після коми).

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab18_"))
(d / "sub").mkdir()
(d / "a.bin").write_bytes(b"x" * 1000)
(d / "sub" / "b.bin").write_bytes(b"x" * 2000)
(d / "sub" / "c.bin").write_bytes(b"x" * 536)
os.symlink(d / "a.bin", d / "link.bin")      # [POSIX] не має враховуватися
```

**Вхідні дані / Інтерфейс:**
- `dir_size(root: Path) -> int`
- `human(n: int) -> str`

**Очікуваний результат:** `dir_size(d) == 3536`; `human(3536) == "3.5 KiB"`.

**Приклад:**
Input: `human(0)`, `human(1023)`, `human(1536)`, `human(1048576)`
Output: `"0 B"`, `"1023 B"`, `"1.5 KiB"`, `"1.0 MiB"`

**Edge Cases:**
- порожня директорія: `0`
- від'ємне значення: `ValueError`

**Критерії приймання:**
- симлінк не збільшує розмір
- усі чотири приклади `human` збігаються
- значення ≥ 1 TiB не ламають функцію

**Starter Code:**
```python
from pathlib import Path

def dir_size(root: Path) -> int:
    ______

def human(n: int) -> str:
    ______
```

---

### Task 19 — Режим «лише читання» для дерева **[POSIX]**
**Складність:** Elementary

**Сценарій:** Після закриття звітного періоду файли мають стати незмінними. Скрипт запускають щоразу, тож не чіпає те, що вже заблоковано.

**Умова:** Зніми право запису з усіх звичайних файлів дерева, виставивши режим `0o444`. Поверни кількість файлів, у яких режим **справді змінився**. Символічні посилання не слідуємо.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab19_"))
(d / "sub").mkdir()
for name in ["a.txt", "sub/b.txt", "c.txt"]:
    (d / name).write_text("x")
    os.chmod(d / name, 0o644)
os.chmod(d / "c.txt", 0o444)    # уже заблокований
```

**Вхідні дані / Інтерфейс:** `lock_down(root: Path) -> int`

**Очікуваний результат:** перший виклик повертає `2`, другий `0`.

**Приклад:**
Input: `lock_down(d)` двічі
Output: `2`, потім `0`

**Edge Cases:**
- порожня директорія
- файл уже `0o444`

**Критерії приймання:**
- усі файли мають режим `0o444`
- директорії не змінено
- лічильник коректний у двох запусках

**Starter Code:**
```python
import stat
from pathlib import Path

def lock_down(root: Path) -> int:
    changed = 0
    for p in root.rglob("*"):
        if p.is_symlink() or not p.is_file():
            continue
        mode = stat.S_IMODE(p.stat().st_mode)
        if mode != 0o444:
            ______
            changed += 1
    return changed
```

---

# LEVEL 3 — INTERMEDIATE (Task 20–39)

### Task 20 — Групи дублікатів за вмістом
**Складність:** Intermediate

**Сценарій:** У сховищі фото та документів накопичились копії під різними іменами. Потрібні групи ідентичних файлів, щоб вирішити, що лишити.

**Умова:** Знайди групи файлів з ідентичним вмістом. Порожні файли ігноруй. Файли з унікальним розміром не мають читатися взагалі.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab20_"))
(d / "sub").mkdir()
files = {"a.txt": "same", "c.txt": "same", "sub/b.txt": "same",
         "d.txt": "diff", "e.txt": "sAme", "z1.dat": "", "z2.dat": ""}
for n, c in files.items():
    (d / n).write_text(c)
```

**Вхідні дані / Інтерфейс:** `find_duplicates(root: Path) -> list[list[Path]]`

**Очікуваний результат:** `[[a.txt, c.txt, sub/b.txt]]`. Групи відсортовані за першим шляхом, усередині групи шляхи відсортовані.

**Приклад:**
Input: `find_duplicates(d)`
Output: одна група з трьох файлів

**Edge Cases:**
- однаковий розмір, різний вміст (`d.txt`, `e.txt`)
- порожні файли
- дублікати в різних підкаталогах

**Критерії приймання:**
- повертається рівно одна група
- порожні файли не потрапили в результат
- файл з унікальним розміром не відкривався (перевір підміною `open`/лічильником)
- результат детермінований

---

## Progress Check #2 (після Task 20)

**Що ти вже маєш вміти:**
- `gzip.open`, `shutil.copyfileobj`, `tarfile.open("w:gz")` з фільтром
- `Path.chmod`, `stat.S_IMODE`, `os.utime`
- `zipfile.ZipFile.extractall/namelist/infolist`
- `Counter`/словники для агрегації, сортування за складеним ключем
- двоетапна дедуплікація «розмір → хеш»

**Типові інженерні помилки:**
- порівняння `mtime` як float без допуску (FAT має крок 2 с)
- недетермінований порядок `iterdir`/`os.walk` у тестах і архівах
- `tar.add(dir)` тягне за собою всі вкладені файли, включно з тими, що треба виключити
- `Path.rglob("*")` не слідує симлінкам на директорії, а `os.walk(followlinks=True)` слідує й може піти по колу
- хешування всіх файлів без попередньої фільтрації за розміром
- `shutil.copyfileobj` без контекстного менеджера: дескриптор тече при винятку
- режим (mode) після `write_text` залежить від `umask`, а тести чекають конкретного значення
- порівняння розмірів у байтах і «кілобайтах» різними осями (1000 vs 1024)

**Контрольна задача (без коду):**
Є каталог на 2 ТБ із мільйоном файлів. Потрібно знайти дублікати за вмістом. Опиши словами послідовність фільтрів (від найдешевшого до найдорожчого), які скоротять кількість прочитаних байтів у тисячі разів, і поясни, як обробити жорсткі посилання.

---

## Special Task — CODING WITHOUT AI #2

**Складність:** Advanced (не входить у лічильник 55)

**Сценарій:** Нічний бекап не має падати від одного битого файлу. У дереві є файл без прав читання, битий симлінк, файл, що зникає під час копіювання, ім'я з нестандартними символами й порожня директорія.

**Підготовка середовища (POSIX):**
```python
import os, tempfile
from pathlib import Path

base = Path(tempfile.mkdtemp(prefix="special02_"))
src = base / "src"; (src / "empty_dir").mkdir(parents=True)
(src / "good.txt").write_text("good")
(src / "secret.txt").write_text("nope"); os.chmod(src / "secret.txt", 0o000)
os.symlink("missing", src / "broken_link")
(src / "weird name (1).txt").write_text("w")
(src / "unicode_файл.txt").write_text("u")
```

**Умова:** Напиши скрипт, який створює резервну копію `src` у tar.gz (або директорію `dst`), і:
- продовжує роботу після помилок на окремих файлах;
- веде лог помилок (шлях + причина) у файлі поряд із копією;
- повертає код `0` якщо помилок не було, `1` якщо були нефатальні, `2` якщо `src` недоступний;
- не залишає недописаного архіву при аварійному завершенні;
- підтримує `--dry-run`.

**Критерії приймання:**
- `good.txt`, `weird name (1).txt`, `unicode_файл.txt` в архіві
- `secret.txt` у лозі помилок, але не зупиняє процес
- порожня директорія збережена
- повторний запуск не залишає тимчасових файлів

**Обмеження:** без підказок, шаблонів і стартового коду.

---

### Task 21 — Розкладання за датою модифікації
**Складність:** Intermediate

**Сценарій:** Фотоархів треба розкласти в `YYYY/MM` за датою останньої зміни (UTC).

**Умова:** Перенеси всі файли дерева в `root/YYYY/MM/`. Якщо в цільовій теці вже є файл із таким ім'ям, не затирай, а додай суфікс `_1`, `_2`… перед розширенням. Файли, що вже лежать у правильному `YYYY/MM`, не чіпай. Підтримай `dry_run`. Поверни кількість перенесених файлів по місяцях.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab21_"))
(d / "old").mkdir()
spec = {"img.jpg": 1_700_000_000, "old/img.jpg": 1_700_100_000,
        "june.txt": 1_690_000_000, "dec.txt": 1_701_728_000}
for n, t in spec.items():
    (d / n).write_text(n)
    os.utime(d / n, (t, t))
```

**Вхідні дані / Інтерфейс:** `organize_by_month(root: Path, dry_run: bool = False) -> dict[str, int]`

**Очікуваний результат:** `{"2023/11": 2, "2023/07": 1, "2023/12": 1}`; у `2023/11` лежать `img.jpg` та `img_1.jpg`.

**Приклад:**
Input: `organize_by_month(d)` двічі
Output: перший виклик словник вище, другий `{}`

**Edge Cases:**
- порожні підкаталоги після переносу
- повторний запуск
- однакові імена в одному місяці

**Критерії приймання:**
- жодного файлу не втрачено (сума вмісту збігається)
- повторний запуск повертає `{}`
- dry-run не змінює дерево
- порожній `old/` прибрано

---

### Task 22 — Безпечні імена (snake_case)
**Складність:** Intermediate

**Сценарій:** У файлових іменах пробіли, дужки й змішаний регістр ламають скрипти нижче за течією.

**Умова:** Приведи імена файлів каталогу (без рекурсії) до вигляду: нижній регістр, кожна послідовність небуквено-цифрових символів замінюється одним `_`, `_` по краях відкидаються, розширення в нижньому регістрі. Літери кирилиці зберігаються. При колізії додавай `_1`, `_2`… Обробляй файли в порядку відсортованих старих імен. Поверни пари `(старе, нове)` лише для змінених. Підтримай `dry_run`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab22_"))
for n in ["My Report (final).txt", "my_report_final.txt", "Budget 2026.XLSX",
          "already_ok.csv", "Звіт 2026.doc"]:
    (d / n).write_text(n)
```

**Вхідні дані / Інтерфейс:** `sanitize_names(folder: Path, dry_run: bool = False) -> list[tuple[str, str]]`

**Очікуваний результат (множина пар):**
- `Budget 2026.XLSX` → `budget_2026.xlsx`
- `My Report (final).txt` → `my_report_final_1.txt` (бо `my_report_final.txt` уже існує)
- `Звіт 2026.doc` → `звіт_2026.doc`
- `already_ok.csv` та `my_report_final.txt` без змін

**Приклад:**
Input: `sanitize_names(d)`
Output: 3 пари вище

**Edge Cases:**
- ім'я складається лише зі спецсимволів
- повторний запуск
- два файли, що нормалізуються до однієї назви

**Критерії приймання:**
- жоден файл не затерто
- повторний запуск повертає `[]`
- dry-run нічого не змінює

---

### Task 23 — Звіт по першому рівню каталогів (без `pathlib`)
**Складність:** Intermediate

**Сценарій:** Треба швидко побачити, яка тека скільки займає.

**Умова:** Для кожної підтеки першого рівня порахуй кількість файлів і сумарний розмір (рекурсивно). Файли, що лежать безпосередньо в корені, віднеси до ключа `"."`. Порожня підтека дає `(0, 0)`. Симлінки не слідуємо.

**Підготовка середовища:**
```python
import os, tempfile

root = tempfile.mkdtemp(prefix="lab23_")
def w(rel, n):
    p = os.path.join(root, rel)
    os.makedirs(os.path.dirname(p), exist_ok=True)
    with open(p, "wb") as f:
        f.write(b"x" * n)
w("readme.txt", 50); w("docs/a.txt", 100); w("docs/b.txt", 200)
w("docs/sub/c.txt", 300); w("media/v.bin", 1000)
os.makedirs(os.path.join(root, "empty"))
```

**Вхідні дані / Інтерфейс:** `summary(root: str) -> dict[str, tuple[int, int]]` (ключі відсортовані)

**Очікуваний результат:** `{".": (1, 50), "docs": (3, 600), "empty": (0, 0), "media": (1, 1000)}`

**Приклад:**
Input: `summary(root)`
Output: словник вище

**Edge Cases:**
- порожній корінь
- тека без прав доступу пропускається з повідомленням у `stderr`

**Критерії приймання:**
- значення збігаються з очікуваними
- ключі в порядку сортування
- симлінк на сусідню теку не дублює підрахунок

**Обмеження:** модуль `pathlib` заборонено, лише `os`/`os.path`.

---

### Task 24 — Маніфест і порівняння станів
**Складність:** Intermediate

**Сценарій:** Інкрементальний бекап має знати, що додалось, змінилось і зникло з моменту минулого запуску.

**Умова:** Побудуй маніфест дерева у вигляді `{відносний_POSIX_шлях: {"size": int, "sha256": str}}` і порівняй два маніфести. Зміну визнавай за вмістом, а не за `mtime`. Маніфест має серіалізуватися в JSON і читатися назад без втрат.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab24_"))
old_dir, new_dir = d / "old", d / "new"
for base, files in [(old_dir, {"a.txt": "1", "b.txt": "2", "c.txt": "3"}),
                    (new_dir, {"a.txt": "1", "b.txt": "22", "d.txt": "4"})]:
    base.mkdir()
    for n, c in files.items():
        (base / n).write_text(c)
```

**Вхідні дані / Інтерфейс:**
- `build_manifest(root: Path) -> dict[str, dict]`
- `diff_manifests(old: dict, new: dict) -> dict[str, list[str]]` з ключами `added`, `removed`, `changed`

**Очікуваний результат:** `{"added": ["d.txt"], "removed": ["c.txt"], "changed": ["b.txt"]}`

**Приклад:**
Input: `diff_manifests(build_manifest(old_dir), build_manifest(new_dir))`
Output: словник вище

**Edge Cases:**
- файл зі зміненим `mtime`, але тим самим вмістом: не `changed`
- порожнє дерево
- однакова довжина, різний вміст

**Критерії приймання:**
- `json.loads(json.dumps(m)) == m`
- зміна `mtime` без зміни вмісту не потрапляє в `changed`
- списки відсортовані

---

### Task 25 — Детермінований tar.gz із виключеннями
**Складність:** Intermediate

**Сценарій:** Архів проєкту не повинен містити кеш, артефакти збірки та `.git`. Два запуски на незмінному дереві мають давати однаковий перелік файлів в однаковому порядку.

**Умова:** Запакуй дерево в `.tar.gz`, виключивши файли за списком glob-шаблонів, що застосовуються до POSIX-шляху відносно кореня. Імена в архіві відносні, порядок відсортований, записів для директорій немає. Поверни список імен в архіві.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab25_"))
proj = d / "project"
for n in ["main.py", "util.py", "notes.txt", "__pycache__/main.cpython-312.pyc",
          ".git/HEAD", "build/out.o"]:
    p = proj / n
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(n)
EXCLUDE = ["*.pyc", "__pycache__/*", ".git/*", "build/*"]
out = d / "project.tar.gz"
```

**Вхідні дані / Інтерфейс:** `make_tarball(root: Path, out: Path, exclude: list[str]) -> list[str]`

**Очікуваний результат:** `["main.py", "notes.txt", "util.py"]`

**Приклад:**
Input: `make_tarball(proj, out, EXCLUDE)`
Output: `["main.py", "notes.txt", "util.py"]`

**Edge Cases:**
- порожній список виключень
- архів лежить усередині пакованого дерева
- шаблон, що збігається з усім

**Критерії приймання:**
- у архіві рівно три файли
- двічі створені архіви мають однаковий перелік імен
- архів не потрапляє сам у себе
- імена не починаються з `/` чи `..`

---

### Task 26 — Осиротілі та «висячі» файли
**Складність:** Intermediate

**Сценарій:** Каталог даних має індекс `index.json`. Частина файлів на диску вже ніде не згадується, а частина записів в індексі вказує в нікуди.

**Умова:** Порівняй вміст каталогу з переліком в індексі. Поверни кортеж `(orphans, dangling)`: файли на диску, яких немає в індексі; записи індексу без файлів. Шляхи в індексі нормалізуй (`./x` і `x` однакові); запис, що виходить за межі каталогу (`../x`), викликає `ValueError`. Сам `index.json` у порівнянні не бере участі.

**Підготовка середовища:**
```python
import json, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab26_"))
(d / "sub").mkdir()
for n in ["a.dat", "sub/b.dat", "c.dat", "sub/d.dat"]:
    (d / n).write_bytes(b"x")
(d / "index.json").write_text(json.dumps(
    {"files": ["a.dat", "./sub/b.dat", "missing.dat"]}))
```

**Вхідні дані / Інтерфейс:** `audit_index(root: Path) -> tuple[list[str], list[str]]`

**Очікуваний результат:** `(["c.dat", "sub/d.dat"], ["missing.dat"])`

**Приклад:**
Input: `audit_index(d)`
Output: `(["c.dat", "sub/d.dat"], ["missing.dat"])`

**Edge Cases:**
- порожній індекс
- дублікати в індексі
- запис `../secret`

**Критерії приймання:**
- `./sub/b.dat` розпізнано як `sub/b.dat`
- списки відсортовані
- `../secret` дає `ValueError`

---

### Task 27 — Прибирання порожніх каталогів
**Складність:** Intermediate

**Сценарій:** Після чисток лишилися вкладені порожні папки. Тека, що стала порожньою внаслідок видалення дочірніх, теж має зникнути.

**Умова:** Видали всі порожні каталоги знизу вгору, але не сам корінь. `dry_run` лише повертає перелік того, що було б видалено (включно з батьками, які стали б порожніми). Поверни відсортований список відносних POSIX-шляхів.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab27_"))
for n in ["a/b/c", "a/keep", "x", "y/z"]:
    (d / n).mkdir(parents=True)
(d / "a" / "keep" / "file.txt").write_text("!")
```

**Вхідні дані / Інтерфейс:** `prune_empty_dirs(root: Path, dry_run: bool = False) -> list[str]`

**Очікуваний результат:** `["a/b", "a/b/c", "x", "y", "y/z"]`; `a`, `a/keep` і корінь лишаються.

**Приклад:**
Input: `prune_empty_dirs(d, dry_run=True)`, потім `prune_empty_dirs(d)`
Output: список вище в обох випадках; після другого запуску третій повертає `[]`

**Edge Cases:**
- корінь порожній
- тека із симлінком усередині не вважається порожньою
- повторний запуск

**Критерії приймання:**
- dry-run повертає той самий список, що й реальний запуск
- корінь ніколи не видаляється
- після запуску жодної порожньої підтеки не лишається

---

### Task 28 — Інкрементальне копіювання дерева
**Складність:** Intermediate

**Сценарій:** Щоденна синхронізація має копіювати лише нове та змінене, а не все підряд.

**Умова:** Скопіюй `src` у `dst`: копіюй файл, якщо його немає в `dst`, або розмір відрізняється, або `mtime` у `src` новіший. Після копіювання `mtime` копії має збігатися з оригіналом (інакше наступний запуск знову все скопіює). Поверни `(copied, skipped)`. Видалення зайвого в `dst` не потрібне.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab28_"))
src, dst = d / "src", d / "dst"
(src / "sub").mkdir(parents=True); (dst / "sub").mkdir(parents=True)
(src / "a.txt").write_text("aaa");            (dst / "a.txt").write_text("aaa")
(src / "sub" / "b.txt").write_text("new-bigger"); (dst / "sub" / "b.txt").write_text("old")
(src / "sub" / "c.txt").write_text("ccc")
for p in (src / "a.txt", dst / "a.txt"):
    os.utime(p, (1_650_000_000, 1_650_000_000))
```

**Вхідні дані / Інтерфейс:** `incremental_copy(src: Path, dst: Path) -> tuple[int, int]`

**Очікуваний результат:** `(2, 1)`; другий виклик повертає `(0, 3)`.

**Приклад:**
Input: `incremental_copy(src, dst)` двічі
Output: `(2, 1)`, потім `(0, 3)`

**Edge Cases:**
- `dst` не існує
- файл у `src` замінено каталогом у `dst`
- порожнє `src`

**Критерії приймання:**
- повторний запуск нічого не копіює
- `mtime` копій збігається з оригіналами
- `dst` створюється з потрібною структурою

---

### Task 29 — Аудит символічних посилань **[POSIX]**
**Складність:** Intermediate

**Сценарій:** Перед архівацією треба перевірити, що симлінки в дереві не ведуть у нікуди й не виходять за межі пакованої теки.

**Умова:** Класифікуй усі симлінки в дереві (не слідуючи за ними під час обходу): `ok` (веде на існуючий об'єкт усередині `root`), `broken` (ціль не існує), `escaping` (ціль існує, але поза `root`), `loops` (циклічні посилання). Поверни відсортовані відносні імена.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

base = Path(tempfile.mkdtemp(prefix="lab29_"))
root = base / "root"; root.mkdir()
(base / "outside.txt").write_text("secret")
(root / "real.txt").write_text("ok")
os.symlink("real.txt", root / "ok_link")
os.symlink("missing.txt", root / "broken")
os.symlink("../outside.txt", root / "escape")
os.symlink("loop_b", root / "loop_a")
os.symlink("loop_a", root / "loop_b")
```

**Вхідні дані / Інтерфейс:** `audit_symlinks(root: Path) -> dict[str, list[str]]`

**Очікуваний результат:** `{"ok": ["ok_link"], "broken": ["broken"], "escaping": ["escape"], "loops": ["loop_a", "loop_b"]}`

**Приклад:**
Input: `audit_symlinks(root)`
Output: словник вище

**Edge Cases:**
- абсолютний симлінк усередині `root`
- симлінк на директорію
- ланцюжок симлінків

**Критерії приймання:**
- кожен симлінк у рівно одній категорії
- скрипт не зависає на циклах
- `real.txt` не потрапляє в результат

---

### Task 30 — Розбиття файлу на частини та склейка
**Складність:** Intermediate

**Сценарій:** Великий файл треба передати через канал із лімітом 1 МБ на вкладення.

**Умова:** Розбий файл на частини заданого розміру (`name.part001`, `name.part002`, …) і склей їх назад у інший файл. Поверни список частин. Порожній файл дає порожній список; склейка порожнього списку дає порожній файл. Читай і пиши блоками ≤ 64 КіБ.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab30_"))
src = d / "data.bin"
src.write_bytes(bytes(range(256)) * 9766)     # 2 500 096 байт
parts_dir = d / "parts"
```

**Вхідні дані / Інтерфейс:**
- `split_file(src: Path, part_size: int, out_dir: Path) -> list[Path]`
- `join_parts(parts: list[Path], dst: Path) -> None`

**Очікуваний результат:** для `part_size = 1_000_000` три частини: 1 000 000, 1 000 000, 500 096 байт. Склеєний файл збігається з оригіналом.

**Приклад:**
Input: `parts = split_file(src, 1_000_000, parts_dir); join_parts(parts, d / "back.bin")`
Output: `back.bin` ідентичний `data.bin`

**Edge Cases:**
- розмір файлу кратний `part_size`
- `part_size <= 0`: `ValueError`
- порожній файл

**Критерії приймання:**
- кількість і розміри частин коректні
- хеші оригіналу та склейки збігаються
- пікова пам'ять не залежить від `part_size`

**Обмеження:** `shutil` заборонено. Копіюй блоками вручну.

---

## Progress Check #3 (після Task 30)

**Що ти вже маєш вміти:**
- групування за часом у UTC (`datetime.fromtimestamp(ts, tz=timezone.utc)`)
- нормалізація імен та унікалізація при колізіях із детермінованим порядком обробки
- JSON-маніфести: побудова, серіалізація, порівняння за вмістом
- `tarfile` з фільтрацією та відсортованим порядком записів
- `os.scandir`/`os.walk` без `pathlib`
- `os.symlink`, `os.readlink`, `os.path.realpath`, `Path.is_relative_to`
- розбиття та склейка файлів блоками

**Типові інженерні помилки:**
- локальний час замість UTC у групуванні: файли «переїжджають» між місяцями залежно від часового поясу сервера
- `Path.resolve()` на симлінку веде за межі кореня, а перевірка «всередині root» помилково проходить
- видалення порожніх директорій у неправильному порядку (батьки ще не порожні)
- колізії імен вирішуються в порядку `iterdir`, який недетермінований
- після копіювання не збережено `mtime`: наступний інкрементальний запуск знову копіює все
- частини названі `part1…part10`: лексикографічне сортування ставить `part10` перед `part2`
- порівняння файлів за `mtime` замість вмісту
- перевірка `is_relative_to` до нормалізації шляху

**Контрольна задача (без коду):**
Є дві теки з фото: на ноутбуці та на зовнішньому диску. Частина файлів перейменована, частина переміщена в інші підтеки, частина змінена. Опиши словами, як визначити «що нове, що змінено, що просто переїхало», не читаючи кожен файл повністю, і які випадки цей метод розрізнить погано.

---

## Special Task — CODING WITHOUT AI #3

**Складність:** Advanced (не входить у лічильник 55)

**Сценарій:** Конфігурація застосунку збирається з шарів: `base/`, `env/prod/`, `local/`. Верхній шар перекриває нижній, а файл-«надгробок» `.wh.<ім'я>` означає «видалити цей файл із підсумкового результату», як в overlayfs.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="special03_"))
layers = [root / "base", root / "prod", root / "local"]
def put(layer, rel, text=""):
    p = layer / rel
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(text)
put(layers[0], "app.ini", "base")
put(layers[0], "db/conn.ini", "base-db")
put(layers[0], "legacy.ini", "legacy")
put(layers[1], "app.ini", "prod")
put(layers[1], "db/pool.ini", "prod-pool")
put(layers[2], ".wh.legacy.ini")          # надгробок: прибрати legacy.ini
put(layers[2], "db/conn.ini", "local-db")
```

**Умова:** Напиши інструмент, що матеріалізує злитий вигляд шарів у каталог `out/`:
- файл верхнього шару перекриває нижній;
- `.wh.<ім'я>` видаляє файл із підсумку (і сам у підсумок не потрапляє);
- підтримай `.wh..opq` (непрозорий каталог): усе з нижніх шарів у цьому каталозі ігнорується;
- режим `--dry-run` друкує план (звідки береться кожен файл);
- повторний запуск на незмінних вхідних даних не міняє `out/` і повідомляє `0` змін;
- прибирає з `out/` файли, яких у новому підсумку вже немає.

**Очікуваний результат для прикладу:** `app.ini == "prod"`, `db/conn.ini == "local-db"`, `db/pool.ini == "prod-pool"`, `legacy.ini` відсутній.

**Критерії приймання:**
- результат збігається з очікуваним
- повторний запуск: `0` змін
- зміна лише одного шару оновлює лише залежні файли
- `.wh.` файли не потрапляють в `out/`

**Обмеження:** без підказок, шаблонів і стартового коду.

---

### Task 31 — Ротація резервних копій за ім'ям
**Складність:** Intermediate

**Сценарій:** Каталог бекапів наповнюється щодня. Лишити треба K найновіших, а «найновіша» визначається датою в імені, бо `mtime` міняється при копіюванні.

**Умова:** Залиш `keep` найновіших архівів за форматом `backup-YYYYMMDD-HHMMSS.tar.gz` (дата й час у імені), решту видали. Файли, що не відповідають формату (у тому числі з неіснуючою датою), не чіпай, але виведи про них попередження. `keep < 0` це `ValueError`; `keep = 0` видаляє всі відповідні. Підтримай `dry_run`. Поверни список видалених імен від найстарішого.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab31_"))
names = ["backup-20260901-020000.tar.gz", "backup-20260902-020000.tar.gz",
         "backup-20260903-020000.tar.gz", "backup-20260904-020000.tar.gz",
         "backup-20260905-020000.tar.gz",
         "notes.txt", "backup-latest.tar.gz", "backup-20261399-000000.tar.gz"]
for i, n in enumerate(names):
    p = d / n
    p.write_text(n)
    t = 1_700_000_000 - i * 1000       # mtime навмисно «навпаки»
    os.utime(p, (t, t))
```

**Вхідні дані / Інтерфейс:** `rotate_backups(folder: Path, keep: int, dry_run: bool = False) -> list[str]`

**Очікуваний результат (keep=2):** видалено `backup-20260901…`, `…0902…`, `…0903…`; решта на місці.

**Приклад:**
Input: `rotate_backups(d, keep=2)`
Output: `["backup-20260901-020000.tar.gz", "backup-20260902-020000.tar.gz", "backup-20260903-020000.tar.gz"]`

**Edge Cases:**
- файлів менше за `keep`
- порожній каталог
- повторний запуск

**Критерії приймання:**
- «найновіші» визначаються за ім'ям, а не за `mtime`
- сторонні файли не зачеплено, для них є попередження
- повторний запуск повертає `[]`

---

### Task 32 — Пошук байтового рядка у файлах (`grep -l`)
**Складність:** Intermediate

**Сценарій:** Треба знайти, у яких файлах згадується ідентифікатор, навіть якщо це бінарні дані чи кодування не UTF-8.

**Умова:** Поверни відсортований список файлів дерева, що містять заданий байтовий рядок. Збіг може перетинати межу блоків читання. Пам'ять не залежить від розміру файлу. Файл без прав читання пропускай із записом у `logging`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab32_"))
needle = b"NEEDLE-42"
(d / "hit.txt").write_bytes(b"prefix " + needle + b" suffix")
(d / "miss.txt").write_bytes(b"nothing here")
(d / "cp1251.txt").write_bytes("помилка: NEEDLE-42\n".encode("cp1251"))
big = bytearray(b"x" * 200_000)
pos = 65_536 - 4                              # збіг перетинає межу 64 КіБ
big[pos:pos + len(needle)] = needle
(d / "big.bin").write_bytes(bytes(big))
```

**Вхідні дані / Інтерфейс:** `files_containing(root: Path, needle: bytes) -> list[Path]`

**Очікуваний результат:** `[big.bin, cp1251.txt, hit.txt]`

**Приклад:**
Input: `files_containing(d, b"NEEDLE-42")`
Output: список із 3 файлів

**Edge Cases:**
- порожній `needle` це `ValueError`
- збіг на самому початку/кінці файлу
- файл менший за розмір блока

**Критерії приймання:**
- `big.bin` знайдено (збіг на межі блоків)
- файл читається блоками, а не цілком
- відсутній доступ не зупиняє пошук

---

### Task 33 — Вирівнювання mtime за еталоном
**Складність:** Intermediate

**Сценарій:** Після відновлення з бекапу всі файли мають «сьогоднішню» дату. Еталонне дерево зберегло справжні `mtime`, і треба їх перенести.

**Умова:** Для кожного файлу, що є в обох деревах, виставити в цільовому дереві `mtime` еталонного, якщо вони відрізняються більше ніж на 1 с. Файли, яких немає в цілі, пропусти; зайві файли цілі не чіпай. Поверни кількість оновлених.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab33_"))
ref, tgt = d / "ref", d / "tgt"
for base in (ref, tgt):
    (base / "sub").mkdir(parents=True)
for n, t_ref, t_tgt in [("a.txt", 1_600_000_000, 1_700_000_000),
                        ("sub/b.txt", 1_610_000_000, 1_700_000_000)]:
    (ref / n).write_text(n); (tgt / n).write_text(n)
    os.utime(ref / n, (t_ref, t_ref)); os.utime(tgt / n, (t_tgt, t_tgt))
(ref / "d.txt").write_text("only in ref")
(tgt / "c.txt").write_text("only in tgt")
```

**Вхідні дані / Інтерфейс:** `sync_mtimes(ref: Path, tgt: Path) -> int`

**Очікуваний результат:** перший виклик `2`, другий `0`.

**Приклад:**
Input: `sync_mtimes(ref, tgt)` двічі
Output: `2`, потім `0`

**Edge Cases:**
- різниця менша за 1 с: не оновлювати
- порожні директорії
- файл в еталоні, але каталог у цілі

**Критерії приймання:**
- `mtime` у цілі збігається з еталоном
- `c.txt` і `d.txt` не зачеплено (`d.txt` не створено)
- повторний запуск повертає `0`

---

### Task 34 — Сплющення дерева з унікальними іменами
**Складність:** Intermediate

**Сценарій:** З десятка підтек треба зібрати всі фото в одну плоску теку, не втративши жодного файлу, навіть якщо імена збігаються.

**Умова:** Перенеси всі файли дерева `src` у плоску теку `dst`. Порядок обробки: відсортовані відносні шляхи. При колізії імен додавай `_1`, `_2`… перед розширенням. `dst` усередині `src` це `ValueError`. Порожні підтеки в `src` після переносу видали (корінь лишається). Поверни словник `відносний_шлях_джерела → нове_ім'я`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab34_"))
src, dst = d / "src", d / "flat"
for n in ["a/photo.jpg", "b/photo.jpg", "c/photo.jpg", "c/doc.txt"]:
    p = src / n
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(n)
```

**Вхідні дані / Інтерфейс:** `flatten(src: Path, dst: Path) -> dict[str, str]`

**Очікуваний результат:** `{"a/photo.jpg": "photo.jpg", "b/photo.jpg": "photo_1.jpg", "c/doc.txt": "doc.txt", "c/photo.jpg": "photo_2.jpg"}`

**Приклад:**
Input: `flatten(src, dst)`
Output: словник вище

**Edge Cases:**
- `dst` уже містить `photo.jpg`
- файли без розширення
- `dst` усередині `src`

**Критерії приймання:**
- у `dst` рівно 4 файли зі збереженим вмістом
- у `src` немає порожніх підтек
- `dst` всередині `src` це `ValueError` ще до змін

---

### Task 35 — Кеш-каталог: TTL і ліміт розміру
**Складність:** Intermediate

**Сценарій:** Кеш-папка має самоочищатися: прострочені записи видаляються, а якщо загальний розмір усе ще завеликий, то вилучаються найстаріші.

**Умова:** Спершу видали файли старші за `max_age` секунд (строго). Потім, поки сумарний розмір > `max_bytes`, видаляй найстаріші за `mtime`. Поверни імена видалених у порядку видалення. Підтримай `dry_run`. Час «зараз» передається параметром.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab35_"))
now = 1_700_000_000
for name, age in [("f1", 10_000), ("f2", 5_000), ("f3", 300), ("f4", 200), ("f5", 100)]:
    p = d / name
    p.write_bytes(b"x" * 400)
    os.utime(p, (now - age, now - age))
```

**Вхідні дані / Інтерфейс:** `evict(cache: Path, max_age: int, max_bytes: int, now: float, dry_run: bool = False) -> list[str]`

**Очікуваний результат для `max_age=6000`, `max_bytes=1000`:** `["f1", "f2", "f3"]` (f1 за віком; f2 і f3 за розміром).

**Приклад:**
Input: `evict(d, 6000, 1000, now)`
Output: `["f1", "f2", "f3"]`

**Edge Cases:**
- `max_bytes = 0`
- порожній кеш
- однаковий `mtime` (порядок за ім'ям)

**Критерії приймання:**
- після запуску сума розмірів ≤ `max_bytes`
- dry-run повертає той самий список без видалення
- повторний запуск повертає `[]`

---

### Task 36 — Перевірка цілісності за маніфестом
**Складність:** Intermediate

**Сценарій:** Після відновлення з бекапу треба довести, що дерево збігається з маніфестом.

**Умова:** За маніфестом формату `{шлях: {"size": int, "sha256": str}}` визнач статус кожного шляху: `OK`, `MISSING` (є в маніфесті, немає на диску), `CORRUPT` (розмір або хеш не збігаються), `EXTRA` (є на диску, немає в маніфесті). Додай функцію коду повернення: `0` якщо все `OK`, інакше `1`. Якщо розмір не збігається, хешувати файл не потрібно.

**Підготовка середовища:**
```python
import hashlib, tempfile
from pathlib import Path

def entry(data: bytes):
    return {"size": len(data), "sha256": hashlib.sha256(data).hexdigest()}

d = Path(tempfile.mkdtemp(prefix="lab36_"))
files = {"ok.txt": b"good", "gone.txt": b"lost", "bad.txt": b"abcd", "size.txt": b"short"}
manifest = {n: entry(c) for n, c in files.items()}
(d / "ok.txt").write_bytes(b"good")
(d / "bad.txt").write_bytes(b"abce")            # той самий розмір, інший вміст
(d / "size.txt").write_bytes(b"much longer!")   # інший розмір
(d / "extra.txt").write_bytes(b"?")
```

**Вхідні дані / Інтерфейс:**
- `verify(root: Path, manifest: dict) -> dict[str, str]`
- `exit_code(result: dict[str, str]) -> int`

**Очікуваний результат:** `{"bad.txt": "CORRUPT", "extra.txt": "EXTRA", "gone.txt": "MISSING", "ok.txt": "OK", "size.txt": "CORRUPT"}`, код `1`.

**Приклад:**
Input: `verify(d, manifest)`
Output: словник вище

**Edge Cases:**
- порожній маніфест і порожня директорія: усе `OK`, код `0`
- у маніфесті шлях, що є директорією на диску

**Критерії приймання:**
- усі 5 статусів коректні
- `size.txt` не хешується (перевір лічильником читань)
- результат відсортований за ключем

---

### Task 37 — Spool-каталог: обробка черги файлів
**Складність:** Intermediate

**Сценарій:** Зовнішня система складає файли завдань у `spool/`. Один битий файл не має зупиняти чергу.

**Умова:** Обробляй файли `spool/` (без рекурсії) функцією-обробником `handler(path)`. Успішно оброблений файл переноситься в `spool/done/`, при винятку у `spool/failed/` з супровідним файлом `<ім'я>.err`, де текст помилки. Файли з розширенням `.part` (ще пишуться) ігноруй. Вже перенесені файли повторно не обробляються. Імена в `done/` і `failed/` не перезаписуй. Поверни `{"done": n, "failed": m}`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

spool = Path(tempfile.mkdtemp(prefix="lab37_"))
for n in ["job1.json", "job2.json", "job3.json"]:
    (spool / n).write_text(n)
(spool / "job4.json.part").write_text("ще пишеться")

def handler(p: Path):
    if p.name == "job2.json":
        raise ValueError("bad job")
```

**Вхідні дані / Інтерфейс:** `process_spool(spool: Path, handler) -> dict[str, int]`

**Очікуваний результат:** `{"done": 2, "failed": 1}`; `failed/job2.json.err` містить `bad job`; `job4.json.part` лишається на місці.

**Приклад:**
Input: `process_spool(spool, handler)` двічі
Output: `{"done": 2, "failed": 1}`, потім `{"done": 0, "failed": 0}`

**Edge Cases:**
- у `done/` уже є файл з таким ім'ям
- порожній spool
- обробник піднімає `KeyboardInterrupt`: файл лишається в черзі

**Критерії приймання:**
- збій однієї задачі не зупиняє решту
- повторний запуск нічого не робить
- `.part` не чіпається
- при `KeyboardInterrupt` файл не втрачено

---

### Task 38 — Свіжі файли за годинами
**Складність:** Intermediate

**Сценарій:** Для графіка активності треба знати, скільки файлів змінювалося в кожну з останніх годин.

**Умова:** Порахуй файли дерева, змінені за останні `hours` годин відносно `now`, згрупувавши за годиною (UTC) у форматі `YYYY-MM-DDTHH`. Файли старші за вікно ігноруй. Порожні години в результат не потрапляють.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab38_"))
(d / "sub").mkdir()
now = 1_700_000_000          # 2023-11-14T22:13:20Z
for name, ago in [("f1", 600), ("f2", 1200), ("sub/f3", 1800),
                  ("f4", 3 * 3600), ("f5", 25 * 3600)]:
    p = d / name
    p.write_text(name)
    os.utime(p, (now - ago, now - ago))
```

**Вхідні дані / Інтерфейс:** `recent_by_hour(root: Path, now: float, hours: int = 24) -> dict[str, int]`

**Очікуваний результат:** `{"2023-11-14T22": 1, "2023-11-14T21": 2, "2023-11-14T19": 1}`

**Приклад:**
Input: `recent_by_hour(d, now)`
Output: словник вище

**Edge Cases:**
- файл рівно на межі вікна (включається)
- `mtime` у майбутньому (ігнорується)
- порожня директорія

**Критерії приймання:**
- `f5` відсутній
- групування за UTC, а не за локальним часом
- кількість у групах збігається

---

### Task 39 — Нормалізація кінців рядків CRLF → LF
**Складність:** Intermediate

**Сценарій:** Після роботи на Windows у репозиторії змішані кінці рядків. Потрібно виправити текстові файли й не зіпсувати бінарні.

**Умова:** Заміни `\r\n` на `\n` у текстових файлах дерева. Одинокий `\r` лишається. Файл із нульовим байтом у перших 8 КіБ вважається бінарним і пропускається. Запис атомарний, режим доступу зберігається. Пара `\r\n`, що розірвана межею блоків читання, має оброблятися правильно. Не читай файл цілком. Поверни відсортований список змінених файлів.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab39_"))
(d / "a.txt").write_bytes(b"l1\r\nl2\r\n")
(d / "b.py").write_bytes(b"x\r\n")
(d / "c.bin").write_bytes(b"\x00\x01\r\n\x02")
(d / "d.txt").write_bytes(b"already\n")
(d / "e.txt").write_bytes(b"a\r\nb\nc\r")
(d / "edge.txt").write_bytes(b"x" * 65_535 + b"\r\n" + b"tail\n")   # \r\n на межі 64 КіБ
```

**Вхідні дані / Інтерфейс:** `normalize_eol(root: Path) -> list[str]`

**Очікуваний результат:** `["a.txt", "b.py", "e.txt", "edge.txt"]`. `e.txt` стає `b"a\nb\nc\r"`. `edge.txt` не містить `\r`.

**Приклад:**
Input: `normalize_eol(d)`
Output: список вище

**Edge Cases:**
- порожній файл
- файл без кінців рядків
- `\r` як останній байт блока, `\n` на початку наступного

**Критерії приймання:**
- `c.bin` не змінено байт-у-байт
- `edge.txt` коректний
- повторний запуск повертає `[]`
- після запуску в каталозі немає тимчасових файлів

---

# LEVEL 4 — ADVANCED (Task 40–47)

### Task 40 — Безпечне розпакування ZIP (zip-slip і «бомба»)
**Складність:** Advanced

**Сценарій:** Ваш сервіс приймає архіви від користувачів. Запис `../../etc/cron.d/x` в архіві не має вийти за межі теки призначення, а архів на 50 МБ нулів не має з'їсти диск.

**Умова:** Реалізуй розпакування, що:
- спершу перевіряє **всі** записи; якщо хоч один небезпечний (абсолютний шлях, `..`, вихід за межі `dest` після нормалізації), піднімає `UnsafeArchiveError` і **нічого** не розпаковує;
- обмежує сумарний **фактично розпакований** обсяг лімітом; при перевищенні піднімає `UnsafeArchiveError` і прибирає вже створене;
- повертає відсортований список розпакованих файлів.

**Підготовка середовища:**
```python
import tempfile, zipfile
from pathlib import Path

class UnsafeArchiveError(Exception):
    pass

d = Path(tempfile.mkdtemp(prefix="lab40_"))
def mk(name, entries):
    with zipfile.ZipFile(d / name, "w", zipfile.ZIP_DEFLATED) as zf:
        for n, data in entries:
            zf.writestr(n, data)
mk("safe.zip", [("ok.txt", b"1"), ("dir/ok2.txt", b"2")])
mk("slip.zip", [("ok.txt", b"1"), ("../evil.txt", b"x")])
mk("abs.zip",  [("/tmp/evil_abs.txt", b"x")])
mk("deep.zip", [("dir/../../evil3.txt", b"x")])
mk("bomb.zip", [("zeros.bin", b"\x00" * 50_000_000)])
```

**Вхідні дані / Інтерфейс:** `safe_extract(zip_path: Path, dest: Path, max_total_bytes: int = 10_000_000) -> list[str]`

**Очікуваний результат:** `safe.zip` дає `["dir/ok2.txt", "ok.txt"]`. `slip.zip`, `abs.zip`, `deep.zip`, `bomb.zip` піднімають `UnsafeArchiveError`; `dest` після цього порожній (або не існує); поза `dest` нічого не створено.

**Приклад:**
Input: `safe_extract(d/"slip.zip", d/"out")`
Output: `UnsafeArchiveError`; `d/"evil.txt"` не існує, `out` порожній

**Edge Cases:**
- записи-директорії
- Windows-стиль шляху (`..\\x`)
- порожній архів
- `dest` уже існує з файлами (вони не зачіпаються)

**Критерії приймання:**
- жодного файлу поза `dest`
- при помилці `dest` повертається до стану «до виклику»
- «бомба» зупиняється до заповнення диска (перевіряй пікове використання)
- коректний архів розпаковано повністю

---

## Progress Check #4 (після Task 40)

**Що ти вже маєш вміти:**
- розбір імені за форматом (`datetime.strptime`) і ротація «K найновіших»
- пошук байтового рядка в потоці з перекриттям (`len(needle)-1`) між блоками
- синхронізація метаданих з допуском на точність файлової системи
- патерн `spool → done/failed` із супровідним `.err`
- кеш із TTL та обмеженням розміру (LRU за `mtime`)
- перевірка цілісності за маніфестом (`OK/MISSING/CORRUPT/EXTRA`)
- нормалізація `\r\n` у потоці з правильною обробкою межі блока
- безпечний unzip: перевірка всіх записів до розпакування, ліміт фактичного обсягу

**Типові інженерні помилки:**
- пошук по блоках без перекриття: збіг на межі 64 КіБ губиться
- перевірка `startswith(str(dest))` без нормалізації: `dest2/…` проходить перевірку
- довіра до `ZipInfo.file_size` (значення можна підробити)
- ротація за `mtime`, який змінюється при копіюванні, замість дати в імені
- обробник у spool падає, а файл уже перенесено в `done/`
- `\r` наприкінці блока і `\n` на початку наступного оброблено як окремі символи
- хешування файла, розмір якого вже не збігається з маніфестом (зайві читання)
- перевірка «безпечності» лише для першого запису архіву

**Контрольна задача (без коду):**
Сервіс приймає архіви, розпаковує їх і одразу індексує файли. Опиши словами три незалежні перевірки між «файл прийшов» і «файл потрапив в індекс», щоб зловмисний архів не завдав шкоди (шляхи, обсяг, типи записів). Чому однієї перевірки недостатньо?

---

## Special Task — CODING WITHOUT AI #4

**Складність:** Advanced (не входить у лічильник 55)

**Сценарій:** Старе сховище зберігало все плоско: `file_<id>.dat` в одній теці на мільйон файлів. Треба мігрувати на шардований шлях `ab/cd/<id>.dat`, не зупиняючи сервіс, і міграцію можна обірвати в будь-який момент.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

store = Path(tempfile.mkdtemp(prefix="special04_"))
for i in range(1, 2001):
    (store / f"file_{i:06d}.dat").write_bytes(f"data-{i}".encode())
(store / "README.txt").write_text("не чіпати")
```

**Умова:** Напиши міграцію, що переносить `file_000123.dat` у `00/01/000123.dat` (шарди від перших 4 цифр id по 2). Вимоги:
- міграція безпечна до обриву на будь-якій операції (журнал або відновлювана схема);
- повторний запуск завершує недоторкане й не чіпає вже перенесене;
- сторонні файли (`README.txt`) лишаються;
- є функція `locate(store, id) -> Path`, що знаходить файл **і в старому, і в новому розташуванні** у будь-який момент міграції;
- режим `--dry-run`, звіт (перенесено / пропущено / помилок);
- для тестів: змінна оточення `MIGRATE_CRASH_AFTER=k` імітує обрив після k-ї операції.

**Очікуваний результат:** після повного запуску всі 2000 файлів у шардованих теках, `README.txt` на місці, старі плоскі файли відсутні. Після обриву на 700-й операції та повторного запуску результат той самий.

**Критерії приймання:**
- обрив у будь-якій точці не втрачає файлів
- `locate` працює протягом усієї міграції
- повторний запуск ідемпотентний
- жодних файлів поза `store`

**Обмеження:** без підказок, шаблонів і стартового коду.

---

### Task 41 — Атомарне перемикання релізу **[POSIX]**
**Складність:** Advanced

**Сценарій:** Застосунок читає файли з `app/current/`. Деплой має підміняти версію без жодного моменту, коли `current` відсутній або наполовину оновлений. Відкат має бути миттєвим.

**Умова:** Реалізуй:
- `deploy(app: Path, source: Path, name: str, keep: int = 3) -> Path`: копіює `source` у `app/releases/<name>` (спершу в тимчасову теку того ж каталогу, потім перейменування), атомарно переставляє `app/current` на новий реліз, залишає `keep` найновіших релізів (за порядком створення), ніколи не видаляючи той, на який вказує `current`;
- `rollback(app: Path) -> str`: повертає `current` на попередній реліз і повертає його ім'я; якщо попереднього немає, `RuntimeError`;
- повторний деплой з тим самим `name` це `FileExistsError` без змін у `current`.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

base = Path(tempfile.mkdtemp(prefix="lab41_"))
app = base / "app"; app.mkdir()
def make_src(v):
    s = base / f"src_{v}"; s.mkdir()
    (s / "version.txt").write_text(v)
    return s
srcs = {v: make_src(v) for v in ["v1", "v2", "v3", "v4"]}
```

**Вхідні дані / Інтерфейс:** функції вище; `current` це відносний симлінк `releases/<name>`.

**Очікуваний результат:** після `deploy` v1, v2, v3, v4 (keep=3) існують `v2, v3, v4`, `current/version.txt == "v4"`. `rollback` дає `"v3"`. Ще один `rollback` дає `"v2"`.

**Приклад:**
Input: чотири деплої, потім `rollback(app)`
Output: `"v3"`, `current/version.txt == "v3"`

**Edge Cases:**
- перший деплой (ще немає `current`)
- збій копіювання посеред деплою: `current` не змінено, тимчасова тека прибрана
- `keep = 1`

**Критерії приймання:**
- фоновий потік у циклі читає `current/version.txt` під час 200 деплоїв і жодного разу не отримує `FileNotFoundError`
- при збої копіювання не лишається напівготового релізу
- `current` завжди вказує на існуючий реліз
- після обрізання `current` не «висить» на видаленому релізі

---

### Task 42 — Детермінований дайджест дерева
**Складність:** Advanced

**Сценарій:** Треба порівнювати два каталоги на різних серверах одним рядком-відбитком, не пересилаючи файли.

**Умова:** Обчисли hex-дайджест дерева, що:
- залежить від відносних шляхів і вмісту файлів, але **не** від `mtime`, порядку обходу чи абсолютного розташування;
- розрізняє дерево з порожньою директорією та без неї;
- змінюється, якщо два файли поміняти вмістом (шлях прив'язаний до вмісту);
- не плутає `("ab","c")` і `("a","bc")` (однозначне кодування меж полів);
- читає файли блоками, пам'ять не залежить від розміру файлу.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

def build(base: Path, files: dict, dirs=()):
    for n, c in files.items():
        p = base / n
        p.parent.mkdir(parents=True, exist_ok=True)
        p.write_bytes(c)
    for n in dirs:
        (base / n).mkdir(parents=True, exist_ok=True)

d = Path(tempfile.mkdtemp(prefix="lab42_"))
t1, t2, t3 = d / "t1", d / "t2", d / "t3"
build(t1, {"a/x.txt": b"1", "b.txt": b"2"})
build(t2, {"b.txt": b"2", "a/x.txt": b"1"})            # інший порядок створення
os.utime(t2 / "b.txt", (1, 1))                          # інший mtime
build(t3, {"a/x.txt": b"1", "b.txt": b"2"}, dirs=["empty"])
```

**Вхідні дані / Інтерфейс:** `tree_digest(root: Path) -> str`

**Очікуваний результат:** `tree_digest(t1) == tree_digest(t2)`, `tree_digest(t1) != tree_digest(t3)`.

**Приклад:**
Input: `tree_digest(t1)`, `tree_digest(t2)`
Output: однакові рядки

**Edge Cases:**
- обмін вмістом між `a/x.txt` і `b.txt` змінює дайджест
- `{"ab": b"c"}` ≠ `{"a": b"bc"}`
- порожнє дерево має стабільний дайджест
- юнікодні та «дивні» імена

**Критерії приймання:**
- усі перелічені властивості виконуються (автоматизована перевірка на 6+ парах дерев)
- копія дерева в іншу теку має той самий дайджест
- файл на 200 МБ не піднімає пам'ять понад кілька МБ

---

### Task 43 — Заміна дублікатів жорсткими посиланнями
**Складність:** Advanced

**Сценарій:** Сховище роздулося від копій. Замість видалення треба зробити так, щоб дублікати ділили одне місце на диску, а шляхи залишилися робочими.

**Умова:** Для груп ідентичних файлів залиш перший за відсортованим шляхом, решту замін жорсткими посиланнями на нього. Вимоги:
- заміна атомарна (шлях ніколи не зникає);
- файли, що вже є посиланнями на один inode, пропускаються й не рахуються;
- файли з різними правами доступу не об'єднуються;
- якщо створити посилання неможливо (інша файлова система, помилка), файл лишається як був, а причина потрапляє у звіт;
- `dry_run` нічого не змінює.

Поверни `{"linked": int, "saved_bytes": int, "skipped": list[tuple[str, str]]}`.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab43_"))
(d / "sub").mkdir()
data = b"\x00" * 1_000_000
(d / "a.bin").write_bytes(data)
(d / "sub" / "b.bin").write_bytes(data)                # дублікат
(d / "c.bin").write_bytes(data); os.chmod(d / "c.bin", 0o755)   # інші права
os.link(d / "a.bin", d / "d.bin")                      # уже те саме inode
(d / "unique.bin").write_bytes(b"u")
```

**Вхідні дані / Інтерфейс:** `hardlink_duplicates(root: Path, dry_run: bool = False) -> dict`

**Очікуваний результат:** `linked == 1`, `saved_bytes == 1_000_000`; `a.bin`, `d.bin` і `sub/b.bin` мають однаковий `st_ino`, `st_nlink == 3`; `c.bin` окремо.

**Приклад:**
Input: `hardlink_duplicates(d)`
Output: `{"linked": 1, "saved_bytes": 1000000, "skipped": [...]}`

**Edge Cases:**
- порожні файли
- файл змінено між порівнянням і заміною (перевірка перед заміною)
- повторний запуск (`linked == 0`)

**Критерії приймання:**
- шлях ніколи не зникає (тест у фоновому потоці з `os.stat` у циклі)
- повторний запуск повертає `linked == 0`
- dry-run лишає всі `st_ino` без змін
- `c.bin` не об'єднано через права

---

### Task 44 — Кошик із відновленням
**Складність:** Advanced

**Сценарій:** «Видалити» має означати «перемістити в кошик із можливістю відновлення», як у файловому менеджері.

**Умова:** Реалізуй кошик у `root/.trash/`:
- `trash(root, rel) -> str`: переносить файл або директорію в кошик і повертає унікальний id; зберігає метадані (початковий шлях, час видалення) у файлі поруч;
- `list_trash(root) -> list[dict]`;
- `restore(root, id, on_conflict="error") -> Path`: повертає на початкове місце; якщо місце зайняте, `FileExistsError` або (при `on_conflict="rename"`) відновлення під новим ім'ям;
- `empty(root, older_than_days, now) -> int`: остаточно видаляє записи старші за поріг.

Шлях за межами `root` або всередині `.trash/` це `ValueError`. Два файли з однаковою назвою з різних місць не конфліктують у кошику. Метадані пишуться атомарно.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="lab44_"))
(root / "docs").mkdir()
(root / "docs" / "a.txt").write_text("A1")
(root / "b").mkdir()
(root / "b" / "a.txt").write_text("A2")
(root / "proj" / "src").mkdir(parents=True)
(root / "proj" / "src" / "m.py").write_text("print(1)")
```

**Вхідні дані / Інтерфейс:** функції вище.

**Очікуваний результат:** після `trash(docs/a.txt)` і `trash(b/a.txt)` у `list_trash` два різні id; `restore` кожного повертає свій вміст (`A1`/`A2`); `trash("proj")` переносить усю теку, `restore` відтворює `proj/src/m.py`.

**Приклад:**
Input: `i = trash(root, "docs/a.txt"); restore(root, i)`
Output: `docs/a.txt` знову існує з вмістом `A1`

**Edge Cases:**
- повторне видалення вже видаленого: `FileNotFoundError`
- відновлення, коли початкове місце зайняте
- `rel = "../x"` та `rel = ".trash/…"`

**Критерії приймання:**
- два файли з однаковим іменем не перезаписують один одного
- метадані не зіпсовані при обриві (атомарний запис)
- `empty` видаляє тільки прострочене
- директорії переносяться й відновлюються цілком

---

### Task 45 — Конкурентний запис у спільні файли **[POSIX]**
**Складність:** Advanced

**Сценарій:** Вісім worker-процесів дописують рядки в один журнал і збільшують спільний лічильник. Жоден рядок не можна загубити чи змішати з іншим.

**Умова:** Реалізуй:
- `append_line(path: Path, line: str) -> None`: безпечне дописування одного цілого рядка;
- `increment(path: Path) -> int`: атомарне читання-збільшення-запис числа у файлі, повертає нове значення;
- блокування має звільнятися автоматично, якщо процес було вбито;
- без сторонніх бібліотек.

**Підготовка середовища:**
```python
import multiprocessing as mp, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab45_"))
log, counter = d / "journal.log", d / "counter.txt"
counter.write_text("0")
N_PROC, N_ITER = 8, 500

def worker(i):
    for k in range(N_ITER):
        append_line(log, f"{i}:{k}")
        increment(counter)

# запуск:  with mp.Pool(N_PROC) as pool: pool.map(worker, range(N_PROC))
```

**Вхідні дані / Інтерфейс:** функції вище.

**Очікуваний результат:** у `journal.log` рівно 4000 рядків, усі унікальні й цілі; `counter.txt == "4000"`.

**Приклад:**
Input: запуск 8 процесів
Output: 4000 унікальних рядків формату `i:k`, лічильник `4000`

**Edge Cases:**
- процес убито під час утримання блокування (інші не зависають)
- порожній або відсутній `counter.txt`
- рядок довший за розмір буфера

**Критерії приймання:**
- повторити запуск 20 разів без жодної втрати
- після `kill -9` одного з worker-ів інші завершуються за розумний час
- у логу немає змішаних або обрізаних рядків

---

### Task 46 — Копіювання з можливістю продовження
**Складність:** Advanced

**Сценарій:** Копіювання 50 ГБ обірвалось на 70%. Повторний запуск має продовжити, а не починати спочатку.

**Умова:** Реалізуй `resumable_copy(src, dst, chunk=1 << 20, on_chunk=None)`:
- пише у `dst.part` і стан у `dst.part.state` (кількість підтверджених байтів);
- при повторному запуску перевіряє, що вже скопований префікс збігається з початком `src` (інакше починає заново), обрізає хвіст `.part` до межі підтвердженого блока й продовжує;
- по завершенні перевіряє розмір і хеш, атомарно перейменовує `.part` у `dst`, прибирає стан;
- `on_chunk(bytes_done)` викликається після кожного блока (для імітації збою в тестах).

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab46_"))
src = d / "big.bin"
with src.open("wb") as f:
    for i in range(50):
        f.write(bytes([i]) * (1 << 20))              # 50 МіБ
dst = d / "copy.bin"

class Crash(Exception): pass
def crash_at_40pct(done):
    if done >= 20 * (1 << 20):
        raise Crash
```

**Вхідні дані / Інтерфейс:** `resumable_copy(src: Path, dst: Path, chunk: int = 1 << 20, on_chunk=None) -> None`

**Очікуваний результат:** перший виклик із `crash_at_40pct` падає з `Crash`, `dst` не існує, `dst.part` існує. Другий виклик завершується; `dst` ідентичний `src`; `.part` і стан прибрані; на другому виклику з `src` прочитано менше 70% байтів.

**Приклад:**
Input: два виклики (перший обривається)
Output: `copy.bin` ідентичний `big.bin`

**Edge Cases:**
- `.part` довший за `src`
- префікс `.part` не збігається (джерело змінилося)
- порожній `src`
- `dst` уже існує

**Критерії приймання:**
- повторний запуск читає з `src` лише непідтверджену частину
- при зміні джерела копіювання починається спочатку
- у `dst` ніколи не з'являється напівготовий файл
- хеші збігаються

**Обмеження:** `shutil` заборонено. Пам'ять не залежить від розміру файлу.

---

### Task 47 — Стійкий обхід дерева **[POSIX]**
**Складність:** Advanced

**Сценарій:** Обхід дерева, де трапляються закриті теки, файли, що зникають посеред обходу, нескінченні симлінки та імена з неперевіреними байтами, має дійти до кінця й дати звіт.

**Умова:** Реалізуй `audit_tree(root, on_visit=None) -> Report`:
- не слідує симлінкам; рахує файли та байти;
- збирає помилки як пари `(шлях, ім'я_errno)` і продовжує;
- `on_visit(path)` викликається перед `stat` кожного файлу (для імітації зникнення файлу в тестах);
- імена з невалідним UTF-8 коректно відображаються (без винятків);
- код повернення: `0` помилок немає, `1` були нефатальні, `2` корінь недоступний.

**Підготовка середовища (POSIX, запускати не від root):**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab47_"))
(d / "ok").mkdir(); (d / "ok" / "f.txt").write_text("fine")
(d / "locked").mkdir(); (d / "locked" / "x.txt").write_text("x"); os.chmod(d / "locked", 0o000)
(d / "loop").mkdir(); os.symlink("..", d / "loop" / "up")
(d / "vanish.txt").write_text("v")
bad = os.fsencode(str(d)) + b"/bad\xff.txt"
open(bad, "wb").write(b"b")
(d / "line\nbreak.txt").write_text("n")

def on_visit(p):
    if p.name == "vanish.txt":
        p.unlink(missing_ok=True)       # файл «зникає» прямо перед stat
```

**Вхідні дані / Інтерфейс:** `audit_tree(root: Path, on_visit=None) -> Report` (`Report`: `files`, `bytes`, `errors`, `exit_code`).

**Очікуваний результат:** обхід завершується; у `errors` є `locked` (`EACCES`) і `vanish.txt` (`ENOENT`); `files` враховує `ok/f.txt`, `bad\xff.txt`, `line\nbreak.txt`; `exit_code == 1`.

**Приклад:**
Input: `audit_tree(d, on_visit)`
Output: звіт із 2 помилками, код `1`

**Edge Cases:**
- корінь не існує: код `2`
- цикл симлінків не викликає зависання
- порожня директорія

**Критерії приймання:**
- жодного необробленого винятку
- симлінк `loop/up` не слідується
- звіт друкується без `UnicodeEncodeError` (наприклад, через `surrogateescape`/`repr`)
- нефатальні помилки не зупиняють обхід

---

# LEVEL 5 — CHALLENGE (Task 48–55)

> Для всіх Challenge: самостійний CLI-інструмент `з нуля`, логування через `logging`, осмислені коди повернення, `--dry-run` де є зміни, тести (мінімум: успішний сценарій, помилка аргументів, граничні випадки, повторний запуск). Коди за замовчуванням: `0` успіх, `1` часткові помилки, `2` помилка аргументів, `3` небезпечна/недопустима операція. Працюй лише в тимчасових директоріях.

### Task 48 — `pyfind`: власний `find`
**Складність:** Challenge

**Сценарій:** Потрібна утиліта, що шукає файли за кількома критеріями та стрімить результат у пайп, навіть якщо дерево велике.

**Умова:** CLI `pyfind.py PATH [опції]`:
- `--name GLOB`, `--iname GLOB`, `--type f|d|l`, `--size +10K|-1M|500` (більше, менше, рівно; суфікси K, M, G), `--mtime -7|+30` (днів), `--maxdepth N`, `--empty`;
- `--print0` розділяє результати нульовим байтом;
- результат друкується потоково в міру знаходження, без збирання в список;
- закриття каналу (`| head -1`) завершує програму тихо, без traceback;
- недоступні теки дають повідомлення в `stderr`, обхід триває, код `1`;
- симлінки не слідуються.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="lab48_"))
for i in range(10_000):
    p = root / f"d{i % 100}" / f"f{i}.txt"
    p.parent.mkdir(exist_ok=True)
    p.write_bytes(b"x" * (i % 50))
(root / "d0" / "empty.dat").write_bytes(b"")
(root / "locked").mkdir(); os.chmod(root / "locked", 0o000)    # [POSIX]
```

**Вхідні дані / Інтерфейс:** `python pyfind.py PATH ...` (stdout: знайдені шляхи; stderr: повідомлення).

**Очікуваний результат:**
- `pyfind.py ROOT --name "f1*.txt" --type f --size +20` друкує лише відповідні файли
- `pyfind.py ROOT --empty --type f` знаходить порожні файли

**Приклад:**
Input: `python pyfind.py ROOT --name "*.dat" --type f`
Output: `ROOT/d0/empty.dat`

**Edge Cases:**
- `--maxdepth 0`
- невірний формат `--size`: код `2`
- шлях, що не існує: код `2`
- імена з пробілами та новими рядками (перевірка `--print0`)

**Критерії приймання:**
- `| head -1` не дає traceback
- пам'ять не росте з кількістю файлів
- недоступна тека: повідомлення та код `1`
- тести покривають кожен критерій фільтрації

**Обмеження:** `os.scandir`/`os.walk` дозволено, готові утиліти (`find`) заборонено. Без збирання всього списку в пам'ять.

---

### Task 49 — `pydu`: власний `du`
**Складність:** Challenge

**Сценарій:** Треба швидко побачити, що займає диск, з урахуванням жорстких посилань і фактичного зайнятого місця.

**Умова:** CLI `pydu.py PATH [опції]`:
- `-d N` глибина виводу, `--top N` лише N найбільших, `--human`, `--json`, `--exclude GLOB` (повторюваний), `--apparent` (видимий розмір замість зайнятого місця `st_blocks*512`), `-x` не виходити за межі файлової системи;
- жорсткі посилання на один inode рахуються **один раз**;
- симлінки не слідуються й не додаються до розміру цілі;
- результат сортується за розміром спадно; `--json` друкує структурований вивід;
- недоступні теки логуються, підрахунок триває.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="lab49_"))
(root / "a").mkdir(); (root / "b" / "deep").mkdir(parents=True)
(root / "a" / "big.bin").write_bytes(b"x" * 100_000)
os.link(root / "a" / "big.bin", root / "b" / "big_link.bin")
(root / "b" / "deep" / "small.bin").write_bytes(b"x" * 10)
os.symlink(root / "a", root / "b" / "sym_a")
(root / "cache").mkdir(); (root / "cache" / "tmp.bin").write_bytes(b"x" * 5000)
```

**Вхідні дані / Інтерфейс:** `python pydu.py PATH [-d N] [--top N] [--human] [--json] [--exclude GLOB] [--apparent] [-x]`

**Очікуваний результат:** `--apparent` для `ROOT`: сума `100_000 + 10 + 5000 = 105_010` (жорстке посилання враховано один раз, симлінк не додає нічого). З `--exclude cache` сума `100_010`.

**Приклад:**
Input: `python pydu.py ROOT --apparent --json`
Output: JSON з підсумком по `ROOT` = 105010 та по підтеках

**Edge Cases:**
- порожня директорія: `0`
- `-d 0` виводить лише корінь
- `PATH` це файл, а не директорія

**Критерії приймання:**
- hardlink і симлінк оброблено як описано
- `--exclude` виключає цілі піддерева
- `--json` валідний JSON
- тести не залежать від розміру блока ФС (використовують `--apparent`)

---

### Task 50 — `rsync-lite`: одностороння синхронізація
**Складність:** Challenge

**Сценарій:** Треба дзеркалити теку на резервний диск, швидко визначаючи зміни та безпечно видаляючи зайве.

**Умова:** CLI `rsync_lite.py SRC DST [опції]`:
- вміст `SRC` синхронізується в `DST` (створюється за потреби);
- швидка перевірка за розміром і `mtime`; `--checksum` змушує порівнювати хеші;
- копіювання атомарне по файлу (тимчасовий файл у `DST`, потім заміна), зберігає `mtime` і права; симлінки копіюються як симлінки;
- `--delete` видаляє з `DST` те, чого немає в `SRC` (лише з явним прапорцем), `--backup-dir D` замість видалення переносить у `D`;
- `--exclude GLOB` (повторюваний), `--dry-run`, `--itemize` (виводить зміну для кожного шляху: `+` нове, `>` оновлене, `-` видалене);
- відмовляється працювати (код `3`), якщо `DST` всередині `SRC` або навпаки, або `DST` це корінь файлової системи;
- SIGINT не лишає тимчасових файлів.

**Підготовка середовища:**
```python
import os, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab50_"))
src, dst = d / "src", d / "dst"
for n, c in {"a.txt": "A", "sub/b.txt": "B", "sub/deep/c.txt": "C", "skip.log": "L"}.items():
    p = src / n; p.parent.mkdir(parents=True, exist_ok=True); p.write_text(c)
os.symlink("a.txt", src / "link_to_a")
(dst / "sub").mkdir(parents=True)
(dst / "sub" / "b.txt").write_text("old-B")
(dst / "stale.txt").write_text("stale")
```

**Вхідні дані / Інтерфейс:** `python rsync_lite.py SRC DST [--delete] [--backup-dir D] [--exclude GLOB]... [--checksum] [--dry-run] [--itemize]`

**Очікуваний результат:** після `--delete --exclude "*.log"`: у `dst` є `a.txt`, `sub/b.txt` (оновлений), `sub/deep/c.txt`, `link_to_a` (симлінк), немає `skip.log` і `stale.txt`. Повторний запуск нічого не змінює.

**Приклад:**
Input: `python rsync_lite.py src dst --delete --itemize --exclude "*.log"`
Output: рядки `+ a.txt`, `> sub/b.txt`, `+ sub/deep/c.txt`, `+ link_to_a`, `- stale.txt`

**Edge Cases:**
- `DST` всередині `SRC`
- файл замінено директорією (та навпаки)
- порожній `SRC`
- недоступний файл: код `1`, решта копіюється

**Критерії приймання:**
- повторний запуск: нуль змін
- без `--delete` нічого не видаляється
- `--dry-run` не змінює `dst` та не створює тимчасових файлів
- SIGINT посеред копіювання не лишає `.tmp` у `dst`

---

## Progress Check #5 (після Task 50)

**Що ти вже маєш вміти:**
- атомарна підміна симлінка (тимчасовий лінк + `os.replace`), `os.link`, `st_ino`/`st_nlink`
- блокування файлів (`fcntl.flock`/lock-файл) з автоматичним звільненням при смерті процесу
- відновлюване копіювання: стан `.part`, перевірка префікса, `on_chunk` як точка впровадження збою
- стійкий обхід дерева: `os.scandir`, `errno`, `surrogateescape`
- CLI з `argparse`, що повертає осмислені коди (`0/1/2/3`) і пише повідомлення в `stderr`
- потокова обробка без накопичення списків; тиха обробка `BrokenPipeError`
- порівняння файлів у кілька етапів (розмір → частковий хеш → повний хеш)
- `signal.signal` для SIGINT/SIGTERM з прибиранням тимчасових файлів

**Типові інженерні помилки:**
- гонка «порівняли хеші → зробили посилання», а файл уже змінили
- `fsync` файлу без `fsync` каталогу: перейменування може не пережити збій живлення
- `chmod 000` не блокує root, тому тести на права падають у контейнері
- `print()` у кінці великого виводу без обробки закритого пайпа
- голий `except:` ловить `KeyboardInterrupt`, а тимчасові файли лишаються
- `--delete` як поведінка за замовчуванням
- порівняння `st_blocks`-розміру з «видимим» у тестах
- `os.walk` без `onerror`: недоступні теки мовчки пропускаються

**Контрольна задача (без коду):**
Твій `rsync-lite` запущено нічною задачею, а о 02:10 хтось запустив його вручну на ту саму пару директорій. Опиши словами: що може піти не так, як ти це виявиш і які два незалежні механізми (один на рівні процесу, другий на рівні файлів) захистять від пошкодження даних.

---

## Special Task — CODING WITHOUT AI #5

**Складність:** Advanced (не входить у лічильник 55)

**Сценарій:** Два співробітники змінювали копії однієї теки від спільної бази. Треба зліти зміни без втрат, а там, де обидва змінили один файл по-різному, зберегти обидві версії.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="special05_"))
def put(tree, rel, text):
    p = root / tree / rel
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(text)
for tree, files in {
    "base": {"same.txt": "s", "a_edits.txt": "base", "b_edits.txt": "base",
             "both_edit.txt": "base", "a_deletes.txt": "x", "conflict_del.txt": "base"},
    "A":    {"same.txt": "s", "a_edits.txt": "A-new", "b_edits.txt": "base",
             "both_edit.txt": "A-version", "new_a.txt": "from A", "conflict_del.txt": "A-edited"},
    "B":    {"same.txt": "s", "a_edits.txt": "base", "b_edits.txt": "B-new",
             "both_edit.txt": "B-version", "a_deletes.txt": "x", "new_b.txt": "from B"},
}.items():
    for rel, text in files.items():
        put(tree, rel, text)
```

**Умова:** Напиши інструмент триваймового злиття `merge BASE A B OUT`:
- зміна лише з одного боку береться;
- однакова зміна з обох боків береться один раз;
- різна зміна того ж файлу з обох боків зберігає обидві версії як `<ім'я>.conflict-A` і `<ім'я>.conflict-B`, з кодом повернення `1`;
- видалення з одного боку при незмінності на іншому застосовується;
- видалення з одного боку при зміні на іншому це конфлікт (залиш змінену версію, познач `.conflict-…`);
- нові файли з обох боків додаються, а однакові імена з різним вмістом це конфлікт;
- режим `--dry-run` друкує план;
- повторний запуск на тих самих вхідних даних дає той самий `OUT`.

**Очікуваний результат для прикладу:** `a_edits.txt == "A-new"`, `b_edits.txt == "B-new"`, `a_deletes.txt` відсутній, `new_a.txt` і `new_b.txt` присутні, `both_edit.txt.conflict-A/B`, `conflict_del.txt` залишено з позначкою конфлікту; код `1`.

**Критерії приймання:**
- жодна зміна не втрачена
- конфлікти явно позначені
- повторний запуск детермінований
- `OUT` не збігається з `BASE`, `A`, `B` і не лежить в них

**Обмеження:** без підказок, шаблонів і стартового коду.

---

### Task 51 — `logrotate-lite`
**Складність:** Challenge

**Сценарій:** Логи застосунків треба ротувати за розміром або віком, стискати старі копії та зберігати обмежену кількість.

**Умова:** CLI `rotate.py --config cfg.json [--state state.json] [--force] [--dry-run]`. Конфіг містить правила `{"path": "glob", "rotate": N, "max_size": bytes, "max_age_days": d, "compress": bool, "mode": "rename"|"copytruncate"}`:
- ротація, якщо файл перевищує `max_size` або минуло `max_age_days` від останньої ротації (за `state.json`), або задано `--force`;
- схема: `app.log` → `app.log.1` → `app.log.2.gz` …; старші за `rotate` видаляються;
- `rename`: переіменувати й створити новий порожній файл із тими ж правами; `copytruncate`: скопіювати та обнулити оригінал;
- стан записується атомарно; два одночасні екземпляри не працюють паралельно (lock) **[POSIX]**;
- порожній або відсутній файл не ротується;
- повторний запуск без росту файлу нічого не робить.

**Підготовка середовища:**
```python
import json, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab51_"))
logs = d / "logs"; logs.mkdir()
(logs / "app.log").write_text("line\n" * 5000)
(logs / "tiny.log").write_text("x\n")
(logs / "empty.log").write_text("")
cfg = {"rules": [{"path": str(logs / "*.log"), "rotate": 3, "max_size": 10_000,
                  "max_age_days": 7, "compress": True, "mode": "rename"}]}
(d / "cfg.json").write_text(json.dumps(cfg))
```

**Вхідні дані / Інтерфейс:** `python rotate.py --config cfg.json --state state.json [--force] [--dry-run]`

**Очікуваний результат:** `app.log` (25 000 байт > 10 000) ротується: з'являється `app.log.1` (незжатий), новий порожній `app.log`; `tiny.log` і `empty.log` не ротуються. Після ще кількох ротацій: `app.log.1`, `app.log.2.gz`, `app.log.3.gz`, далі видалення.

**Приклад:**
Input: `python rotate.py --config cfg.json --state state.json`
Output: `rotated app.log -> app.log.1`

**Edge Cases:**
- відсутній `state.json`
- зламаний JSON конфігу: код `2`
- файл зростає під час ротації
- glob, що нічого не знаходить

**Критерії приймання:**
- без `--force` і без росту файлу повторний запуск нічого не робить
- кількість копій не перевищує `rotate`
- сумарна кількість рядків у всіх копіях і поточному файлі дорівнює початковій (для режиму `rename`)
- `--dry-run` нічого не змінює

---

### Task 52 — `fdupes-lite`
**Складність:** Challenge

**Сценарій:** Потрібен швидкий пошук дублікатів у каталозі з десятками тисяч файлів, з можливістю безпечно їх прибрати.

**Умова:** CLI `fdupes_lite.py PATH... [опції]`:
- конвеєр відбору: розмір → хеш перших і останніх 4 КіБ → повний хеш; кожен етап читає лише те, що потрібно;
- `--min-size N` (за замовчуванням 1), `--json`, прогрес у `stderr`;
- за замовчуванням лише звіт. Дії: `--hardlink` (заміна жорсткими посиланнями) або `--delete --keep first|newest|oldest`, і **лише** з `--apply`; без `--apply` це dry-run;
- симлінки ігноруються; файли з одним `inode` це не дублікати один одному;
- звіт містить кількість прочитаних байтів і «зекономлених» байтів;
- код повернення `0` без дублікатів чи з успішною дією, `1` якщо були помилки, `2` помилка аргументів.

**Підготовка середовища:**
```python
import os, random, tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab52_"))
random.seed(1)
for i in range(5000):
    (d / f"u{i}.bin").write_bytes(os.urandom(1000 + i))     # унікальні розміри
(d / "dup1.bin").write_bytes(b"A" * 100_000)
(d / "dup2.bin").write_bytes(b"A" * 100_000)
(d / "dup3.bin").write_bytes(b"A" * 100_000)
(d / "near.bin").write_bytes(b"A" * 99_999 + b"B")         # той самий розмір, інший кінець
os.link(d / "dup1.bin", d / "dup1_link.bin")
```

**Вхідні дані / Інтерфейс:** `python fdupes_lite.py PATH... [--min-size N] [--json] [--hardlink | --delete --keep first|newest|oldest] [--apply]`

**Очікуваний результат:** одна група `dup1.bin`, `dup2.bin`, `dup3.bin` (`dup1_link.bin` згадується як жорстке посилання на `dup1.bin`, але не як окремий дублікат); `near.bin` не в групі. Загалом прочитано менше за 1 МБ (а не десятки мегабайтів).

**Приклад:**
Input: `python fdupes_lite.py ROOT --json`
Output: JSON із групою з 3 файлів і полем `bytes_read`

**Edge Cases:**
- порожні файли
- два шляхи до одного файлу (через симлінк)
- `--delete` без `--apply`: нічого не видаляється
- недоступний файл: код `1`, обробка триває

**Критерії приймання:**
- `bytes_read` у звіті відповідає очікуваному порядку
- `near.bin` не визнано дублікатом
- без `--apply` дерево не змінюється
- тести покривають усі три етапи відбору

---

### Task 53 — `snap`: знімки з контентно-адресованим сховищем
**Складність:** Challenge

**Сценарій:** Потрібен міні-аналог restic: інкрементальні знімки дерева, де однакові файли зберігаються лише один раз, а цілісність можна перевірити.

**Умова:** CLI `snap.py` з підкомандами:
- `init REPO`, `backup REPO SRC [--tag T]`, `list REPO`, `restore REPO SNAP_ID DEST [--path SUB]`, `verify REPO`, `prune REPO --keep-last N [--dry-run]`;
- об'єкти (файли) зберігаються за SHA-256 у `REPO/objects/ab/cdef…`; хеш рахується під час запису в тимчасовий файл, потім перейменування на підсумкове ім'я;
- знімок це JSON-дерево (шлях, режим, `mtime`, розмір, id об'єкта), яке з'являється в `REPO/snapshots/` **лише після** збереження всіх об'єктів;
- повторний `backup` без змін у `SRC` не зберігає нових об'єктів (0 нових байтів);
- `verify` перевіряє, що кожен об'єкт, на який посилаються знімки, існує й відповідає своєму хешу (код `1` при пошкодженні);
- `prune` залишає N останніх знімків і видаляє непотрібні об'єкти (mark & sweep), `--dry-run` лише показує;
- `restore` відмовляється від шляхів у знімку, що виходять за межі `DEST` (код `3`);
- файли до ~1 ГБ обробляються блоками, пам'ять обмежена.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab53_"))
src = d / "src"; (src / "sub").mkdir(parents=True)
(src / "a.txt").write_text("alpha")
(src / "copy_of_a.txt").write_text("alpha")          # той самий вміст
(src / "sub" / "b.bin").write_bytes(b"\x00" * 3_000_000)
repo, dest = d / "repo", d / "restore"
```

**Вхідні дані / Інтерфейс:** `python snap.py <підкоманда> ...`

**Очікуваний результат:**
- після `backup` у репозиторії 2 унікальні об'єкти (вміст `alpha` збережено один раз)
- другий `backup` без змін: 0 нових об'єктів
- `restore` відтворює дерево з тими ж вмістом та `mtime`
- зіпсований об'єкт: `verify` повертає `1`

**Приклад:**
Input: `python snap.py backup repo src`
Output: `snapshot 20261001-… saved: 3 files, 2 new objects`

**Edge Cases:**
- порожній `SRC`
- знімок без об'єктів
- обрив `backup` посеред (не має залишити знімка)
- зіпсований JSON знімка

**Критерії приймання:**
- дедуплікація працює між файлами та між знімками
- обірваний `backup` не лишає напівготового знімка
- `prune` не видаляє об'єкти, потрібні залишеним знімкам
- `restore` ніколи не пише поза `DEST`

---

### Task 54 — `fswatch-lite`: спостерігач за директорією
**Складність:** Challenge

**Сценарій:** Треба запускати дію при появі чи зміні файлів у теці без сторонніх бібліотек і без системних нотифікацій.

**Умова:** CLI `fswatch_lite.py DIR [опції]`:
- опитування з інтервалом `--interval SEC`; події `created`, `modified`, `deleted` (та `moved`, якщо той самий inode змінив ім'я, як stretch) у вигляді JSON-рядків у `stdout`;
- початковий знімок мовчазний (`--emit-initial` його друкує);
- `--include`/`--exclude` glob-фільтри; `--debounce SEC`: файл, що змінюється безперервно, дає **одну** подію `modified` після паузи;
- обхід швидкий: `scandir` із використанням кешованого `stat`;
- `SIGINT`/`SIGTERM` **[POSIX]** завершують програму коректно, скинувши відкладені події, код `0`; `BrokenPipeError` на `stdout` завершує тихо;
- зникнення кореня `DIR`: подія й код `3`;
- `--cycles N` виконує N опитувань і завершується (для тестів).

**Підготовка середовища:**
```python
import tempfile, threading, time
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab54_"))
(d / "watched").mkdir()
(d / "watched" / "existing.txt").write_text("e")

def scenario():
    time.sleep(0.3); (d / "watched" / "new.txt").write_text("n")
    time.sleep(0.3); (d / "watched" / "existing.txt").write_text("changed")
    time.sleep(0.3); (d / "watched" / "new.txt").unlink()
threading.Thread(target=scenario, daemon=True).start()
```

**Вхідні дані / Інтерфейс:** `python fswatch_lite.py DIR [--interval S] [--include G] [--exclude G] [--debounce S] [--emit-initial] [--cycles N]`

**Очікуваний результат:** три JSON-події: `created new.txt`, `modified existing.txt`, `deleted new.txt` (у цьому порядку). Початкового `existing.txt` без `--emit-initial` немає.

**Приклад:**
Input: `python fswatch_lite.py watched --interval 0.1 --cycles 15`
Output: `{"event":"created","path":"new.txt"}` тощо

**Edge Cases:**
- швидкі зміни одного файлу підряд (debounce)
- файл створено та видалено між двома опитуваннями
- `DIR` не існує на старті: код `2`
- `--exclude "*.tmp"`

**Критерії приймання:**
- кожна подія друкується один раз
- debounce зводить серію змін до однієї події
- `SIGTERM` завершує з кодом `0` без traceback
- опитування 50 000 файлів не читає їх вміст

---

### Task 55 — `mmv`: транзакційне масове перейменування
**Складність:** Challenge

**Сценарій:** Масове перейменування, де `a → b` і `b → a` (обмін), ланцюжки та колізії мають оброблятися коректно, а збій посередині не має лишати хаос.

**Умова:** CLI `mmv.py 'SRC_PATTERN' 'DST_TEMPLATE' [--dry-run] [--force] [--log FILE]` та `mmv.py --undo LOGFILE`:
- шаблон джерела це glob із зірочками; їхні збіги підставляються в шаблон призначення як `#1`, `#2`…;
- спершу будується повний **план**, і лише після перевірки виконується:
  - два джерела з однаковою ціллю: помилка, зміни не виконуються;
  - ціль існує й не є частиною плану: помилка, якщо немає `--force`;
  - обмін і цикли (`a→b, b→a`, `a→b→c→a`) виконуються коректно, ланцюжки (`a→b, b→c`) у правильному порядку;
  - ціль поза робочою текою (`../x`) заборонена (код `3`);
- кожна операція журналюється у `LOG` (JSON-рядок, скидається на диск **до** виконання операції);
- збій посередині автоматично відкочує вже виконане;
- `--undo LOGFILE` повністю скасовує попереднє перейменування;
- для тестів підтримай змінну оточення `MMV_FAIL_AT=k`, що імітує збій на k-й операції.

**Підготовка середовища:**
```python
import tempfile
from pathlib import Path

d = Path(tempfile.mkdtemp(prefix="lab55_"))
for n in ["a.txt", "b.txt", "c.txt", "IMG_1.JPG", "IMG_2.JPG"]:
    (d / n).write_text(n)
```

**Вхідні дані / Інтерфейс:** `python mmv.py 'PATTERN' 'TEMPLATE' [--dry-run] [--force] [--log FILE]` та `python mmv.py --undo LOGFILE`

**Очікуваний результат:**
- `mmv 'IMG_*.JPG' 'photo_#1.jpg'` дає `photo_1.jpg`, `photo_2.jpg`
- обмін `a.txt ↔ b.txt` реалізується двома викликами/спеціальним планом без втрати вмісту
- збій на другій операції повертає дерево до початкового стану

**Приклад:**
Input: `python mmv.py 'IMG_*.JPG' 'photo_#1.jpg' --log undo.log`
Output: `IMG_1.JPG -> photo_1.jpg`, `IMG_2.JPG -> photo_2.jpg`

**Edge Cases:**
- шаблон не знайшов файлів: код `0` з повідомленням
- у шаблоні призначення більше `#N`, ніж зірочок у джерелі: код `2`
- регістро-незалежна файлова система (перейменування лише регістру)
- порожнє ім'я призначення

**Критерії приймання:**
- обмін і цикл з 3 елементів коректні (вміст не втрачено)
- при `MMV_FAIL_AT=2` дерево повертається до початкового стану
- `--undo` повертає все назад, повторний `--undo` безпечний
- конфлікти виявляються до будь-яких змін

---

## Progress Check #6 (фінальний, після Task 55)

**Що ти маєш уміти після теми:**
- будь-яку операцію з файлами виконувати атомарно, ідемпотентно і з dry-run
- обирати між `rename`, `replace`, `copy2`, `link` з розумінням обмежень файлових систем
- тримати пам'ять сталою на файлах довільного розміру
- захищатися від path traversal, zip-slip, гонок TOCTOU, симлінк-атак
- будувати CLI-інструменти з кодами повернення, логуванням і коректною реакцією на сигнали
- тестувати файлові операції на `tmp_path` без залежності від часу, розміру блока ФС та порядку обходу

**Типові інженерні помилки (підсумок):**
- недетермінований порядок обходу в тестах і архівах
- видалення до підтвердження того, що копія цілісна
- відсутність `fsync` перед `os.replace` там, де важлива переживаність збою
- довіра до даних архіву, імен файлів і метаданих з джерела
- мовчазне ковтання помилок: дерево «оброблено», а половина пропущена
- `--delete`/`--force` за замовчуванням
- тести, що залежать від поточного часу, `umask` або прав root

**Контрольна задача (без коду):**
Спроєктуй словами сервіс «щоденна синхронізація 5 ТБ між двома дата-центрами». Опиши: спосіб виявлення змін без читання всього вмісту, як обробляти файли, що змінюються під час копіювання, як перевірити цілісність, як продовжити після обриву зв'язку та які метрики й коди помилок ти виведеш оператору.

---

*Далі за планом: тема 2 — Text & Regex Automation (Task 1–55).*
