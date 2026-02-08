## Part 07: Operator Overloading, Tests, Formatting

```rust
pub use part05::BigInt;
```

ด้วยความรู้ใหม่ของเราเรื่อง lifetimes ตอนนี้เราสามารถเขียนไทป์ที่ต้องการของ `min` ได้แล้ว: เราต้องการให้ฟังก์ชันรับสอง references ที่มี lifetime เดียวกัน แล้วคืน reference ที่มี lifetime นั้น ถ้าสอง input lifetimes ต่างกัน เราจะไม่รู้ว่าควรใช้ lifetime ไหนสำหรับผลลัพธ์

```rust
pub trait Minimum {
    fn min<'a>(&'a self, other: &'a Self) -> &'a Self;
}
```

ทีนี้เราสามารถ implement ฟังก์ชัน generic `vec_min` ที่ทำงานกับ trait ข้างต้นได้ โค้ดค่อนข้างตรงไปตรงมา และ Rust ตรวจสอบว่า lifetimes ทั้งหมดทำงานถูกต้อง สังเกตว่าเราไม่ต้องทำสำเนาใดๆ เลย!

```rust
pub fn vec_min<T: Minimum>(v: &Vec<T>) -> Option<&T> {
    let mut min: Option<&T> = None;
    for e in v {
        min = Some(match min {
            None => e,
            Some(n) => n.min(e)
        });
    }
    min
}
```

สังเกตว่าไทป์ที่คืน `Option<&T>` ทางเทคนิคแล้ว (ละเรื่อง borrowing ไปก่อน) คือ pointer ไปยัง `T` ที่อาจเป็น invalid ได้ กล่าวอีกนัยหนึ่ง มันเหมือนกับ pointer ใน C(++) หรือ Java ที่อาจเป็น NULL อย่างไรก็ตาม ขอบคุณที่ `Option` เป็น enum เราจึงไม่สามารถลืมตรวจสอบ pointer ว่าถูกต้องหรือไม่ได้ หลีกเลี่ยงปัญหาความปลอดภัยของ C(++)

นอกจากนี้ ถ้าคุณกังวลเรื่องการเสียพื้นที่ สังเกตว่า Rust รู้ว่า `&T` ไม่มีทางเป็น NULL ดังนั้นจึง optimize `Option<&T>` ให้มีขนาดไม่ใหญ่กว่า `&T` กรณี `None` ถูกแทนด้วย NULL นี่เป็นอีกตัวอย่างที่ดีของ **zero-cost abstraction**: `Option<&T>` เหมือนกับ pointer ใน C(++) ถ้าคุณดูว่าเกิดอะไรขึ้นระหว่าง execution - แต่ใช้งานปลอดภัยกว่ามาก

**แบบฝึกหัด 07.1:** สำหรับให้ `vec_min` ของเราใช้งานได้กับ `BigInt` คุณต้อง provide implementation ของ `Minimum` คุณควรสามารถ copy โค้ดที่เขียนสำหรับแบบฝึกหัด 06.1 ได้เลย คุณไม่ควรทำสำเนา `BigInt` ใดๆ!

```rust
impl Minimum for BigInt {
    fn min<'a>(&'a self, other: &'a Self) -> &'a Self {
        unimplemented!()
    }
}
```

## Operator Overloading

เราจะรู้ได้อย่างไรว่าฟังก์ชัน `min` ของเราทำงานถูกต้องจริงๆ? ความเป็นไปได้หนึ่งคือการทดสอบ Rust มาพร้อมการสนับสนุนที่ดีสำหรับทั้ง unit tests และ integration tests อย่างไรก็ตาม ก่อนจะไปถึงตรงนั้น เราต้องมีวิธีตรวจสอบว่าผลลัพธ์ของการเรียกฟังก์ชันถูกต้องหรือไม่ กล่าวอีกนัยหนึ่ง เราต้องนิยามวิธีทดสอบความเท่ากันของ `BigInt` การสามารถทดสอบความเท่ากันเป็นคุณสมบัติของไทป์ ที่ - คุณเดาถูกแล้ว - Rust แสดงออกเป็น trait: `PartialEq`

การทำสิ่งนี้สำหรับ `BigInt` ค่อนข้างง่าย ขอบคุณที่เรากำหนด requirement ว่าต้องไม่มีเลขศูนย์ต่อท้าย เราแค่ reuse การทดสอบความเท่ากันบน vectors ซึ่งเปรียบเทียบ elements แต่ละตัวแยกกัน attribute `inline` บอก Rust ว่าเราต้องการให้ฟังก์ชันนี้ถูก inline โดยทั่วไป

```rust
impl PartialEq for BigInt {
    #[inline]
    fn eq(&self, other: &BigInt) -> bool {
        debug_assert!(self.test_invariant() && other.test_invariant());
        self.data == other.data
    }
}
```

เนื่องจากการ implement `PartialEq` เป็นเรื่องค่อนข้าง mechanical คุณสามารถให้ Rust automate สิ่งนี้ได้โดยการเพิ่ม attribute `derive(PartialEq)` ไปที่คำนิยามไทป์ ในกรณีที่คุณสงสัยเรื่อง "partial" ผมแนะนำให้คุณดูเอกสารของ `PartialEq` และ `Eq` `Eq` สามารถ derive โดยอัตโนมัติได้เช่นกัน

ทีนี้เราสามารถเปรียบเทียบ `BigInt` ได้แล้ว Rust ปฏิบัติกับ `PartialEq` เป็นพิเศษตรงที่มันเชื่อมต่อกับตัวดำเนินการ `==`: ตัวดำเนินการนั้นสามารถใช้กับตัวเลขของเราได้แล้ว! พูดในภาษา C++ เราเพิ่ง overload ตัวดำเนินการ `==` สำหรับ `BigInt` Rust ไม่มี function overloading (คือจะไม่ dispatch ไปยังฟังก์ชันต่างกันขึ้นอยู่กับไทป์ของอาร์กิวเมนต์) แทนที่จะเป็นแบบนั้น โดยทั่วไปจะหา (หรือนิยาม) trait ที่จับลักษณะสำคัญร่วมกันของทุก overloads และเขียนฟังก์ชันเดียวที่เป็น generic ใน trait ตัวอย่างเช่น แทนที่จะ overload ฟังก์ชันสำหรับทุกวิธีที่ string สามารถถูกแทนได้ เราเขียนฟังก์ชัน generic ผ่าน `ToString` โดยทั่วไป มี trait แบบนี้ที่เหมาะกับวัตถุประสงค์ - และถ้ามี ข้อได้เปรียบที่ยิ่งใหญ่คือไทป์ใดๆ ที่คุณเขียน ที่สามารถแปลงเป็น string ได้ แค่ต้อง implement trait นั้นเพื่อใช้งานได้ทันทีกับฟังก์ชันทั้งหมดที่ generalize ผ่าน `ToString` เปรียบเทียบกับ C++ หรือ Java ที่โอกาสเดียวที่จะเพิ่ม overloading variant ใหม่คือการแก้ไข class ของ receiver

ทำไมเราถึงใช้ `!=` ได้ด้วย ทั้งที่เราแค่ overload `==`? คำตอบอยู่ที่สิ่งที่เรียกว่า **default implementation** ถ้าคุณดูเอกสารของ `PartialEq` ที่ผมลิงก์ข้างต้น คุณจะเห็นว่า trait จริงๆ แล้ว provide สองเมธอด: `eq` เพื่อทดสอบความเท่ากัน และ `ne` เพื่อทดสอบความไม่เท่ากัน อย่างที่คุณอาจเดาได้ `!=` เชื่อมต่อกับ `ne` คำนิยาม trait ยัง provide default implementation ของ `ne` ให้เป็น negation ของ `eq` ดังนั้นคุณแค่ provide `eq` และ `!=` ก็จะทำงานได้ดี หรือถ้าคุณมีวิธีที่มีประสิทธิภาพกว่าในการตัดสินใจความไม่เท่ากัน คุณสามารถ provide `ne` สำหรับไทป์ของคุณเองได้

```rust
fn compare_big_ints() {
    let b1 = BigInt::new(13);
    let b2 = BigInt::new(37);
    println!("b1==b1: {}; b1==b2: {}; b1!=b2: {}", b1 == b1, b1 == b2, b1 != b2);
}
```

## Testing

ด้วยการทดสอบความเท่ากันที่เขียนแล้ว ตอนนี้เราพร้อมเขียน testcase แรกของเรา มันไม่ซับซ้อนไปกว่านี้: คุณแค่เขียนฟังก์ชัน (ไม่มีอาร์กิวเมนต์หรือค่าคืน) และให้ attribute `test` กับมัน `assert!` เหมือนกับ `debug_assert!` แต่ไม่ถูก compile ออกไปใน release builds

```rust
#[test]
fn test_min() {
    let b1 = BigInt::new(1);
    let b2 = BigInt::new(42);
    let b3 = BigInt::from_vec(vec![0, 1]);
    assert!(*b1.min(&b2) == b1);
    assert!(*b3.min(&b2) == b2);
}
```

ทีนี้รัน `cargo test` เพื่อ execute test ถ้าคุณ implement `min` ถูกต้อง มันควรทำงานได้ทั้งหมด!

## Formatting

ยังมีมาโคร `assert_eq!` ที่เชี่ยวชาญเฉพาะการทดสอบความเท่ากัน และจะพิมพ์สองค่า (ซ้ายและขวา) ถ้าพวกมันต่างกัน เพื่อทำสิ่งนั้นได้ มาโครต้องรู้วิธี format ค่าสำหรับการพิมพ์ นี่หมายความว่าเรา - เดาถูกอีกแล้ว? - ต้อง implement trait ที่เหมาะสม Rust รู้จักสองวิธีในการ format ค่า: `Display` สำหรับ pretty-print สิ่งที่ผู้ใช้เข้าใจได้ ในขณะที่ `Debug` มีไว้แสดงสถานะภายในของข้อมูลและเป้าหมายที่ programmer หลังนี้คือสิ่งที่เราต้องการสำหรับ `assert_eq!` ดังนั้นมาเริ่มกันเลย

การ format ทั้งหมดถูกจัดการโดย `std::fmt` ผมจะไม่อธิบายรายละเอียดทั้งหมด และแนะนำให้คุณดูเอกสาร

```rust
use std::fmt;
```

ในกรณีของ `BigInt` เราอยาก output internal data array ของเรา ดังนั้นเราแค่เรียก formatting function ของ `Vec<u64>`

```rust
impl fmt::Debug for BigInt {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        self.data.fmt(f)
    }
}
```

`Debug` implementations สามารถสร้างโดยอัตโนมัติได้โดยใช้ `derive(Debug)`

ทีนี้เราพร้อมใช้ `assert_eq!` เพื่อทดสอบ `vec_min`

```rust
/*#[test]*/
fn test_vec_min() {
    let b1 = BigInt::new(1);
    let b2 = BigInt::new(42);
    let b3 = BigInt::from_vec(vec![0, 1]);
    let v1 = vec![b2.clone(), b1.clone(), b3.clone()];
    let v2 = vec![b2.clone(), b3.clone()];
    assert_eq!(vec_min(&v1), Some(&b1));
    assert_eq!(vec_min(&v2), Some(&b2));
}
```

**แบบฝึกหัด 07.2:** เพิ่ม testcases อีกสักหน่อย โดยเฉพาะ ตรวจสอบให้แน่ใจว่าคุณทดสอบพฤติกรรมของ `vec_min` บนเวกเตอร์ว่างเปล่า นอกจากนี้เพิ่ม tests สำหรับ `BigInt::from_vec` (โดยเฉพาะ การลบเลขศูนย์ต่อท้าย) สุดท้าย ทำลายหนึ่งในฟังก์ชันของคุณในวิธีที่ subtle และดู test fail

**แบบฝึกหัด 07.3:** กลับไปที่ `SomethingOrNothing` ที่ดีๆ ของคุณ และ implement `Display` สำหรับมัน (อันนี้จะต้องการ `Display` bound บน `T` แน่นอน) แล้วคุณควรสามารถใช้มันกับ `println!` ได้เหมือนกับที่คุณทำกับตัวเลข และกำจัด inherent functions สำหรับ print `SomethingOrNothing<i32>` และ `SomethingOrNothing<f32>` ได้
