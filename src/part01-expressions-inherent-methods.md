## Part 01: Expressions (นิพจน์) และ Inherent Methods

แม้โค้ดจากส่วนแรกจะทำงานได้ดีอยู่แล้ว แต่เรายังเรียนรู้อะไรได้อีกเยอะจากการปรับโค้ดให้สวยงามขึ้น เหตุผลก็คือ Rust เป็นภาษาแบบ **"expression-based"** (เน้นนิพจน์) ซึ่งหมายความว่าสิ่งที่คุณเขียนลงไปส่วนใหญ่ไม่ใช่แค่คำสั่ง (ที่สั่งให้โค้ดทำงาน) แต่เป็นนิพจน์ (ที่มีการคืนค่าข้อมูล) หลักการนี้ใช้ได้แม้กระทั่งกับเนื้อหาภายในฟังก์ชันทั้งก้อนเลยทีเดียว

> **หมายเหตุ:** ในบทนี้เรายังใช้ `NumberOrNothing` ที่สร้างเองจากบทที่แล้ว เพื่อให้เห็นภาพการทำงานชัดเจน แต่ในโค้ดจริงควรใช้ `Option<i32>` พร้อม method `unwrap_or(default)` แทนการสร้าง enum เอง

## การเขียนโปรแกรมแบบ Expression-based

ลองดูตัวอย่าง `sqr` 

```rust
fn sqr(i: i32) -> i32 { i * i }
```

สิ่งที่เราใส่ไว้ในวงเล็บปีกกาคือนิพจน์ที่ใช้คำนวณค่าที่จะส่งคืน เราจึงเขียนแค่ `i * i` ซึ่งเป็นนิพจน์สำหรับคืนค่ากำลังสองของ `i` ได้เลย วิธีนี้คล้ายกับเวลาที่นักคณิตศาสตร์เขียนฟังก์ชันมาก (เพียงแต่ต้องระบุ Type เพิ่มเติม)

> **ข้อสังเกตสำคัญ** ใน Rust นิพจน์จะ **ไม่มี** เครื่องหมายเซมิโคลอน (`;`) ปิดท้าย ในขณะที่คำสั่ง (statement) จะมีเครื่องหมายเซมิโคลอนเป็นตัวระบุว่า *"นี่คือคำสั่ง ไม่ใช่นิพจน์ที่จะคืนค่า"* ดังนั้น เมื่อคุณต้องการให้บล็อกโค้ดหรือฟังก์ชันคืนค่าอะไรออกไป คุณจะทิ้งนิพจน์สุดท้ายไว้โดยไม่มีเซมิโคลอน

คำสั่งเงื่อนไขก็ถือเป็นนิพจน์เช่นกัน ซึ่งเทียบได้กับตัวดำเนินการ Ternary `? :` ในภาษา C

```rust
fn abs(i: i32) -> i32 { if i >= 0 { i } else { -i } }
```

หลักการเดียวกันนี้ใช้กับการแยกกรณีด้วย `match` ด้วยเช่นกัน โดยทุกๆ Arm ของ `match` จะระบุนิพจน์ที่ต้องการคืนค่าในกรณีนั้นๆ

ลองดูตัวอย่างจากบทที่แล้ว แต่เขียนให้กระชับขึ้น:

```rust
enum NumberOrNothing {
    Number(i32),
    Nothing
}
use self::NumberOrNothing::{Number,Nothing};

fn number_or_default(n: NumberOrNothing, default: i32) -> i32 {
    match n {
        Nothing => default,
        Number(n) => n,
    }
}
```

แม้แต่บล็อกโค้ดก็ยังนับเป็นนิพจน์ โดยค่าของมันจะเท่ากับนิพจน์ตัวสุดท้ายที่อยู่ข้างใน

```rust
fn compute_stuff(x: i32) -> i32 {
    let y = { let z = x*x; z + 14 }; // บล็อก { ... } เป็นนิพจน์ที่คืนค่า (z + 14)
    y*y
}
```

## รีแฟกเตอร์ vec_min ให้กระชับขึ้น

ทีนี้ เรามาลองรีแฟกเตอร์ `vec_min` จากบทที่แล้วกันบ้าง

```rust
fn vec_min(v: Vec<i32>) -> NumberOrNothing {
```

ยังจำฟังก์ชันช่วยงาน `min_i32` ได้ไหม? ใน Rust เราสามารถนิยามฟังก์ชันช่วยงานแบบนี้ไว้ "ข้างใน" ฟังก์ชันอื่นได้เลย นี่เป็นแค่เรื่องของ namespacing เท่านั้น โดยฟังก์ชันภายในจะเข้าถึงข้อมูลของฟังก์ชันภายนอกไม่ได้ แต่นั่นก็ช่วยให้เราจัดกลุ่มฟังก์ชันได้เป็นระเบียบ และทำให้อ่านโค้ดได้ง่ายขึ้นมาก
 
```rust
    fn min_i32(a: i32, b: i32) -> i32 {
        if a < b { a } else { b }
    }

    let mut min = Nothing;
    for e in v {
```

สังเกตว่า สิ่งที่เราทำตรงนี้คือการคำนวณค่าใหม่ให้ `min` ซึ่งผลลัพธ์สุดท้ายจะเป็น `Number` เสมอ ไม่ใช่ `Nothing` อีกต่อไป ในภาษา Rust โครงสร้างของโค้ดสามารถสื่อถึงความสม่ำเสมอนี้ออกมาได้ชัดเจน โดยใช้ `match` เป็นนิพจน์:

```rust
        // match เป็น expression: คืนค่าออกมาได้
        let new_min_val = match min {
            Nothing => e,           // ถ้ายังไม่มีค่า ใช้ค่าแรกที่เจอ
            Number(n) => min_i32(n, e), // ถ้ามีแล้ว เอาค่าต่ำสุด
        };
        min = Number(new_min_val);
    }
```

> **💡 ทำไมไม่เขียนสั้นกว่านี้?** เราสามารถเขียนเป็น `min = Number(match min { ... })` ได้ แต่การแยกเป็นสองบรรทัดทำให้โค้ดอ่านง่ายขึ้น โดยเฉพาะสำหรับผู้เรียนใหม่ คุณจะเห็นชัดว่า `match` คืนค่า `i32` ออกมา แล้วเรานำไปห่อด้วย `Number()` ในขั้นตอนถัดไป
    
Rust มีคีย์เวิร์ด `return` ให้ใช้ก็จริง แต่ไม่ค่อยนิยมใช้กันเท่าไหร่ ปกติเราจะอาศัยหลักการที่ว่า ตัวฟังก์ชันทั้งก้อนถือเป็นนิพจน์หนึ่งตัว ดังนั้นเราจึงเขียนค่าที่ต้องการส่งคืนลงไปตรงๆ ได้เลย

```rust
    min // นี่คือนิพจน์สุดท้ายของฟังก์ชัน vec_min ซึ่งจะถูกคืนค่า
}
```

โค้ดสั้นลงกว่าเดิมเยอะเลย ลองไล่ดูโค้ดด้านบนอีกที และตรวจสอบให้แน่ใจว่าคุณเข้าใจการทำงานทุกขั้นตอนจริงๆ

## Inherent implementations

พอแค่นี้สำหรับ `vec_min` เรากลับมาดู `print_number_or_nothing` กันต่อ จริงๆ แล้วฟังก์ชันนี้ควรจะอยู่ใกล้ๆ กับไทป์ `NumberOrNothing` ถ้าเป็น C++ หรือ Java คุณคงสร้างมันเป็นเมธอดของไทป์นั้นไปแล้ว ใน Rust เราก็ทำแบบนั้นได้เหมือนกันโดยการสร้าง **inherent implementation**

```rust
impl NumberOrNothing {
    fn print(self) {
        match self {
            Nothing => println!("The number is: <nothing>"),
            Number(n) => println!("The number is: {}", n),
        };
    }
}
```

เกิดอะไรขึ้นตรงนี้? เนื่องจาก Rust แยกโค้ดออกจากข้อมูล การนิยามเมธอดให้กับ `enum` (และรวมถึง `struct` ที่เราจะเรียนกันทีหลัง) จึงแยกส่วนออกจากการนิยามไทป์

สำหรับ `self` นั้น มันเป็นตัวแทนของอินสแตนซ์ของ `NumberOrNothing` ที่เรากำลังเรียกเมธอด `print` อยู่ โดย Rust มีวิธีการจัดการ `self` ได้หลายแบบ (เช่น `self`, `&self`, `&mut self`) ซึ่งเกี่ยวข้องกับการจัดการความเป็นเจ้าของ (ownership) และการยืม (borrowing) ข้อมูล แต่ในกรณีนี้ `self` หมายถึงการรับค่า `NumberOrNothing` เข้ามาแบบเต็ม ๆ (โดยการย้ายความเป็นเจ้าของ) เพื่อนำมาใช้งานภายในเมธอด

> **⚠️ ทำไมใช้ `self` ไม่ใช่ `&self`?** 
> 
> ในกรณีนี้เราใช้ `self` (แบบเต็ม) เพราะ `NumberOrNothing` เป็นข้อมูลขนาดเล็ก (แค่ตัวเลขหนึ่งตัวหรือว่างเปล่า) แต่ถ้าเป็นข้อมูลใหญ่ๆ ควรใช้ `&self` (borrow) เพื่อไม่ให้ย้ายข้อมูลทั้งก้อน เราจะเรียนรู้เรื่อง Ownership และ Borrowing อย่างละเอียดในบทต่อไป

ในเมธอด `print` ของ `impl NumberOrNothing` ตัวแปร `self` จะมี Type เป็น `NumberOrNothing` เสมอ ดังนั้นตอนนี้ `print` จึงเป็นเมธอดที่รับ `NumberOrNothing` เป็นอาร์กิวเมนต์ตัวแรก เหมือนกับ `print_number_or_nothing` นั่นเอง เวลาคุณเรียก `min.print()` จริงๆ แล้วมันก็คือการเรียกฟังก์ชันที่รับ `min` เป็นอาร์กิวเมนต์ตัวแรก คล้ายกับ `NumberOrNothing::print(min)` นั่นเอง

### รีแฟกเตอร์ number_or_default เป็น Method

ตอนนี้ลองเปลี่ยน `number_or_default` จากข้างบนให้เป็น method ของ `NumberOrNothing` ดู:

```rust
impl NumberOrNothing {
    // ฟังก์ชันเดิม: number_or_default(my_num, 0)
    // หลังรีแฟกเตอร์: my_num.or_default(0)
    fn or_default(self, default: i32) -> i32 {
        match self {
            Nothing => default,
            Number(n) => n,
        }
    }
}
```

> **ลองทำ:** สังเกตว่าการเปลี่ยนจากฟังก์ชันเป็น method ทำให้โค้ดอ่านเป็นธรรมชาติมากขึ้น `my_num.or_default(0)` อ่านว่า "my_num หรือค่าเริ่มต้น 0" ในขณะที่ `number_or_default(my_num, 0)` อ่านยากกว่า

หลังจากรีแฟกเตอร์ฟังก์ชันและเมธอดเรียบร้อยแล้ว หน้าตาของ `main` จะเป็นแบบนี้

```rust
fn read_vec() -> Vec<i32> {
    vec![18, 5, 7, 2, 9, 27]
}

pub fn main() {
    let vec = read_vec();
    let min = vec_min(vec);
    min.print(); // เรียกเมธอด print() ของ NumberOrNothing โดยตรง
    
    // ทดสอบ or_default
    println!("With default 0: {}", Number(42).or_default(0));
    println!("With default 0: {}", Nothing.or_default(0));
}
```

เอาล่ะ ทีนี้เรามาดูโค้ดรวมทั้งหมดเพื่อให้เห็นภาพการเชื่อมโยงสิ่งที่เราได้เรียนรู้ไปทั้งหมด ก็จะได้หน้าตาโค้ดประมาณนี้

```rust
enum NumberOrNothing {
    Number(i32),
    Nothing,
}

use self::NumberOrNothing::{Nothing, Number};

impl NumberOrNothing {
    fn print(self) {
        match self {
            Nothing => println!("The number is: <nothing>"),
            Number(n) => println!("The number is: {}", n),
        };
    }

    fn or_default(self, default: i32) -> i32 {
        match self {
            Nothing => default,
            Number(n) => n,
        }
    }
}

fn vec_min(v: Vec<i32>) -> NumberOrNothing {
    // ฟังก์ชันภายใน (local function) - ใช้ได้แค่ในฟังก์ชันนี้
    fn min_i32(a: i32, b: i32) -> i32 {
        if a < b { a } else { b }
    }

    let mut min = Nothing;
    for e in v {
        // match เป็น expression: คืนค่าออกมาได้
        let new_min_val = match min {
            Nothing => e,           // ถ้ายังไม่มีค่า ใช้ค่าแรกที่เจอ
            Number(n) => min_i32(n, e), // ถ้ามีแล้ว เอาค่าต่ำสุด
        };
        min = Number(new_min_val);
    }

    min  // ส่งคืนโดยไม่ต้องใช้ return
}

fn read_vec() -> Vec<i32> {
    vec![18, 5, 7, 2, 9, 27]
}

pub fn main() {
    let vec = read_vec();
    let min = vec_min(vec);
    min.print();

    println!("With default 0: {}", Number(42).or_default(0));
    println!("With default 0: {}", Nothing.or_default(0));
}
```

## เชื่อมโยงกับโลกจริง: Option<T>

ในบทที่แล้วเราได้รู้ว่า `NumberOrNothing` ที่เราสร้างขึ้นนั้น เทียบเท่ากับ `Option<i32>` ที่ Rust มีให้ใช้อยู่แล้ว ในโค้ดจริง เราสามารถใช้ method ของ `Option<T>` แทนการเขียนเองได้ดังนี้:

```rust
// แทนที่จะสร้าง NumberOrNothing และ or_default เอง
// ใช้ Option<i32> และ unwrap_or แทน

fn vec_min_real(v: Vec<i32>) -> Option<i32> {
    let mut min = None;
    for e in v {
        min = Some(match min {
            None => e,
            Some(n) => if n < e { n } else { e },
        });
    }
    min
}

// การใช้งาน
let min = vec_min_real(vec);
println!("{}", min.unwrap_or(0)); // คืนค่าใน Some(...) หรือ 0 ถ้าเป็น None
```

`Option<T>` มี methods ที่เป็นประโยชน์มากมาย เช่น:
- `unwrap_or(default)` - คืนค่าหรือค่าเริ่มต้น
- `map(f)` - แปลงค่าภายใน
- `is_some()` / `is_none()` - ตรวจสอบว่ามีค่าหรือไม่

เราจะได้เรียนรู้ methods เหล่านี้เพิ่มเติมในบทต่อไป

## แบบฝึกหัด

**แบบฝึกหัด 01.1:** จงเขียนฟังก์ชัน `vec_sum` เพื่อคำนวณผลรวมของค่าทั้งหมดใน `Vec<i32>` โดยคืนค่าเป็น `i32` (ถ้าเวกเตอร์ว่างให้คืน 0)

<details>
<summary>เฉลย</summary>

```rust
fn vec_sum(v: Vec<i32>) -> i32 {
    let mut sum = 0;
    for e in v {
        sum = sum + e; // หรือ sum += e;
    }
    sum
}

// แบบใช้ iterator (กระชับกว่า)
fn vec_sum_iter(v: Vec<i32>) -> i32 {
    v.iter().sum()
}
```
</details>

**แบบฝึกหัด 01.2:** จงเขียนฟังก์ชัน `vec_print` ที่รับค่าเวกเตอร์ แล้วพิมพ์สมาชิกทั้งหมดออกมา โดยไม่คืนค่า (คืน `()` หรือ unit type)

<details>
<summary>เฉลย</summary>

```rust
fn vec_print(v: Vec<i32>) {
    for e in v {
        println!("{}", e);
    }
}

// หรือแบบสั้น
fn vec_print_short(v: Vec<i32>) {
    v.iter().for_each(|e| println!("{}", e));
}
```
</details>

**แบบฝึกหัด 01.3 (ท้าทาย):** จงเพิ่ม method `is_nothing()` ให้กับ `NumberOrNothing` ที่คืนค่า `true` ถ้าเป็น `Nothing` และ `false` ถ้าเป็น `Number`

<details>
<summary>เฉลย</summary>

```rust
impl NumberOrNothing {
    fn is_nothing(self) -> bool {
        match self {
            Nothing => true,
            Number(_) => false,
        }
    }
    
    // หรือแบบย่อ
    fn is_nothing_short(self) -> bool {
        matches!(self, Nothing)
    }
}
```
</details>
