## Part 09: Iterators

```rust
use part05::BigInt;
```

ต่อไปนี้ เราจะดูเข้าไปในกลไก iterator ของ Rust และทำให้ `BigInt` ของเรา compatible กับ for loops แน่นอน สิ่งนี้เกี่ยวกับการ implement traits บางอย่างอีกครั้ง โดยเฉพาะ iterator คือสิ่งที่ implement `Iterator` trait อย่างที่คุณเห็นในเอกสาร trait นี้บังคับฟังก์ชันเดียวคือ `next` ที่คืน `Option<Self::Item>` โดยที่ `Item` เป็น associated type ที่ถูกเลือกโดย implementation (มี methods มากมายที่ให้มาสำหรับ `Iterator` แต่พวกมันทั้งหมดมี default implementations ดังนั้นเราไม่ต้องกังวลกับพวกมันตอนนี้)

สำหรับกรณีของ `BigInt` เราต้องการ iterator ของเราให้วนลูปผ่านหลักต่างๆ ในลำดับปกติตามสัญกรณ์: หลักที่มีนัยสำคัญมากที่สุดมาก่อน ดังนั้น เราต้องเขียนไทป์บางอย่าง และ implement `Iterator` สำหรับมัน เพื่อให้ `next` คืนหลักต่างๆ ทีละตัว ชัดเจนว่า iterator ต้องสามารถเข้าถึงตัวเลขที่มันวนลูปผ่านได้อย่างใดอย่างหนึ่ง และมันต้องเก็บตำแหน่งปัจจุบันของมัน อย่างไรก็ตาม มันไม่สามารถ own `BigInt` ได้ เพราะงั้นตัวเลขจะหายไปหลังการวนลูป! แน่นอนว่านั่นไม่ดี ทางเลือกเดียวคือให้ iterator ยืม (borrow) ตัวเลข ดังนั้นมันจึงรับ reference

ในการเขียนสิ่งนี้ลงไป เราอีกต้องชัดเจนเกี่ยวกับ lifetime ของ reference: เราไม่สามารถมีแค่ `Iter` ได้ เราต้องมี `Iter<'a>` ที่ยืมตัวเลขสำหรับ lifetime `'a` นี่คือตัวอย่างแรกของชนิดข้อมูลที่เป็น polymorphic ใน lifetime ต่างจาก type variable

`usize` ที่นี่คือไทป์ของ unsigned numbers ที่มีขนาดเท่ากับ pointer โดยทั่วไปมันเป็นไทป์ของ "ความยาวของสิ่งต่างๆ" โดยเฉพาะ มันเป็นไทป์ของความยาวของ `Vec` และดังนั้นจึงเป็นไทป์ที่ถูกต้องสำหรับเก็บ offset เข้าไปในเวกเตอร์ของหลัก

```rust
pub struct Iter<'a> {
    num: &'a BigInt,
    idx: usize, // the index of the last number that was returned
}
```

ทีนี้เราพร้อมที่จะ implement `Iterator` สำหรับ `Iter`

```rust
impl<'a> Iterator for Iter<'a> {
```

เราเลือกไทป์ของสิ่งที่เราวนลูปผ่านให้เป็นไทป์ของหลัก คือ `u64`

```rust
    type Item = u64;
    
    fn next(&mut self) -> Option<u64> {
```

ก่อนอื่น ตรวจสอบว่ามีหลักเหลืออยู่อีกหรือไม่

```rust
        if self.idx == 0 {
```

เราได้คืนหลักทั้งหมดแล้ว ไม่มีอะไรเหลือ

```rust
            None
        } else {
```

มิฉะนั้น: ลดค่า และคืนหลักถัดไป

```rust
            self.idx = self.idx - 1;
            Some(self.num.data[self.idx])
        }
    }
}
```

สิ่งที่เราต้องการตอนนี้คือฟังก์ชันที่สร้าง iterator แบบนี้สำหรับ `BigInt` ที่กำหนด

```rust
impl BigInt {
```

สังเกตว่าเมื่อเราเขียนไทป์ของ `iter` เราจริงๆ แล้วไม่ต้องให้ lifetime parameter ของ `Iter` เหมือนกับกรณีของฟังก์ชันที่คืน references คุณสามารถละ lifetime ได้ กฎสำหรับการเพิ่ม lifetimes เหมือนกันเป๊ะ (ดูส่วนสุดท้ายของบท 06)

```rust
    fn iter(&self) -> Iter {
        Iter { num: self, idx: self.data.len() }
    }
}
```

เราพร้อมที่จะวนลูปสักที! อย่าลืมแก้ไข `main.rs` เพื่อรันโค้ดนี้

```rust
pub fn main() {
    let b = BigInt::new(1 << 63) + BigInt::new(1 << 16) + BigInt::new(1 << 63);
    for digit in b.iter() {
        println!("{}", digit);
    }
}
```

แน่นอน เราไม่จำเป็นต้องใช้ `for` เพื่อ apply iterator เรายังสามารถเรียก `next` อย่างชัดเจนได้

```rust
fn print_digits_v1(b: &BigInt) {
    let mut iter = b.iter();
```

`loop` คือ keyword สำหรับ loop ที่ไม่มีเงื่อนไข: มันวนไปเรื่อยๆ จนกว่าคุณจะ break ออกจากมันด้วย `break` หรือ `return`

```rust
    loop {
```

ทุกครั้งที่เราผ่าน loop เราวิเคราะห์ element ถัดไปที่ iterator นำเสนอ - จนกว่ามันจะหมด

```rust
        match iter.next() {
            None => break,
            Some(digit) => println!("{}", digit)
        }
    }
}
```

ทีนี้ ปรากฏว่า combination นี้ของการทำ loop และ pattern matching ค่อนข้าง common และ Rust ให้ syntactic sugar ที่สะดวกสำหรับมัน

```rust
fn print_digits_v2(b: &BigInt) {
    let mut iter = b.iter();
```

`while let` ทำ pattern matching ที่กำหนดในทุกรอบของ loop และยกเลิก loop ถ้า pattern ไม่ตรง ยังมี `if let` ที่ทำงานคล้ายกัน แต่แน่นอนไม่มีการวนลูป

```rust
    while let Some(digit) = iter.next() {
        println!("{}", digit)
    }
}
```

**แบบฝึกหัด 09.1:** เขียน testcase สำหรับ iterator ตรวจสอบให้แน่ใจว่ามัน yield หลักที่ถูกต้อง

**แบบฝึกหัด 09.2:** เขียนฟังก์ชัน `iter_ldf` ที่วนลูปผ่านหลักโดยที่หลักที่มีนัยสำคัญน้อยที่สุดมาก่อน เขียน testcase สำหรับมัน

## Iterator invalidation และ lifetimes

คุณอาจ surprise ว่าเราต้อง annotate lifetime อย่างชัดเจนเมื่อเราเขียน `Iter` แน่นอน ด้วย lifetimes มีอยู่ทุก reference ใน Rust สิ่งนี้เป็นเพียง consistent แต่เราได้รับอะไรจากภาระ annotation พิเศษนี้หรือไม่? (ขอบคุณ ภาระนี้เกิดขึ้นเฉพาะเมื่อเรานิยาม types และไม่ใช่เมื่อเรานิยามฟังก์ชัน - ซึ่งโดยทั่วไปมากกว่ามาก)

ปรากฏว่าคำตอบคือใช่! แง่มุมเฉพาะนี้ของแนวคิด lifetimes ช่วยให้ Rust กำจัดปัญหา **iterator invalidation** ได้ พิจารณาโค้ดส่วนนี้

```rust
fn iter_invalidation_demo() {
    let mut b = BigInt::new(1 << 63) + BigInt::new(1 << 16) + BigInt::new(1 << 63);
    for digit in b.iter() {
        println!("{}", digit);
        /* b = b + BigInt::new(1); */ /* BAD! */
    }
}
```

ถ้าคุณ enable บรรทัดที่ไม่ดี Rust จะปฏิเสธโค้ด ทำไม? ปัญหาคือเรากำลังแก้ไขตัวเลขขณะที่วนลูปผ่านมัน ในภาษาอื่นๆ สิ่งนี้สามารถมีผลกระทบหลากหลายจากข้อมูลไม่ consistent หรือ throwing exception (Java) ไปจนถึง dereference bad pointers (C++) อย่างไรก็ตาม Rust สามารถตรวจจับสถานการณ์นี้ได้ เมื่อคุณเรียก `iter` คุณต้องยืม `b` สำหรับ lifetime บางอย่าง `'a` และคุณได้รับ `Iter<'a>` นี่คือ iterator ที่ valid สำหรับ lifetime `'a` เท่านั้น โชคดี เรามี annotation นี้พร้อมใช้เพื่อทำ statement แบบนั้น Rust บังคับใช้ว่า `'a` ครอบคลุมทุกการเรียก `next` ซึ่งหมายความว่ามันต้องครอบคลุม loop ดังนั้น `b` ถูกยืมสำหรับระยะเวลาของ loop และเราไม่สามารถ mutate มันได้ นี่เป็น yet another example สำหรับว่า combination ของ mutation และ aliasing นำไปสู่ผลกระทบที่ไม่พึงประสงค์ (ไม่จำเป็นต้อง crash คิดถึง Java) ซึ่ง Rust ป้องกันได้สำเร็จ

## Iterator conversion trait

ถ้าคุณเปรียบเทียบ for loop ใน `main` ข้างต้น กับที่ใน `part06::vec_min` อย่างใกล้ชิด คุณจะสังเกตว่าเราสามารถเขียน `for e in v` ได้ก่อนหน้านี้ แต่ตอนนี้เราต้องเรียก `iter` ทำไมล่ะ? นั่นก็เพราะ sugar ของ `for` ไม่ได้ tied กับ `Iterator` จริงๆ แต่มันต้องการ implementation ของ `IntoIterator` นั่นคือ trait ของ types ที่ provide conversion function ไปยังบางชนิดของ iterator Conversion traits เป็น pattern ที่พบบ่อยใน Rust: มากกว่าที่จะต้องการว่าบางอย่างเป็น iterator หรือ string หรืออะไรก็ตาม; เราต้องการว่าบางอย่างสามารถแปลงเป็น iterator/string/whatever ได้ สิ่งนี้ให้ความสะดวกคล้ายกับ overloading ของฟังก์ชัน: ฟังก์ชันสามารถถูกเรียกด้วย types ต่างๆ มากมาย โดยการ implement traits แบบนี้สำหรับ types ของคุณ คุณยังสามารถทำให้ types ของคุณเองทำงานราบรื่นกับ library functions ที่มีอยู่ได้ อย่างที่เป็นกฎสำหรับ Rust abstraction นี้มาที่ zero cost: ถ้าข้อมูลของคุณเป็นไทป์ที่ถูกต้องอยู่แล้ว ฟังก์ชัน conversion จะไม่ทำอะไรและถูก optimize ออกไปอย่าง trivially

ถ้าคุณดูเอกสารของ `IntoIterator` คุณจะสังเกตว่าฟังก์ชัน `into_iter` ที่มัน provide จริงๆ แล้ว consume argument ของมัน ดังนั้นเรา implement trait สำหรับ references ไปยังตัวเลข เพื่อให้ตัวเลขไม่หายไปหลังการวนลูป

```rust
impl<'a> IntoIterator for &'a BigInt {
    type Item = u64;
    type IntoIter = Iter<'a>;
    
    fn into_iter(self) -> Iter<'a> {
        self.iter()
    }
}
```

ด้วยสิ่งนี้ คุณสามารถแทนที่ `b.iter()` ใน `main` ด้วย `&b` ไปลองดูสิ! เดี๋ยว `&b` ? ทำไมล่ะ? นั่นก็เพราะเรา implement `IntoIterator` สำหรับ `&BigInt` ถ้าเราอยู่ในที่ที่ `b` ถูกยืมอยู่แล้ว เราสามารถทำ `for digit in b` ได้ แต่ถ้าเรา own `b` เราต้องสร้าง reference ไปยังมัน หรือเราอาจ implement `IntoIterator` สำหรับ `BigInt` - ซึ่งอย่างที่กล่าวไปแล้ว จะหมายความว่า `b` ถูก consume โดยการวนลูป และหายไป สิ่งนี้สามารถเกิดขึ้นได้ง่ายๆ เช่น กับ `Vec`: ทั้ง `Vec` และ `&Vec` (และ `&mut Vec`) implement `IntoIterator` ดังนั้นถ้าคุณทำ `for e in v` และ `v` มีไทป์ `Vec` แล้วคุณจะได้รับ ownership ของ elements ระหว่างการวนลูป - และทำลาย vector ไปในกระบวนการ เราจริงๆ แล้วทำแบบนั้นใน `part01::vec_min` แต่เราไม่สนใจ คุณสามารถเขียน `for e in &v` หรือ `for e in v.iter()` เพื่อหลีกเลี่ยงสิ่งนี้
