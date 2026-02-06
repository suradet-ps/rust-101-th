## Part 02: Generic types, Traits

ลองย้อนกลับมาดู type `NumberOrNothing` กันอีกสักนิด ไม่รู้สึกขัดใจบ้างหรือที่เราต้องฮาร์ดโค้ด type `i32` ลงไปตรงๆ? แล้วถ้าพรุ่งนี้เราอยากได้ `CharOrNothing` และวันถัดไปอยากได้ `FloatOrNothing` ล่ะ? แน่นอนว่าเราคงไม่อยากเขียน type และเมธอดประจำตัวของมันขึ้นมาใหม่ทั้งหมดแน่ๆ

### Generic datatypes

ทางออกของปัญหานี้เรียกว่า *Generics* หรือ *Polymorphism* (คำหลังเป็นภาษากรีก แปลว่า "หลายรูปทรง") คุณอาจเคยเห็นอะไรคล้ายๆ กันนี้มาแล้วใน C++ (ที่เรียกว่า *Template*) หรือใน Java รวมถึงภาษาตระกูล Functional languages อื่นๆ เอาล่ะ ในที่นี้เราจะนิยาม Generic type ที่ชื่อ `SomethingOrNothing` กัน

```rust
pub enum SomethingOrNothing<T> {
    Something(T),
    Nothing,
}
```

แทนที่จะต้องคอยเขียนชื่อเต็มของรูปแบบ (Variant) ทุกครั้ง เราสามารถ Import ทั้งหมดเข้ามาทีเดียวได้เลย

```rust
pub use self::SomethingOrNothing::*;
```

สิ่งที่ทำตรงนี้คือการนิยามตระกูลของ type ขึ้นมาทั้งชุด ตอนนี้เราสามารถเขียน `SomethingOrNothing<i32>` เพื่อให้ได้ `NumberOrNothing` แบบเดิมกลับมา

```rust
type NumberOrNothing = SomethingOrNothing<i32>;
```

อย่างไรก็ตาม เรายังสามารถเขียน `SomethingOrNothing<bool>` หรือแม้กระทั่ง `SomethingOrNothing<SomethingOrNothing<i32>>` ก็ได้ อันที่จริง type อย่าง `SomethingOrNothing` นั้นมีประโยชน์มากจนมีให้ใช้อยู่แล้วในไลบรารีมาตรฐาน เรียกว่า *ชนิดข้อมูล Option* (Option type) เขียนแทนด้วย `Option<T>` ลองเข้าไปดู[เอกสาร](https://doc.rust-lang.org/stable/std/option/index.html))ของมันได้เลย (และไม่ต้องกังวล ในนั้นมีเนื้อหาอีกเยอะที่เรายังเรียนไปไม่ถึง)

### Generic impl, ฟังก์ชัน Static

type เหล่านี้คล้ายคลึงกันมาก จนเราสามารถสร้างฟังก์ชัน Generic เพื่อแปลง `SomethingOrNothing<T>` จาก `Option<T>` และแปลงกลับได้

สังเกตไวยากรณ์ (Syntax) การเขียน Implementation แบบ Generic ให้กับ Generic types ให้มองว่า `<T>` ตัวแรกคือการ *ประกาศ* ตัวแปร type ("ฉันกำลังจะทำบางอย่างกับทุกๆ type `T`") ส่วน `<T>` ตัวที่สองคือการ *นำไปใช้* ("สิ่งที่ฉันทำ คือการอิมพลีเมนต์ `SomethingOrNothing<T>`")

ภายใน `impl`, `Self` จะหมายถึง type ที่เรากำลังเขียน Implementation ให้ ในที่นี้คือชื่อเรียกแทน (Alias) ของ `SomethingOrNothing<T>` จำไว้ว่า `self` ก็คือ `this` ใน Rust และโดยปกติแล้วมันจะมี type เป็น `Self`

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

### Traits

ในเมื่อเรามี `SomethingOrNothing` แบบ Generic แล้ว จะไม่ดีกว่าหรือถ้าจะมี `vec_min` แบบ Generic ด้วย? แน่นอนว่า เราไม่สามารถหาค่าต่ำสุดจาก Vector ของ *ทุก* type ได้ มันต้องเป็น type ที่รองรับการดำเนินการ `min` เท่านั้น Rust เรียกคุณสมบัติที่เราต้องการจาก type เหล่านี้ว่า *Traits*

ดังนั้น เพื่อเป็นก้าวแรกสู่ `vec_min` แบบ Generic เราจะนิยาม Trait `Minimum` ขึ้นมา (ตอนนี้ให้มองข้าม `Copy` ไปก่อน เราจะกลับมาคุยเรื่องนี้กันทีหลัง) `Trait` คล้ายกับ Interface ใน Java มาก คุณนิยามชุดฟังก์ชันที่ต้องการให้อิมพลีเมนต์ พร้อมระบุอาร์กิวเมนต์และ type ที่คืนค่า

ฟังก์ชัน `min` รับอาร์กิวเมนต์สองตัวที่เป็น type เดียวกัน แต่ผมกำหนดให้อาร์กิวเมนต์ตัวแรกเป็น `self` แบบพิเศษ หรืออีกทางหนึ่ง ผมอาจเขียน `min` เป็นฟังก์ชันแบบ static ก็ได้ ดังนี้ `fn min(a: Self, b: Self) -> Self` อย่างไรก็ตาม ใน Rust เรามักนิยมใช้เมธอดมากกว่าฟังก์ชันแบบ static ถ้าทำได้

```rust
pub trait Minimum: Copy {
    fn min(self, b: Self) -> Self;
}
```

ถัดมา เราจะเขียน `vec_min` ให้เป็นฟังก์ชัน Generic บน type `T` โดยมีข้อแม้ว่า type นั้นต้องรองรับ Trait `Minimum` ข้อกำหนดนี้เรียกว่า *Trait bound* ความแตกต่างเพียงอย่างเดียวจากเวอร์ชันก่อนหน้าคือเราเรียก `e.min(n)` แทนที่จะเป็น `min_i32(n, e)` ซึ่ง Rust จะรู้โดยอัตโนมัติว่า `e` เป็น type `T` ที่อิมพลีเมนต์ Trait `Minimum` เราจึงเรียกใช้ฟังก์ชันนั้นได้

มีข้อแตกต่างสำคัญเมื่อเทียบกับ Template ใน C++ คือ เราต้องประกาศให้ชัดเจนเลยว่าเราต้องการ Trait อะไรจาก type นั้น ถ้าเราละ `Minimum` ทิ้งไป Rust จะบ่นทันทีว่าเรียกใช้ `min` ไม่ได้ ลองดูสิ

นี่ตรงกันข้ามอย่างสิ้นเชิงกับ C++ ที่ซึ่ง Compiler จะตรวจสอบรายละเอียดพวกนี้ก็ต่อเมื่อฟังก์ชันถูกนำไปใช้งานจริงๆ เท่านั้น

```rust
pub fn vec_min<T: Minimum>(v: Vec<T>) -> SomethingOrNothing<T> {
    let mut min = Nothing;
    for e in v {
        min = Something(match min {
            Nothing => e,
            Something(n) => e.min(n),
        });
    }
    min
}
```
ตรงนี้ เราสามารถเรียกฟังก์ชัน `min` ของ Trait ได้แล้ว

ก่อนจะไปต่อ ลองใช้เวลาสักนิดพิจารณาความยืดหยุ่นในแนวคิดเรื่อง Abstraction ของ Rust เราเพิ่งนิยาม Trait (Interface) ขึ้นมาเอง แล้วนำไปอิมพลีเมนต์ให้กับ *type ที่มีอยู่แล้ว* ซึ่งถ้าเป็นแนวทางแบบลำดับชั้น (Hierarchical) อย่างใน C++ หรือ Java จะทำแบบนี้ไม่ได้ เราไม่สามารถจับ type ที่มีอยู่แล้วไปสืบทอด (Inherit) จาก Abstract base class ของเราทีหลังได้

หากคุณกังวลเรื่องประสิทธิภาพ โปรดทราบว่า Rust ทำการ *Monomorphisation* กับฟังก์ชัน Generic เมื่อคุณเรียก `vec_min` โดยที่ `T` เป็น `i32` โดยพื้นฐานแล้ว Rust จะสร้างสำเนาของฟังก์ชันสำหรับ type นี้โดยเฉพาะ และเติมส่วนที่ว่างทั้งหมดให้ ในกรณีนี้ การเรียก `T::min` จะถูกเปลี่ยนเป็นการเรียกโค้ดที่เราอิมพลีเมนต์ไว้แบบ *statically* ไม่มีการทำ *Dynamic dispatch* เหมือนอย่างเมธอดของ Java Interface หรือ *Virtual methods* ของ C++ พฤติกรรมนี้คล้ายกับ C++ Template ซึ่งตัวปรับปรุงประสิทธิภาพ (Optimizer - Rust ใช้ LLVM) จะมีข้อมูลครบถ้วนที่ต้องการเพื่อนำไปปรับแต่ง เช่น การทำ Inline ฟังก์ชัน

### การอิมพลีเมนต์ Trait (Trait implementations)

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

**แบบฝึกหัด 02.1** จงเปลี่ยนโปรแกรมของคุณให้มันคำนวณค่าต่ำสุดของ `Vec<f32>` (เมื่อ `f32` คือ type ของเลขทศนิยมแบบ 32-bit) แน่นอนว่าห้ามแก้ไข `vec_min` โดยเด็ดขาด
