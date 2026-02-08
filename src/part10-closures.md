## Part 10: Closures

```rust
use std::fmt;
use part05::BigInt;
```

สมมติว่าเราต้องการเขียนฟังก์ชันที่ทำบางอย่างกับทุกหลักของ `BigInt` เราจะต้องการวิธีแสดง action ที่เราต้องการให้เกิดขึ้น และส่งสิ่งนี้ไปยังฟังก์ชันของเรา ใน Rust ความพยายามแรกที่เป็นธรรมชาติเพื่อแสดงสิ่งนี้คือการมี trait สำหรับมัน

ดังนั้น ให้เรานิยาม trait ที่บังคับว่าไทป์ต้อง provide เมธอด `do_action` บางอย่างบนหลักต่างๆ สิ่งนี้ยกคำถามขึ้นมาทันที: เราส่ง `self` ไปยังฟังก์ชันนั้นอย่างไร? Owned, shared reference หรือ mutable reference? กลยุทธ์ที่เป็นแบบอย่างเพื่อตอบคำถามนี้คือใช้ไทป์ที่แข็งแกร่งที่สุดที่ยังทำงานได้ แน่นอน การส่ง `self` ในรูปแบบ owned ไม่ทำงาน: แล้วฟังก์ชันจะ consume `self` และเราจะไม่สามารถเรียกมันอีกครั้งบนหลักที่สองได้ ดังนั้นให้เราไปกับ mutable reference

```rust
trait Action {
    fn do_action(&mut self, digit: u64);
}
```

ทีนี้เราสามารถเขียนฟังก์ชันที่รับ `a` บางอย่างของไทป์ `A` เช่นที่เราสามารถเรียก `do_action` บน `a` ส่งหลักทุกหลักให้มัน

```rust
impl BigInt {
    fn act_v1<A: Action>(&self, mut a: A) {
```

จำไว้ว่า `mut` ข้างต้นเป็นเพียง annotation บอก Rust ว่าเราโอเคกับการที่ `a` ถูก mutate การเรียก `do_action` บน `a` รับ mutable reference ดังนั้น mutation อาจเกิดขึ้นจริงๆ

```rust
        for digit in self {
            a.do_action(digit);
        }
    }
}
```

เป็นขั้นตอนถัดไป เราต้องคิด action บางอย่าง และเขียน implementation ของ `Action` ที่เหมาะสมสำหรับมัน ดังนั้น ให้เราบอกว่าเราต้องการพิมพ์ทุก digit และเพื่อให้สิ่งนี้น่าสนใจขึ้น เราต้องการให้หลักต่างๆ มี prefix โดย string บางอย่าง `do_action` ต้องรู้จัก string นี้ ดังนั้นเราเก็บมันใน `self`

```rust
struct PrintWithString {
    prefix: String,
}

impl Action for PrintWithString {
```

ที่นี่เราทำการพิมพ์จริงของ prefix และหลัก เราไม่ได้ใช้ความสามารถในการเปลี่ยน `self` ที่นี่ แต่เราอาจ replace prefix ถ้าเราต้องการ

```rust
    fn do_action(&mut self, digit: u64) {
        println!("{}{}", self.prefix, digit);
    }
}
```

สุดท้าย ฟังก์ชันนี้รับ `BigInt` และ prefix และพิมพ์หลักต่างๆ ด้วย prefix ที่กำหนด มันทำสิ่งนี้โดยการสร้าง instance ของ `PrintWithString` ให้มัน prefix แล้วส่งมันไปยัง `act_v1` เนื่องจาก `PrintWithString` implement `Action` Rust ทีนี้รู้ว่าต้องทำอะไร

```rust
fn print_with_prefix_v1(b: &BigInt, prefix: String) {
    let my_action = PrintWithString { prefix: prefix };
    b.act_v1(my_action);
}
```

นี่คือฟังก์ชัน `main` เล็กๆ สาธิตโค้ดข้างต้นใน action อย่าลืมแก้ไข `main.rs` เพื่อรันโค้ดนี้

```rust
pub fn main() {
    let bignum = BigInt::new(1 << 63) + BigInt::new(1 << 16) + BigInt::new(1 << 63);
    print_with_prefix_v1(&bignum, "Digit: ".to_string());
}
```

## Closures

ทีนี้ ปรากฏว่า pattern นี้ของการอธิบายรูปแบบบางอย่างของ action ที่สามารถ carry ข้อมูลเพิ่มเติมได้ เป็นเรื่องที่พบบ่อยมาก โดยทั่วไป สิ่งนี้เรียกว่า **closure** Closures รับ arguments บางอย่างและสร้างผลลัพธ์ และพวกมันมี environment ที่สามารถใช้ได้ ซึ่งสอดคล้องกับไทป์ `PrintWithString` (หรือไทป์อื่นๆ ที่ implement `Action`) อีกครั้งเรามีตัวเลือกในการส่ง environment นี้ในรูปแบบ owned หรือ borrowed ดังนั้นมีสาม traits สำหรับ closures ใน Rust:

- `Fn` - closures ได้รับ shared reference
- `FnMut` - closures ได้รับ mutable reference  
- `FnOnce` - closures consume environment ของพวกมัน (และดังนั้นจึงสามารถถูกเรียกได้เพียงครั้งเดียว)

ไวยากรณ์สำหรับ closure trait ที่รับ arguments ของไทป์ `T1, T2, ...` และคืนบางอย่างของไทป์ `U` คือ `Fn(T1, T2, ...) -> U`

สิ่งนี้นิยาม `act` คล้ายกับข้างต้น แต่ตอนนี้เราต้องการ `A` เป็นไทป์ของ closure ที่ mutate environment ที่ถูกยืมมา รับหลัก และคืน ()

```rust
impl BigInt {
    fn act<A: FnMut(u64)>(&self, mut a: A) {
        for digit in self {
```

เราสามารถเรียก closures เสมือนพวกมันเป็นฟังก์ชัน - แต่จริงๆ แล้วสิ่งที่เกิดขึ้นที่นี่ถูกแปลเป็นโดยพื้นฐานสิ่งที่เราเขียนข้างต้น ใน `act_v1`

```rust
            a(digit);
        }
    }
}
```

ทีนี้ที่เราเห็นวิธีเขียนฟังก์ชันที่ทำงานบน closures ให้เราดูวิธีเขียน closure

```rust
pub fn print_with_prefix(b: &BigInt, prefix: String) {
```

ไวยากรณ์สำหรับ closures คือ `|arg1, arg2, ...| code` สังเกตว่า closure สามารถอ้างอิงถึงตัวแปรอย่าง `prefix` ที่มันไม่ได้รับเป็น argument - ตัวแปรที่บังเอิญมีอยู่ภายนอก closure เราบอกว่า closure **capture** ตัวแปร Rust ทีนี้จะสร้างไทป์อัตโนมัติ (เหมือน `PrintWithString`) สำหรับ environment ของ closure ด้วยฟิลด์สำหรับทุกตัวแปรที่ถูก capture implement closure trait สำหรับไทป์นี้ เช่นที่ action ที่ถูกทำคือโค้ดของ closure และสุดท้ายมันจะ instantiate ไทป์ environment ที่นี่ที่ site ของการนิยาม closure และเติมมัน

```rust
    b.act(|digit| println!("{}{}", prefix, digit));
}
```

คุณสามารถเปลี่ยน `main` เพื่อเรียกฟังก์ชันนี้ และคุณควรสังเกต - ไม่มีอะไร ไม่มีความแตกต่างใน behavior แต่เราเขียน boilerplate code น้อยกว่ามาก!

จำไว้ว่าเราตัดสินใจใช้ trait `FnMut` ข้างต้น? นี่หมายความว่า closure ของเราสามารถ mutate environment ของมันได้จริงๆ เช่น เราสามารถใช้สิ่งนั้นเพื่อนับหลักต่างๆ ขณะที่พวกมันถูกพิมพ์

```rust
pub fn print_and_count(b: &BigInt) {
    let mut count: usize = 0;
```

ครั้งนี้ environment จะมีฟิลด์ของไทป์ `&mut usize` ที่จะถูก initialize ด้วย mutable reference ของ `count` Closure เนื่องจากมัน mutably borrow environment ของมัน จึงสามารถเข้าถึงฟิลด์นี้และ mutate `count` ผ่านมัน เมื่อ `act` คืนค่า closure ถูกทำลายและ `count` ไม่ถูกยืมอีกต่อไป เพราะ closures compile ลงมาเป็น normal types การ borrow checking ทั้งหมดยังคงทำงานตามปกติ และเราไม่สามารถ accidentally leak closure ไปที่ไหนสักแห่งที่ยังมีในตัวมัน ใน environment ของมัน reference ที่ตายแล้ว

```rust
    b.act(|digit| {
        println!("{}: {}", count, digit);
        count = count + 1;
    });
    println!("There are {} digits", count);
}
```

## Fun with iterators and closures

ถ้าคุณคุ้นเคยกับ functional languages คุณอาจรู้ว่าเราสามารถสนุกได้มากกับ iterators และ closures Rust provide methods มากมายบน iterators ที่อนุญาตให้เราเขียน list operations แบบ functional-style ได้สวยงาม

ให้เราบอกว่าเราต้องการเขียนฟังก์ชันที่ increment ทุก entry ของ `Vec` ด้วยตัวเลขบางอย่าง แล้วมองหาตัวเลขที่ใหญ่กว่า threshold บางอย่าง และพิมพ์พวกมัน

```rust
fn inc_print_threshold(v: &Vec<i32>, offset: i32, threshold: i32) {
```

`map` รับ closure ที่ถูก apply กับทุก element ของ iterator `filter` ลบ elements ออกจาก iterator ที่ไม่ผ่านการทดสอบที่กำหนดโดย closure

เนื่องจาก closures ทั้งหมดเหล่านี้ compile ลงมาเป็น pattern ที่อธิบายข้างต้น จริงๆ แล้วไม่มี heap allocation เกิดขึ้นที่นี่ สิ่งนี้ทำให้ closures มีประสิทธิภาพมาก และทำให้ optimization ค่อนข้าง trivial: โค้ดที่ได้จะดูเหมือนคุณเขียน loop ด้วยมือใน C

```rust
    for i in v.iter().map(|n| *n + offset).filter(|n| *n > threshold) {
        println!("{}", i);
    }
}
```

บางครั้งมันมีประโยชน์ที่จะรู้ทั้งตำแหน่งของ element บางอย่างใน list และค่าของมัน นั่นคือที่ที่ฟังก์ชัน `enumerate` ช่วยได้

```rust
fn print_enumerated<T: fmt::Display>(v: &Vec<T>) {
```

`enumerate` เปลี่ยน iterator ผ่าน `T` เป็น iterator ผ่าน `(usize, T)` โดยที่ element แรกนับตำแหน่งใน iterator เราสามารถทำ pattern matching ได้เลยใน header ของ loop เพื่อได้ชื่อสำหรับทั้งตำแหน่งและค่า

```rust
    for (i, t) in v.iter().enumerate() {
        println!("Position {}: {}", i, t);
    }
}
```

และเป็นตัวอย่างสุดท้าย เรายังสามารถ collect ทุก elements ของ iterator และใส่มัน เช่น ในเวกเตอร์

```rust
fn filter_vec_by_divisor(v: &Vec<i32>, divisor: i32) -> Vec<i32> {
```

ที่นี่ ไทป์คืนค่าของ `collect` ถูก infer จากไทป์คืนค่าของฟังก์ชันของเรา โดยทั่วไป มันสามารถคืนอะไรก็ได้ที่ implement `FromIterator` สังเกตว่า `iter` ให้เรา iterator ผ่าน borrowed `i32` แต่เราต้องการ own พวกมันสำหรับผลลัพธ์ ดังนั้นเราแทรก `map` เพื่อ dereference

```rust
    v.iter().map(|n| *n).filter(|n| *n % divisor == 0).collect()
}
```

**แบบฝึกหัด 10.1:** ดูเอกสารของ `Iterator` เพื่อเรียนรู้เกี่ยวกับฟังก์ชันเพิ่มเติมที่สามารถทำงานบน iterators ลองใช้บางอันดู ฟังก์ชันที่ sum เลขคู่ของ iterator ล่ะ? หรือฟังก์ชันที่คำนวณผลคูณของตัวเลขเหล่านั้นที่อยู่ในตำแหน่งคี่? ฟังก์ชันที่ตรวจสอบว่าเวกเตอร์มีตัวเลขบางตัวหรือไม่? ว่าตัวเลขทั้งหมดเล็กกว่า threshold บางอย่างหรือไม่? สร้างสรรค์ดู!

**แบบฝึกหัด 10.2:** เราเริ่มการเดินทางในบท 02 ด้วย `SomethingOrNothing<T>` และต่อมาเรียนรู้เกี่ยวกับ `Option<T>` ในบท 04 `Option<T>` ยังมีฟังก์ชัน `map` ด้วย อ่านเอกสารของมันที่นี่ ฟังก์ชันใดในบทก่อนหน้าที่คุณสามารถเขียนใหม่เพื่อใช้ `map` แทน? (คำใบ้: อ่าน source code ของ `map` และดูว่า pattern ปรากฏในโค้ดของคุณเองหรือไม่) Bonus: `test_invariant` ในบท 05 ไม่ใช้ `match` แต่คุณยังสามารถหาวิธีเขียนมันใหม่ด้วย `map` ได้หรือไม่?
