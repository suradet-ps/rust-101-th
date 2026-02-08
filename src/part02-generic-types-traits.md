## Part 02: Generic Types และ Traits

ลองย้อนกลับมาดู type `NumberOrNothing` กันอีกสักนิด ไม่รู้สึกขัดใจบ้างหรือที่เราต้องฮาร์ดโค้ด type `i32` ลงไปตรงๆ? แล้วถ้าพรุ่งนี้เราอยากได้ `CharOrNothing` และวันถัดไปอยากได้ `FloatOrNothing` ล่ะ? แน่นอนว่าเราคงไม่อยากเขียน type และเมธอดประจำตัวของมันขึ้นมาใหม่ทั้งหมดแน่ๆ

ในบทนี้ เราจะทำให้ `NumberOrNothing` ที่สร้างเองจากบทก่อนหน้า กลายเป็น **Generic Type** ที่ใช้ได้กับข้อมูลหลายชนิด พร้อมเรียนรู้ระบบ **Traits** ซึ่งเป็นรากฐานสำคัญของการเขียนโค้ดที่ยืดหยุ่นใน Rust

> **💡 ความรู้เพิ่มเติม:** จริงๆ แล้ว `Option<T>` เป็น type ที่มีอยู่แล้วใน standard library โดยไม่ต้อง `use` ก็ใช้ได้ (อยู่ใน prelude) พร้อมด้วยตัวแปร `Some` และ `None` แต่เราจะสร้างเองเพื่อเข้าใจหลักการ

## Generic Datatypes

ทางออกของปัญหานี้เรียกว่า *Generics* หรือ *Polymorphism* (คำหลังเป็นภาษากรีก แปลว่า "หลายรูปทรง") คุณอาจเคยเห็นอะไรคล้ายๆ กันนี้มาแล้วใน C++ (ที่เรียกว่า *Template*) หรือใน Java รวมถึงภาษาตระกูล Functional languages อื่นๆ

เอาล่ะ ในที่นี้เราจะนิยาม Generic type ที่ชื่อ `SomethingOrNothing` กัน

```rust
pub enum SomethingOrNothing<T> {
    Something(T),
    Nothing,
}
```

`<T>` ที่ต่อท้ายชื่อ type คือ **Type Parameter** หรือ "ตัวแปรชนิดข้อมูล" ซึ่งหมายความว่า `T` สามารถแทนที่ได้ด้วย type ใดก็ได้ เช่น `i32`, `f32`, `bool` หรือแม้แต่ type ที่ซับซ้อนกว่านั้น

แทนที่จะต้องคอยเขียนชื่อเต็มของรูปแบบ (Variant) ทุกครั้ง เราสามารถ Import ทั้งหมดเข้ามาทีเดียวได้เลย

```rust
pub use self::SomethingOrNothing::*;
```

สิ่งที่ทำตรงนี้คือการนิยามตระกูลของ type ขึ้นมาทั้งชุด ตอนนี้เราสามารถเขียน `SomethingOrNothing<i32>` เพื่อให้ได้ `NumberOrNothing` แบบเดิมกลับมา

```rust
type NumberOrNothing = SomethingOrNothing<i32>;
```

คีย์เวิร์ด `type` ใช้สร้าง **Type Alias** (ชื่อแทน) ซึ่งไม่ได้สร้าง type ใหม่ แค่สร้างชื่อเล่นให้ type ที่มีอยู่แล้วเท่านั้น

อย่างไรก็ตาม เรายังสามารถใช้ `SomethingOrNothing` กับ type อื่นๆ ได้อีกมากมาย:

```rust
// ใช้กับ boolean
let maybe_true: SomethingOrNothing<bool> = Something(true);
let maybe_false: SomethingOrNothing<bool> = Nothing;

// ใช้กับ float
let maybe_pi: SomethingOrNothing<f32> = Something(3.14);

// ซ้อนกันได้ด้วย!
let nested: SomethingOrNothing<SomethingOrNothing<i32>> = Something(Nothing);
```

อันที่จริง type อย่าง `SomethingOrNothing` นั้นมีประโยชน์มากจนมีให้ใช้อยู่แล้วในไลบรารีมาตรฐาน เรียกว่า *ชนิดข้อมูล Option* (Option type) เขียนแทนด้วย `Option<T>` ลองเข้าไปดู[เอกสาร](https://doc.rust-lang.org/stable/std/option/index.html)ของมันได้เลย (และไม่ต้องกังวล ในนั้นมีเนื้อหาอีกเยอะที่เรายังเรียนไปไม่ถึง)

## Generic impl และ Static Functions

type เหล่านี้คล้ายคลึงกันมาก จนเราสามารถสร้างฟังก์ชัน Generic เพื่อแปลง `SomethingOrNothing<T>` จาก `Option<T>` และแปลงกลับได้

สังเกตไวยากรณ์ (Syntax) การเขียน Implementation แบบ Generic:

```rust
impl<T> SomethingOrNothing<T> {
    //      ^ ประกาศ type parameter
    //              ^ นำไปใช้กับ SomethingOrNothing<T>
```

ให้มองว่า `<T>` ตัวแรกคือการ *ประกาศ* ตัวแปร type ("ฉันกำลังจะทำบางอย่างกับทุกๆ type `T`") ส่วน `<T>` ตัวที่สองคือการ *นำไปใช้* ("สิ่งที่ฉันทำ คือการอิมพลีเมนต์ `SomethingOrNothing<T>`")

> **🔍 `Self` vs `self`:**
>
> - `self` (ตัวเล็ก) = ตัวแปรพิเศษ คือ instance ที่เรียก method (เหมือน `this` ในภาษาอื่น)
> - `Self` (ตัวใหญ่) = ชนิดข้อมูล (type) ของ instance นั้น ใช้ใน `impl` เพื่ออ้างถึง type ที่กำลัง implement

ภายใน `impl`, `Self` จะหมายถึง type ที่เรากำลังเขียน Implementation ให้ ในที่นี้คือชื่อเรียกแทน (Alias) ของ `SomethingOrNothing<T>`

```rust
impl<T> SomethingOrNothing<T> {
    fn new(o: Option<T>) -> Self {
        match o {
            None => Nothing,
            Some(t) => Something(t),
        }
    }
    
    fn to_option(self) -> Option<T> {
        match self {
            Nothing => None,
            Something(t) => Some(t),
        }
    }
}
```

สังเกตว่า `new` *ไม่มี* พารามิเตอร์ `self` ซึ่งเทียบได้กับเมธอดแบบ static ใน Java หรือ C++ อันที่จริง `new` เป็นธรรมเนียมปฏิบัติของ Rust ในการนิยามคอนสตรักเตอร์ (Constructor) มันไม่ใช่เรื่องพิเศษอะไร เป็นเพียงฟังก์ชันแบบ static ที่คืนค่า `Self` เท่านั้นเอง

คุณสามารถเรียกฟังก์ชันแบบ static และโดยเฉพาะอย่างยิ่งคอนสตรักเตอร์ ได้ตามตัวอย่างใน `call_constructor`

```rust
fn call_constructor(x: i32) -> SomethingOrNothing<i32> {
    SomethingOrNothing::new(Some(x))
}
```

## Traits

ในเมื่อเรามี `SomethingOrNothing` แบบ Generic แล้ว จะไม่ดีกว่าหรือถ้าจะมี `vec_min` แบบ Generic ด้วย? แน่นอนว่า เราไม่สามารถหาค่าต่ำสุดจาก Vector ของ *ทุก* type ได้ มันต้องเป็น type ที่รองรับการดำเนินการ `min` เท่านั้น Rust เรียกคุณสมบัติที่เราต้องการจาก type เหล่านี้ว่า **Traits**

ดังนั้น เพื่อเป็นก้าวแรกสู่ `vec_min` แบบ Generic เราจะนิยาม Trait `Minimum` ขึ้นมา

> **📝 หมายเหตุเรื่อง `Copy`:**
>
> `Copy` เป็น trait พิเศษที่บอก Rust ว่า type นี้สามารถคัดลอกค่าได้อย่างง่ายดาย (bitwise copy) เช่น `i32`, `f32`, `bool` แต่ไม่ใช่ `String` หรือ `Vec`
>
> เราต้องการ `Copy` ที่นี่เพราะเราใช้ `self` ใน `min(self, b: Self)` ซึ่งจะย้ายค่า (move) ถ้าไม่มี `Copy` เราจะใช้ `&self` แทน ซึ่งเราจะเรียนในบท Borrowing

```rust
pub trait Minimum: Copy {
    fn min(self, b: Self) -> Self;
}
```

`Trait` คล้ายกับ Interface ใน Java มาก คุณนิยามชุดฟังก์ชันที่ต้องการให้อิมพลีเมนต์ พร้อมระบุอาร์กิวเมนต์และ type ที่คืนค่า

ฟังก์ชัน `min` รับอาร์กิวเมนต์สองตัวที่เป็น type เดียวกัน โดยเรากำหนดให้อาร์กิวเมนต์ตัวแรกเป็น `self` แบบพิเศษ หรืออีกทางหนึ่ง เราอาจเขียน `min` เป็นฟังก์ชันแบบ static ก็ได้ ดังนี้ `fn min(a: Self, b: Self) -> Self` อย่างไรก็ตาม ใน Rust เรามักนิยมใช้เมธอดมากกว่าฟังก์ชันแบบ static ถ้าทำได้

ถัดมา เราจะเขียน `vec_min` ให้เป็นฟังก์ชัน Generic บน type `T` โดยมีข้อแม้ว่า type นั้นต้องรองรับ Trait `Minimum` ข้อกำหนดนี้เรียกว่า **Trait bound**

```rust
pub fn vec_min<T: Minimum>(v: Vec<T>) -> SomethingOrNothing<T> {
    let mut min = Nothing;
    for e in v {
        // ชัดเจน: ถ้ามีค่าเก่า (n) ให้หาค่าต่ำสุดระหว่าง n กับ e
        // ถ้ายังไม่มี ใช้ e เป็นค่าแรก
        let new_min = match min {
            Nothing => e,
            Something(n) => n.min(e), // n คือค่าเก่า, e คือค่าใหม่
        };
        min = Something(new_min);
    }
    min
}
```

ความแตกต่างเพียงอย่างเดียวจากเวอร์ชันก่อนหน้าคือเราเรียก `n.min(e)` แทนที่จะเป็น `min_i32(n, e)` ซึ่ง Rust จะรู้โดยอัตโนมัติว่า `n` เป็น type `T` ที่อิมพลีเมนต์ Trait `Minimum` เราจึงเรียกใช้ฟังก์ชันนั้นได้

มีข้อแตกต่างสำคัญเมื่อเทียบกับ Template ใน C++ คือ เราต้องประกาศให้ชัดเจนเลยว่าเราต้องการ Trait อะไรจาก type นั้น (`T: Minimum`) ถ้าเราละ `Minimum` ทิ้งไป Rust จะบ่นทันทีว่าเรียกใช้ `min` ไม่ได้

นี่ตรงกันข้ามอย่างสิ้นเชิงกับ C++ ที่ซึ่ง Compiler จะตรวจสอบรายละเอียดพวกนี้ก็ต่อเมื่อฟังก์ชันถูกนำไปใช้งานจริงๆ เท่านั้น (template instantiation)

ก่อนจะไปต่อ ลองใช้เวลาสักนิดพิจารณาความยืดหยุ่นในแนวคิดเรื่อง Abstraction ของ Rust เราเพิ่งนิยาม Trait (Interface) ขึ้นมาเอง แล้วนำไปอิมพลีเมนต์ให้กับ *type ที่มีอยู่แล้ว* ซึ่งถ้าเป็นแนวทางแบบลำดับชั้น (Hierarchical) อย่างใน C++ หรือ Java จะทำแบบนี้ไม่ได้ เราไม่สามารถจับ type ที่มีอยู่แล้วไปสืบทอด (Inherit) จาก Abstract base class ของเราทีหลังได้

> **⚡ ประสิทธิภาพ:** Rust ทำการ **Monomorphisation** กับฟังก์ชัน Generic เมื่อคุณเรียก `vec_min` โดยที่ `T` เป็น `i32` Rust จะสร้างสำเนาของฟังก์ชันสำหรับ type นี้โดยเฉพาะ การเรียก `T::min` จะถูกเปลี่ยนเป็นการเรียกโค้ดที่เราอิมพลีเมนต์ไว้แบบ *statically* ไม่มีการทำ *Dynamic dispatch* เหมือนอย่างเมธอดของ Java Interface

## การอิมพลีเมนต์ Trait (Trait Implementations)

เพื่อทำให้ `vec_min` ใช้งานได้กับ `Vec<i32>` เราต้องอิมพลีเมนต์ Trait `Minimum` ให้กับ `i32`

```rust
impl Minimum for i32 {
    fn min(self, b: Self) -> Self {
        if self < b { self } else { b }
    }
}
```

เราได้เตรียมฟังก์ชัน `print` ไว้อีกครั้ง ตรงนี้แสดงให้เห็นด้วยว่าเราสามารถมีบล็อก `impl` หลายบล็อกสำหรับ type เดียวกันได้ (อย่าลืมว่า `NumberOrNothing` เป็นแค่ type alias ของ `SomethingOrNothing<i32>`) และเราสามารถสร้างเมธอดเฉพาะสำหรับบางอินสแตนซ์ของ Generic type ได้ด้วย

```rust
impl NumberOrNothing {
    pub fn print(self) {
        match self {
            Nothing => println!("The number is: <nothing>"),
            Something(n) => println!("The number is: {}", n),
        };
    }
}
```

ตอนนี้เราพร้อมรันโค้ดใหม่กันแล้ว อย่าลืมแก้ `main.rs` ให้ถูกต้องด้วยล่ะ Rust จะรู้โดยอัตโนมัติว่าเราต้องการให้ `T` ใน `vec_min` เป็น `i32` และรู้ว่า `i32` อิมพลีเมนต์ `Minimum` ทุกอย่างจึงทำงานได้เรียบร้อย

```rust
fn read_vec() -> Vec<i32> {
    vec![18, 5, 7, 3, 9, 27]
}

pub fn main() {
    let vec = read_vec();
    let min = vec_min(vec);
    min.print();
}
```

ถ้าโปรแกรมพิมพ์เลข `3` ออกมา แสดงว่า `vec_min` แบบ Generic ของคุณทำงานได้ เตรียมตัวให้พร้อมสำหรับส่วนถัดไปได้เลย

## เชื่อมโยงกับโลกจริง: ใช้ Option<T> แทน

ในโค้ดจริง เราไม่ต้องสร้าง `SomethingOrNothing` เอง แต่ใช้ `Option<T>` ที่มีอยู่แล้ว พร้อมด้วย traits ที่มีประโยชน์มากมาย:

```rust
// แทนที่จะสร้างเอง
use std::option::Option; // จริงๆ ไม่ต้อง use ก็ได้ เพราะอยู่ใน prelude

fn vec_min_real<T: Ord>(v: Vec<T>) -> Option<T> {
    let mut min: Option<T> = None;
    for e in v {
        min = Some(match min {
            None => e,
            Some(n) => if n < e { n } else { e }, // หรือใช้ n.min(e) ถ้า T: Ord
        });
    }
    min
}

// ใช้งาน
let min = vec_min_real(vec![5, 2, 8, 1]);
println!("{:?}", min); // Some(1)
```

> **💡 ความรู้เพิ่มเติม:** `Ord` เป็น trait ที่มีอยู่แล้วใน standard library สำหรับ type ที่สามารถเปรียบเทียบกันได้ (มีการเรียงลำดับ) รวมถึง `i32`, `f32`, `String` เป็นต้น

## แบบฝึกหัด

**แบบฝึกหัด 02.1:** จงเปลี่ยนโปรแกรมของคุณให้มันคำนวณค่าต่ำสุดของ `Vec<f32>` (เมื่อ `f32` คือ type ของเลขทศนิยมแบบ 32-bit) แน่นอนว่าห้ามแก้ไข `vec_min` โดยเด็ดขาด

> **💡 คำใบ้:** `f32` (float 32-bit) ก็มี trait `Copy` เช่นเดียวกับ `i32` คุณแค่ต้อง implement trait `Minimum` ให้กับ `f32` เท่านั้น

<details>
<summary>เฉลย</summary>

```rust
impl Minimum for f32 {
    fn min(self, b: Self) -> Self {
        if self < b { self } else { b }
    }
}

// หรือใช้ method ที่มีอยู่แล้วของ f32
impl Minimum for f32 {
    fn min(self, b: Self) -> Self {
        self.min(b) // f32 มี method min อยู่แล้ว!
    }
}

fn main() {
    let vec = vec![3.14, 2.71, 1.41, 1.73];
    let min = vec_min(vec);
    match min {
        Something(n) => println!("Min: {}", n),
        Nothing => println!("Empty vector"),
    }
}
```

</details>

**แบบฝึกหัด 02.2 (ท้าทาย):** สร้าง trait `Printable` ที่มี method `print` และ implement ให้กับ `SomethingOrNothing<T>` โดยที่ `T` ต้อง implement `Display` (จาก standard library)

<details>
<summary>เฉลย</summary>

```rust
use std::fmt::Display;

trait Printable {
    fn print(self);
}

impl<T: Display> Printable for SomethingOrNothing<T> {
    fn print(self) {
        match self {
            Nothing => println!("Nothing"),
            Something(t) => println!("Something: {}", t),
        }
    }
}
```

</details>
